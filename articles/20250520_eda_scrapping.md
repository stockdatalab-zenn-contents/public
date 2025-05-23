---
title: "大量カラムのEDAどうする？自動化ライブラリの比較とsweetvizの限界を補完する方法"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["EDA", "探索的データ分析", "sweetviz", "スクレイピング", "BeautifulSoup"]
published: true
---
## はじめに
こんにちは、SE出身の駆け出しデータサイエンティストの「マチ」です。最近、**カラム数が100を超える構造化データ**の分析に取り組む機会がありました。正直、「この量を1つ1つ見るのは面倒だな…」と感じて、最初の一歩がなかなか進まなかったのが本音です。

そこで、「**なるべく時間をかけずに、効率よくEDA（探索的データ分析）を済ませたい**」と考え、実際に活用して効果的だった方法を備忘録として残します。キーワードは 「**sweetviz × スクレイピング**」 です。



## よく使われるEDAライブラリの比較
まずは、Pythonで使える代表的なEDA自動化ライブラリを簡単に紹介します。なお、「出力されるファイルイメージ」では、以下の通り取得・加工したsklearnの乳がんのデータを使用しています。コードは、[以前の記事](https://zenn.dev/stockdatalab/articles/20250519_tech_env_docker)に沿って構築したdocker環境で簡単な動作確認をしています。
- **データの取得・加工例**
```py:ipynbファイル
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
import pandas as pd

breast_cancer = load_breast_cancer()
X_df = pd.DataFrame(breast_cancer["data"], columns = breast_cancer["feature_names"])
y_df = pd.DataFrame(breast_cancer["target"], columns = ["target"])
X_train, X_test, y_train, y_test = train_test_split(X_df, y_df, test_size = 0.3, random_state = 123)
original_df = pd.concat([X_df, y_df], axis=1)
train_df = pd.concat([X_train, y_train], axis=1)
test_df = pd.concat([X_test, y_test], axis=1)

filename = './breast_cancer.csv'
original_df.to_csv(filename)
display(original_df.head(3))
```

:::details ydata-profiling（旧 pandas_profiling）
- **メリット**
統計要約・欠損値・分布・相関などを網羅しており、インタラクティブなUIが使いやすい。
- **デメリット**
データ量やカラム数が多いと、非常に重くなり処理が遅い。
- **実行コード例**
```py:ipynbファイル
# !pip install ydata_profiling            # 必要に応じてインストール
# !pip install setuptools                 # 必要に応じてインストール
from ydata_profiling import ProfileReport

profile = ProfileReport(original_df, title="Profiling Report")
# profile.to_notebook_iframe()            					# Jupyter Notebookのセル内に、プロファイリングレポートを iframe 形式で表示
profile.to_file("./ydata_profiling.html")					# ファイル出力
```
- **出力されるファイルイメージ**
![](/images/20250520_eda_scrapping/ydata_profiling.png =800x)
:::



:::details AutoViz
- **メリット**
コード1行でグラフ多数を自動生成できる。目的変数を指定して分析することも可能。
- **デメリット**
作成されたグラフが複数ファイルに分割されて出力される
データ量やカラム数が多いと、グラフを出力できない。
- **実行コード例**
```py:ipynbファイル
# !pip install autoviz            # 必要に応じてインストール
# import numpy as np
from autoviz.AutoViz_Class import AutoViz_Class

filename = ""
sep = ','
target = original_df.columns[-1]
AV = AutoViz_Class()

# グラフ出力
# # --------以下のいずれかを実行---------
# pngで出力
dft = AV.AutoViz(filename, sep=sep, depVar=target, dfte=original_df, header=0, verbose=2,
lowess=False,chart_format='png',max_rows_analyzed=150000,max_cols_analyzed=30)
# htmlで出力
# dft = AV.AutoViz(filename, sep=sep, depVar=target, dfte=original_df, header=0, verbose=2,
# lowess=False,chart_format='html',max_rows_analyzed=150000,max_cols_analyzed=30)
# # ---------------------------------------

```
AutoVizの引数は以下の通りです。（参考：[ライブラリの説明](https://pypi.org/project/autoviz/)）
|   引数   |           内容           |
|   ----   |          ----            |
|filename            | 読み込むファイル名（CSV、TXT、JSON 形式）。ファイルを使用しない場合は空文字列 "" を指定し、代わりに dfte にデータフレームを渡します。|
|sep                 | ファイル内の列を区切るセパレーター。デフォルトはカンマ（,）ですが、セミコロン（;）やタブ（\t）なども指定可能。|
|depVar              | 目的変数（ターゲット変数）の列名。指定しない場合は空文字列 "" を設定する。|
|dfte                | Pandas のデータフレームを直接渡す場合に使用する。この場合、filename は空文字列 "" にする。|
|header              | ファイルのヘッダー行の行番号。最初の行がヘッダーの場合は 0 を指定する。|
|verbose             | 出力の詳細度を制御します。0：最小限の情報とチャートを表示、1：詳細な情報とチャートを表示、2：チャートを表示せずにローカルに保存する。|
|lowess              | 小規模なデータセットに対して、連続変数と目的変数の関係を示す回帰線（LOWESS）を描画するかどうかを指定する。大規模なデータセット（10万行以上）には推奨されない。|
|chart_format        | 生成するチャートの形式を指定する。'svg'、'png'、'jpg'：Matplotlib 形式のチャートを生成。'bokeh'：Jupyter Notebook 上でインタラクティブな Bokeh チャートを生成。'server'：各種チャートをブラウザで表示。'html'：インタラクティブな Bokeh チャートを HTML ファイルとして保存。|
|max_rows_analyzed   | 可視化に使用する最大行数を指定する。大規模なデータセットの場合、統計的に有効なサンプルを使用してチャートを生成する。デフォルトは 150000 行。|
|max_cols_analyzed   | 分析対象とする連続変数の最大列数を指定する。デフォルトは 30 列。|
|save_plot_dir       | チャートを保存するディレクトリを指定する。デフォルトは None で、現在の作業ディレクトリ内に AutoViz_Plots フォルダが作成され、そこに保存される。指定したディレクトリが存在しない場合は、自動的に作成される。|
- **出力されるファイルイメージ**
![](/images/20250520_eda_scrapping/autoviz.png =800x)
:::



:::details sweetviz
- **メリット**
訓練データとテストデータの比較ができる。目的変数を指定して分析することも可能。
- **デメリット**
カラム数が多いと、「Associations」の図の文字が潰れて読めない。
numpy2.0.0以上の場合、エラーが出て動作しない場合がある。
- **実行コード例**
  - **初期値などの設定**
```py:ipynbファイル
# https://github.com/fbdesignpro/sweetviz/issues/144
# ! pip uninstall numpy
# ! pip install numpy==1.24.0
# !pip install sweetviz            # 必要に応じてインストール
import sweetviz as sv

# 使用する値を設定（要編集）
htmlfile_path1 = './sweetviz.html'                             # sweetvizが出力するレポートのパス（ファイル名）を設定
htmlfile_path2 = './sweetviz_compare.html'                     # sweetvizが出力するレポートのパス（ファイル名）を設定
cat_list = []         # 例：cat_list = ['区分１', '区分２']     # カテゴリー項目名を設定
num_list = []         # 例：num_list = ['購入金額', '年齢']     # 数値項目名を設定
target_feat = 'target'
```
  - **htmlレポートを出力**
```py:ipynbファイル
#######################################
##### sweetvizでEDAレポートを出力する（１データのみ）
#######################################
def export_sweetvizReport_for1data(cat_list, num_list, df, htmlfile_path, target_feat):
    # 必要に応じて明示的にカテゴリ型に変換
    # df[cat_list] = df[cat_list].astype('category')
    # df[num_list] = df[num_list].astype('float')  # または int など

    # 日本語の文字化けを防止
    sv.config_parser.read_string("[General]\nuse_cjk_font=1")

    # --------以下のいずれか１行を実行---------
    ## 値の加工なし
    # report = sv.analyze(result_forCheck)  
    ## 目的変数指定あり。変換できない値（例えば文字列など）を強制的に NaN に変換。
    ## target_featが先頭に来る（想定と異なる順序に並ぶ）ため、出力結果を後続で定義するメソッドget_associationsには使用しないこと。
    # report = sv.analyze(result_forCheck.apply(pd.to_numeric, errors='coerce'), target_feat=target_feat) 
    ## 目的変数指定なし。変換できない値（例えば文字列など）を強制的に NaN に変換。
    report = sv.analyze(df.apply(pd.to_numeric, errors='coerce'))  
    # ---------------------------------------

    # html出力
    report.show_html(htmlfile_path)
    return None

#######################################
##### sweetvizでEDAレポートを出力する（２データを比較）
#######################################
def export_sweetvizReport_for2data(cat_list, num_list, train_df, test_df, htmlfile_path, target_feat):
    # 必要に応じて明示的にカテゴリ型に変換
    # train_df[cat_list] = train_df[cat_list].astype('category')
    # train_df[num_list] = train_df[num_list].astype('float')  # または int など

    # 日本語の文字化けを防止
    sv.config_parser.read_string("[General]\nuse_cjk_font=1")
    ## 2データ比較
    report = sv.compare(train_df, test_df, target_feat=target_feat)
    # html出力
    report.show_html(htmlfile_path)
    return None


export_sweetvizReport_for1data([], [], original_df, htmlfile_path1, "")
export_sweetvizReport_for2data([], [], train_df, test_df, htmlfile_path2, target_feat)
```
- **出力されるファイルイメージ**
![](/images/20250520_eda_scrapping/sweetviz.png =800x)
:::


- **まとめ・比較**

|   ライブラリ    |  処理  | 目的変数の設定 | 複数データの比較 | 備考 |
|      ----       |  ----  |      ----      |       ----       | ---- |
| ydata-profiling |  重い  |      不可      |       不可       |  大規模データには不向き。  |
| AutoViz         |  軽い  |       可       |       不可       |  多様なグラフを出力できる。<br>超大規模のデータには不向き。  |
| sweetviz        |  軽い  |       可       |        可        |  「Associations」の図のテキストが<br>潰れて読めない場合がある。  |



## sweetvizの弱点を補う方法：HTMLのスクレイピング
sweetvizは、動作が軽い点やデータの概況を比較ができる点が魅力的です。しかし、「Associations」の図は**カラム数が多くなると、文字が小さくなったり重なったりして見づらくなります**。ブラウザの縮尺を変えて図を大きくできないか試してみたり、htmlの要素を開発者ツールで修正しようとしてみたりしましたが、いずれも解決に至ることはできませんでした。そこで、「**sweetvizが出力したHTMLレポートをスクレイピングして、各項目の相関を別途整理する**」という方法を試してみました。

### 実施概要
以下の方法により、視覚的に見づらくなった相関情報をテーブル形式で把握でき、次の分析アクションに移りやすくなりました。

![](/images/20250520_eda_scrapping/flow.png =400x)
1. sweetvizで出力されたHTMLファイルをBeautifulSoupで読み込み
2. 「Associations」の要素から、項目のペアと相関スコアを抽出（スクレイピング）
3. データフレームに変換して整理



### コード
- 1. sweetvizで出力されたHTMLファイルをBeautifulSoupで読み込み
```py:ipynbファイル
def read_html(htmlfile_path):
    with open(htmlfile_path, "r", encoding="utf-8") as file:
        html_content = file.read()
    soup = BeautifulSoup(html_content, 'html.parser')
    return soup
```
- 2. 「Associations」の要素から、項目のペアと相関スコアを抽出（スクレイピング）
```py:ipynbファイル
def get_associations(soup, original_df, associationsCsvfile_path):
    # 出力リスト
    data = []

    # 親要素の名前リスト（列の順番に対応）
    parent_names = original_df.columns.tolist()

    # pos-detail-cat-assoc-1 配下の container-detail-assoc 要素を対象に処理
    assoc_sections = soup.select("div.pos-detail-cat-assoc-1 div.container-detail-assoc")


    for idx, container in enumerate(assoc_sections):

        # タイトル区分を記録
        is_provides_block = False

        parent_name = parent_names[idx] if idx < len(parent_names) else f"親要素{idx+1}"

        for elem in container.find_all(['div'], recursive=False):    # 入れ子（ネスト）された div は対象外となり、直下の階層のみが対象
            # 大見出しの判定
            if elem.get('class') and 'text-large-bold' in elem.get('class'):

            # # 親ブロックの中を順に探索
                if elem.name == 'div' and 'text-large-bold' in elem.get('class', []):
                    # セクションの見出しを確認
                    if any(key in elem.get_text() for key in [
                        # 以下のブロックは、相関項目と相関値を取得
                        'NUMERICAL ASSOCIATIONS',   # Pearson相関係数
                        'CATEGORICAL ASSOCIATIONS', # 相関比
                        'CORRELATION RATIO WITH',   # 相関比
                        'PROVIDES INFORMATION ON',  # 不確実性係数（非対称な指標である点に注意）
                        'THESE FEATURES'            # 不確実性係数（非対称な指標である点に注意）
                    ]):
                        is_provides_block = True
                    elif any(key in elem.get_text() for key in [
                        # 以下のブロックは無視
                        'SMALLEST VALUES',
                        'LARGEST VALUES',
                        'MOST FREQUENT VALUES',
                    ]):
                        is_provides_block = False

            
            elif is_provides_block and elem.name == 'div' and 'assoc-row' in elem.get('class', []):
                child_elem = elem.find('div', class_='pos-assoc-row__label').get_text(strip=True)
                value_elem = elem.find('div', class_='pos-assoc-row__source').get_text(strip=True)
                try:
                    value_elem = float(value_elem)
                    data.append([parent_name, child_elem, value_elem])
                except ValueError:
                    continue  # 数値変換できなかったらスキップ  
    return data

```
- 3. データフレームに変換して整理
```py:ipynbファイル
def to_dataframe(data):
    
    # DataFrameへ変換
    df = pd.DataFrame(data, columns=["項目１", "項目２", "相関値"])
    # 要素1と要素2を順不同で扱うため、並び替えた新しい列を作成
    df["組み合わせ"] = df.apply(lambda row: tuple(sorted([row["要素１"], row["要素２"]])), axis=1)
    # 重複排除（組み合わせと相関値が同じ行を1つだけ残す）
    df = df.drop_duplicates(subset=["組み合わせ", "相関値"]).drop(columns="組み合わせ")

    # csv出力
    df.to_csv(associationsCsvfile_path, index=False)
    return df
```
- 4. ついでに...データの概況をデータフレームにする
```py:ipynbファイル
def get_dataOverview(soup, overviewCsvfile_path):
    # 出力リスト
    data = []

    # ic-cat および ic-numeric の div タグを走査して、それに続く span と stats を取得
    for div in soup.find_all("div", class_=["pos-tab-image ic-cat", "pos-tab-image ic-numeric"]):
        # データ型判定
        if "ic-cat" in div["class"]:
            data_type = "c"  # categoricalの意味
        elif "ic-numeric" in div["class"]:
            data_type = "n"  # numericの意味
        else:
            data_type = "u"  # unknownの意味

        # 直後の span（項目名）
        span = div.find_next("span", class_="text-title-tab color-normal")
        if not span:
            continue
        item_name = span.get_text(strip=True)

        # 対応する統計情報（div class="pos-base-stats"）
        stats_div = span.find_next("div", class_="pos-base-stats")

        # 初期化
        missing = distinct = zeroes = 0

        if stats_div:
            for row in stats_div.find_all("div", class_="base-stats-row"):
                label = row.find("div", class_="text-label color-normal pos-base-stats__label")
                if not label:
                    continue
                label_text = label.get_text(strip=True)

                # 値の取得（text-value または text-distinct）
                # value_div = row.find("div", class_="text-value") or row.find("div", class_="text-distinct")pos-base-stats__source
                value_div = row.find("div", class_="pos-base-stats__source")
                value = value_div.get_text(strip=True) if value_div else '0'

                # 値が '---' または空なら 0、カンマは除去して整数に変換
                if value == '---' or value == '':
                    value = 0
                else:
                    # 複数行がある場合は最初の行だけ使う
                    first_line = value.strip().splitlines()[0]
                    # 数字とカンマ以外を除去（念のため）
                    cleaned = ''.join(c for c in first_line if c.isdigit() or c == ',')
                    value =  int(cleaned.replace(',', '')) if cleaned else 0


                # 各統計に格納
                if label_text == "MISSING:":
                    missing = value
                elif label_text == "DISTINCT:":
                    distinct = value
                elif label_text == "ZEROES:":
                    zeroes = value

        # データとして追加
        data.append({
            "項目": item_name,
            "DATA_TYPE": data_type,
            "MISSING": missing,
            "DISTINCT": distinct,
            "ZEROES": zeroes
        })

    # DataFrame化
    df = pd.DataFrame(data)

    # CSV出力
    df.to_csv(overviewCsvfile_path, index=False)
    return df
```

### 結果の出力イメージ
- コード実行
```py:ipynbファイル
associationsCsvfile_path = './sweetviz_associations.csv'       # 相関図をcsvにする時のパス（ファイル名）を設定
overviewCsvfile_path = './sweetviz_overview.csv'               # データの概況をcsvにする時のパス（ファイル名）を設定

export_sweetvizReport_for1data([], [], original_df, htmlfile_path1, "")
soup = read_html(htmlfile_path1)

associations = get_associations(soup, original_df, associationsCsvfile_path)
overview = get_dataOverview(soup, overviewCsvfile_path)
```
- 出力結果
![](/images/20250520_eda_scrapping/output.png =500x)


### 参考：sweetvizで出力される相関値について
sweetvizでは、変数間の関係性を可視化する「相関値」が自動で出力されますが、以下の通り***変数の型の組み合わせによって使用される指標が異なります**。

|  相関の種類                | 使用される指標 | 値の範囲 | 値の範囲 |
|     ----                   |  ----          |  ----    |  ----    |
|  数値型同士の相関          | Pearson相関係数|  -1～1   |2つの量的変数間の<br>直線的関連の程度を表す係数。  |
|  数値型とカテゴリ型の相関  | 相関比         |   0～1   |数値変数の分散がカテゴリによって<br>説明される程度を表す係数。
|  カテゴリ型同士の相関      | 不確実性係数   |   0～1   |一方から他方が予測可能になる程度を<br>表す。非対称な情報量指標。|


#### 注意点
- 全ての変数ペアに対して相関が出るわけではありません。
**欠損値が多いカラムやユニークな値が多いカテゴリ型のカラムは、計算対象から除外されることがあります**。
- 外れ値や欠損の影響に注意しましょう。
外れ値によってピアソン相関が過大評価されることがあります。**データクレンジング後にも再度EDAを行うことを推奨します**。
- 相関が高いからと言って因果関係があるとは限りません。
例えば「アイスの売上」と「熱中症患者数」は高い相関を持つかもしれません。しかし、これはどちらかが原因ではなく「気温」という**共通の要因に左右されている**のです。


## さいごに
大量カラムのEDAは、着手のハードルが高く感じがちですが、「ライブラリ選定」と「少しの工夫（スクレイピング）」 を組み合わせることで、ある程度楽にこなせるようになります。特にsweetvizは、比較ができる・処理が軽いなど魅力が多いので、「相関図の文字が潰れる問題」を**補完しながら活用する**のがおすすめです。

