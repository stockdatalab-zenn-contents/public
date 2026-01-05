---
title: "高速化"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", ]
published: false
---
## はじめに
以前、「[SQLite検索が遅い？LIKEの限界とFTS5で高速化する方法](https://zenn.dev/stockdatalab/articles/20251010_tech_fts5)」でFTS5で検索処理を高速化する方法について触れました。前回の対応では十分に高速化しない、もっと高速化したいと思い今回は、より詳しく幅広く、検索処理を高速化する方法をまとめます。工夫ポイントの全体像は以下です。各工夫について後続の章で解説します。
![](/images/20251205_tech_speedupsearching/overall.png =600x)

## 1. 並列処理・分散処理
使用するCPUのコア数を増やします。PCが持つすべてのコアは使用せず、半分程度のコア数を指定することを推奨します。並列処理・分散処理の方法としては他に、GPUの使用や複数のPCを使用した分散処理も考えられますが、環境構築の負荷が高くなる可能性があるので、まずは後続の章の対応を進めてみることを推奨します。
:::details アーカイブ
```py
"""
汎用 FTS5＋通常検索テンプレート
----------------------------------
複数の検索タイプ（部分一致/完全一致）に対応した
テンプレート関数として利用可能。
"""

import sqlite3
from datetime import datetime
from concurrent.futures import ProcessPoolExecutor, as_completed, TimeoutError


# =========================================
#  接続作成（読み取り専用）
# =========================================
def open_readonly_conn(db_path: str):
    return sqlite3.connect(f"file:{db_path}?mode=ro", uri=True)


# =========================================
#  FTS5 用クエリビルダ（例）
#  ※実運用では独自実装で差し替え
# =========================================
def build_fts_query(field_value: str, ngram: int):
    """FTS5用に n-gram クエリを組み立てる簡易例"""
    if not field_value:
        return None
    tokens = [field_value[i:i+ngram] for i in range(len(field_value) - ngram + 1)]
    return " AND ".join(tokens)


# =========================================
#  実際の検索（1レコード分）
# =========================================
def search_record(
    row: dict,
    search_type: str,
    fts_table: str,
    data_table: str,
    topk: int,
    n_ngram: int,
    db_path: str,
):
    """
    row        : dict（検索元の行）
    search_type: 'fts_name', 'fts_addr', 'id', 'phone' など自由定義
    fts_table  : FTS5 テーブル名
    data_table : データ保持テーブル名
    topk       : FTS 上位何件を返すか
    n_ngram    : N-gram サイズ
    db_path    : SQLite DB パス
    """

    try:
        conn = open_readonly_conn(db_path)
        cur = conn.cursor()

        result_local = []

        # 必要な項目（自由拡張可）
        text_value = row.get("text_value")
        id_value   = row.get("id_value")
        phone      = row.get("phone_value")

        params = []
        query_conditions = []
        fts_query = None

        # --------------------------------------------------------
        # 検索タイプごとに条件分岐（必要に応じて増やせる）
        # --------------------------------------------------------
        if search_type == "fts_text":
            fts_query = build_fts_query(text_value, n_ngram)

        elif search_type == "id_exact":
            if id_value:
                params.append(id_value)
                query_conditions.append("id_value = ?")

        elif search_type == "phone_exact":
            if phone:
                params.append(phone)
                query_conditions.append("phone_value = ?")

        # FTS 条件（部分一致）
        if fts_query:
            params.append(fts_query)
            query_conditions.append(f'"{fts_table}" MATCH ?')

        # 条件なしならスキップ
        if not query_conditions:
            conn.close()
            return []

        # --------------------------------------------------------
        # SQL の組み立て（FTS検索か通常検索かで分岐）
        # --------------------------------------------------------
        if search_type in ("id_exact", "phone_exact"):
            sql = f"""
                SELECT d.rowid, d.id_value, d.text_value, d.addr_value
                FROM "{data_table}" AS d
                WHERE {" AND ".join(query_conditions)}
            """
        else:
            # FTS5（スコア付き検索）
            sql = f"""
                WITH topk AS (
                    SELECT rowid, bm25("{fts_table}") AS score
                    FROM "{fts_table}"
                    WHERE {" AND ".join(query_conditions)}
                    ORDER BY score ASC
                    LIMIT {topk}
                )
                SELECT t.rowid, d.id_value, d.text_value, d.addr_value, t.score
                FROM topk AS t
                JOIN "{data_table}" AS d ON t.rowid = d.rowid
                ORDER BY t.score ASC
            """

    except Exception as e:
        print(datetime.now())
        print(f"[ERROR] SQL 構築エラー: {e}")
        print("conditions:", query_conditions)
        raise

    # --------------------------------------------------------
    # SQL 実行
    # --------------------------------------------------------
    try:
        hits = cur.execute(sql, params).fetchall()

    except sqlite3.Error as e:
        print(datetime.now())
        print(f"[SQL ERROR] {e}")
        print("SQL:", sql)
        print("PARAMS:", params)
        raise

    # --------------------------------------------------------
    # 結果整形
    # --------------------------------------------------------
    for hit in hits:
        result = {
            "元データ行ID": row.get("__name__"),
            "検索先ID": hit[1],
            "検索先テキスト": hit[2],
            "検索先住所": hit[3],
        }
        # スコア列は存在する場合のみ追加
        if len(hit) > 4:
            result["スコア"] = f"{hit[4]:.2f}"

        result_local.append(result)

    conn.close()
    return result_local


# =========================================
#  並列実行テンプレート
# =========================================
def parallel_search(df, max_workers, **search_kwargs):
    results = []

    with ProcessPoolExecutor(max_workers=max_workers) as executor:
        futures = []

        for _, row in df.iterrows():
            row_dict = row.to_dict()
            row_dict["__name__"] = row.name  # 元行の識別子
            futures.append(executor.submit(search_record, row_dict, **search_kwargs))

        for f in as_completed(futures, timeout=120):
            try:
                results.extend(f.result())
            except Exception as e:
                print(datetime.now(), "[WORKER ERROR]", e)
                continue

    return results

```
:::

## 2. SQLの書き方を工夫する

## 3. 物理的な観点でデータの持ち方を工夫
## 4. 論理的な観点でデータの持ち方を工夫

## おわりに
NIM


