---
title: "大きなテーブルデータの最適化手法：pandas・polars・Daskで速く扱う"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["大規模", "大容量", "テーブルデータ"]
published: true
---
## はじめに
データ分析や機械学習の現場では、数百万〜数千万行規模のテーブルデータを扱う機会が珍しくありません。しかし、そのまま処理しようとするとメモリ不足や実行速度の低下に直面します。
この記事では、pandas・polars・Dask といった Python ライブラリを例に、大容量テーブルデータを効率よく扱うための実践的な工夫を紹介します。
![](/images/20250916_tech_bigtabledata/flow.png =500x)


## 1.データ量の最適化
![](/images/20250916_tech_bigtabledata/selectdata.png =500x)
最も効果的な方法は「そもそも不要なデータを持たない」ことです。読み込み前後の段階で対象を絞るだけで、その後の処理が大幅に軽くなります。

- **必要な列だけに絞り込む**
- **必要な行だけに絞り込む**
- **サンプリングする**（検証時に有効）
- **型を縮小する**（int64→int32 / float32 / category など）

:::details pandasでのコーディング例
```py:pandas.py
import pandas as pd

use_cols = ["A", "B", "C"]
# 必要列だけ読む + 文字列をカテゴリ化 + intを32bitへ
df = pd.read_csv("data.csv", usecols=use_cols, dtype={"A": "int32"})
df["C"] = df["C"].astype("category")

# 条件で間引き（例：A > 1000の行だけ）
df = df.query("A > 1000")

# 検証時はサンプリング（1%）
df_small = df.sample(frac=0.01, random_state=42)
```
:::
:::details polarsでのコーディング例
```py:polars.py
import polars as pl

# 必要列だけ + dtype指定（推論も速い）
df = pl.read_csv("data.csv", columns=["A", "B", "C"])
df = df.with_columns([
    pl.col("A").cast(pl.Int32),
    pl.col("C").cast(pl.Categorical),
]).filter(pl.col("A") > 1000)

# サンプリング
df_small = df.sample(fraction=0.01, seed=42)
```
:::
:::details daskでのコーディング例
```py:dask.py
import dask.dataframe as dd

ddf = dd.read_csv("data.csv", usecols=["A", "B", "C"], dtype={"A": "int32", "C": "object"})
ddf = ddf[ddf["A"] > 1000]
# サンプル（近似）：最初の数パーティションで代表抽出などを自前で実装するのが実務的
head_sample = ddf.head(1_000_000).sample(frac=0.01, random_state=42)
```
:::


## 2. 読み込み・書き出し・実行計画を最適化
![](/images/20250916_tech_bigtabledata/process.png =500x)
ディスク容量やメモリの使用量を効率化して、入出力を高速化するポイントは以下です。

- **チャンク処理してリストに集約**
- zip/gzipなどの**圧縮ファイルを解凍せずそのまま読む**（ディスク容量を節約）
- **書き出しは最後に一括して行う**


:::details pandasでのコーディング例
```py:pandas1.py
import pandas as pd

records = []
for chunk in pd.read_csv("data.zip", usecols=["A", "B", "C"], dtype={"A": "int32"}, chunksize=1_000_000):
    ch = chunk.query("A > 1000")
    ch["X"] = ch["A"] + ch["B"]

    records.append({
        "rows": len(ch),
        "sum_X": ch["X"].sum(),
        "max_X": ch["X"].max()
    })

summary_df = pd.DataFrame(records)
print(summary_df)
```
```py:pandas2.py
import pandas as pd

records = []
for chunk in pd.read_csv("data.csv", usecols=["A", "B"], chunksize=500_000):
    # どうしても行ベースが必要なら itertuples を使う（iterrowsより速い）
    for row in chunk.itertuples(index=False):
        records.append({"A": row.A, "B": row.B, "X": row.A * row.B})

df = pd.DataFrame(records)
```
:::
:::details polarsでのコーディング例
```py:polars1.py
import polars as pl
import pandas as pd  # 既存コード資産が多いなら最終はpandas変換もあり

records = []
# polarsはread_csvにchunksizeはないので、巨大CSVならscan_csv(streaming=True)で直接DFにしない設計が基本
# 「リスト前提」に合わせて、ここでは“読み込み→すぐサマリ→リストへ”の方針にする
df = pl.read_csv("data.csv", columns=["A", "B", "C"])
df = df.filter(pl.col("A") > 1000).with_columns((pl.col("A") + pl.col("B")).alias("X"))

# 例：上位N行だけを記録（全行をリスト化しない）
top = df.sort("X", descending=True).head(100)
records.extend(top.to_dicts())  # ← ここで初めてリスト化

# 最終的にpolars / pandasどちらにしてもOK
result_pl = pl.DataFrame(records)
result_pd = pd.DataFrame(records)
```
```py:polars2.py
import polars as pl

q = (pl.scan_parquet("parq/*.parquet")
       .with_columns((pl.col("A") + pl.col("B")).alias("X"))
       .filter(pl.col("X") > 1000)
       .groupby([])
       .agg(pl.col("X").sum().alias("sumX")))
out = q.collect()  # 必要なときだけ実行
```
:::
:::details daskでのコーディング例
```py:dask1.py
import dask.dataframe as dd
import pandas as pd

ddf = dd.read_csv("data.csv", usecols=["A", "B", "C"])
ddf = ddf[ddf["A"] > 1000]
ddf = ddf.assign(X = ddf["A"] + ddf["B"])

# 各パーティションから「小さい結果」（例：上位5行）だけ取り出す
def topk_part(df, k=5):
    return df.nlargest(k, "X")[["A", "B", "X"]]

parts = ddf.map_partitions(topk_part, k=5).compute()  # ここで初めて収集
records = parts.to_dict("records")
final_df = pd.DataFrame(records)
```
```py:dask2.py
import dask.dataframe as dd

ddf = dd.read_parquet("parq/*.parquet")
ddf = ddf.assign(X = ddf["A"] + ddf["B"])
res = ddf[ddf["X"] > 1000]["X"].sum().compute()  # 計算時に並列実行
```
:::


## 3. 計算資源の使い方を変える（並列・分散の活用）
![](/images/20250916_tech_bigtabledata/process2.png =250x)
単機の限界に近づいたら、並列（polars）・分散（dask）で横に広げる工夫が有効です。
- pandas：1コアで順次処理
- polars：複数コアを並列利用 して処理を分割（設定でスレッド数の調整可）
- Dask：大きなデータフレームを複数の小さいデータフレームに分割し、それぞれを別プロセスや別マシン に投げて並列処理。

:::details polarsでのコーディング例
```py:polars.py
import os
import polars as pl

# 必要に応じてスレッド数を制御（未指定なら自動）
os.environ["POLARS_MAX_THREADS"] = "8"

# 売上明細（多数CSV/CSV.GZ）
sales = (
    pl.scan_csv("data/sales_*.csv.gz")       # 例: columns=["order_id","sku","qty","price","store_id","ts"]
      .with_columns(
          (pl.col("qty") * pl.col("price")).alias("amount")
      )
      .groupby(["store_id", "sku"])
      .agg([
          pl.sum("qty").alias("qty_sum"),
          pl.sum("amount").alias("amount_sum"),
      ])
)

# 商品マスタ（小規模CSV。必要列だけ）
items = (
    pl.scan_csv("data/items.csv")            # 例: columns=["sku","category","cost"]
      .select(["sku", "category"])
)

# 結合 → カテゴリ別/店舗別の売上合計
result = (
    sales.join(items, on="sku", how="left")
         .groupby(["store_id", "category"])
         .agg(pl.sum("amount_sum").alias("revenue"))
         .sort(["store_id", "category"])
)

# ストリーミング（低メモリ）で実行
out_df = result.collect(streaming=True)
print(out_df.head(10))

# 必要ならCSVで出力（分割せず一括）
out_df.write_csv("out/revenue_by_store_category.csv")
```
:::
:::details daskでのコーディング例
```py:dask.py
# dask_parallel_csv.py
import dask
import dask.dataframe as dd

# CPUバウンドの集計想定 → プロセス並列
dask.config.set(scheduler="processes")

# 売上明細（多数CSV/CSV.GZ）をパーティション化して読み込む
# columns=... / dtype=... を指定すると安定しやすい
sales = dd.read_csv(
    "data/sales_*.csv.gz",
    assume_missing=True,    # 推奨: Null推定の安全側
    dtype={"store_id": "int64", "sku": "object", "qty": "float64", "price": "float64"}
)

# パーティション内で先に列を作る → groupbyで圧縮
sales = sales.assign(amount = sales["qty"] * sales["price"])

sales_gb = (
    sales.groupby(["store_id", "sku"])[["qty", "amount"]]
         .sum()
         .rename(columns={"qty": "qty_sum", "amount": "amount_sum"})
)

# 商品マスタ（小規模ならpandasで読んで dd.from_pandas でもOKだが、ここはCSV）
items = dd.read_csv("data/items.csv", dtype={"sku":"object","category":"object"})[["sku", "category"]]

# skuで結合（大規模なら shuffle="tasks" などオプション検討）
joined = sales_gb.merge(items, on="sku", how="left")

# カテゴリ別/店舗別の売上合計
revenue = joined.groupby(["store_id", "category"])["amount_sum"].sum().reset_index()

# 並列実行して結果をpandas DataFrameに収集
out_df = revenue.compute()
print(out_df.head(10))

# 必要ならCSVへ（パーティション分割で出したい場合は to_csv("out/part-*.csv")）
out_df.to_csv("out/revenue_by_store_category.csv", index=False)
```
:::


## おわりに
大容量データを扱う際は、
- 不要なデータを持たない
- I/Oと実行計画を工夫する
- 並列・分散処理を活用する

という3段階で取り組むと効率が改善します。本記事の工夫は、pandas・polars・Dask を中心に紹介しましたが、この考え方は他の分散基盤にも応用できると思います。
