---
title: "デイトレーダーがチャートと出来高・株価から、pythonで投資銘柄を選定する方法１"
emoji: "💹"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["OCR", "Docker", "python", "株", "スクリーニング"]
published: true
---
## はじめに
こんにちは、日本株の個別株投資歴6年目のマチです。デイトレーダーやスイングトレーダーの皆さんは、どのように投資対象の銘柄を選んでいますか。私は「よく投資する銘柄リスト」から時間の許す限りチャートを眺めて決めていました。

本当はちゃんと、出来高と株価などでスクリーニングして、さらにチャートの形状で銘柄を絞り込みたいのですが、2025年7月現在では**チャートと出来高・株価などを組み合わせてスクリーニングできるサイトはあまり見かけない**ですよね。**だから作りました**。今回は、私が作ったスクリーニングのプログラムを紹介します。手順のイメージは以下の通りです。手順４の「目視による厳選」では、**松井証券のツールを使用します（証券口座の開設が必要です）**。
![](/images/20250712_tech_investor_selectcode/allSteps.png =800x)

## 1.環境構築
Dockerでpythonを扱える環境を構築します。手順は、記事「[【図解】Windows11でWSL2＋DockerによるPython開発環境を構築する手順](https://zenn.dev/stockdatalab/articles/20250519_tech_env_docker)」を参照ください。Dockerfileなどの資材は以下を使用しました。
```sh:Dockerfile
#ベースイメージ
FROM jupyter/base-notebook:python-3.10.10

# rootユーザーに切り替え
USER root

# システム依存パッケージのインストール（OCR用）
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    tesseract-ocr \
    tesseract-ocr-jpn \
    poppler-utils \
    libglib2.0-0 \
    libgl1 \
    libsm6 \
    libxext6 \
    libxrender-dev \
    fonts-ipafont-gothic \
    && apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# ノートブックユーザーに戻す（安全対策）
USER ${NB_UID}

#ローカルのrequirements.txtを、コンテナ内にコピーしてライブラリをインストール
COPY requirements.txt ./
RUN pip3 install --no-cache-dir -r requirements.txt
```
```sh:requirements.txt
setuptools
numpy
pandas
openpyxl
xlrd
yfinance

pillow
pytesseract
pdf2image
opencv-python
```
```sh:docker-compose.yaml
services:
  jupyterlab:
    build: 
      context: .
      dockerfile: ./Dockerfile
    image: zenn0
    container_name: zenn0_c
    ports:
      - "127.0.0.1:8888:8888"   #jupyter用
    working_dir: /home/jovyan/work
    volumes:
      - ./source:/home/jovyan/work    #ローカルとコンテナ環境をマウントする
    environment:
      TZ: "Asia/Tokyo"
    tty: true
    restart: always
```

### ここまでの進捗
![](/images/20250712_tech_investor_selectcode/step1.png =400x)


## 2.データ収集（手動）
残念ながら、データ収集の一部は手動です。スクレイピングやクローリングも検討しましたが、サーバーに対する負荷等の都合で、Webページの運営側が規約などで禁止するケースが増えている（今は言及されていなくても今後禁止される可能性が高い）ことから、不採用としました。

### 2-1.銘柄の一覧を取得
以下のいずれかの方法で銘柄の一覧を取得します。前者の方がプログラムで処理するデータ量が少なくなり、実行時間を抑えられるのでおすすめです。

- **SBI証券でスクリーニングした結果をCSVダウンロードする**
[SBI証券のスクリーニング機能](https://search.sbisec.co.jp/v3/ex/domestic_screening.html)（**使用には証券口座の開設が必要**）であれば、株価で絞り込んだ銘柄の一覧をCSVダウンロードできます。2025年7月現在は出来高での絞り込みはできなさそうなので、後述のプログラムで実行します。

- **全銘柄のデータをダウンロードする**
[日本取引所のサイト](https://www.jpx.co.jp/markets/statistics-equities/misc/01.html)からxlsファイルをダウンロードします。月１の頻度で更新されます。



### 2-2.チャートのスクショを取得
チャートの形状でスクリーニングできるサイトから、得意なチャートパターンのwebページ全体のスクショを取得します。サイトは好きなものを使ってください。私は、[株マップ.com](https://jp.kabumap.com/servlets/kabumap/Action?SRC=chartShape/base)を使用しています。後述のプログラムも、株マップ.comのwebページをベースにしています。スクショを撮る際は、[**Chromeの全画面スクショ機能**](https://note.com/pasokonta/n/n92a9758998b4)を使うのがおすすめです。

### ここまでの進捗
![](/images/20250712_tech_investor_selectcode/step2.png =800x)


## 3.プログラム作成・実行（スクリーニング）
プログラムは以下の通り、大きく３つのステップで構成されます。
![](/images/20250712_tech_investor_selectcode/program.png =800x)
フォルダ構成は以下の通りです。「input」フォルダ配下に、収集した銘柄一覧（screener_sbi.csv または data_j.xls）とスクショを配置します。個人的な好みにつき、スクショはチャートの形状パターンごとにフォルダ（フォルダ名に「long」または「short」の文字列を含む）を分けています。
```txt:フォルダ構成
work
│  docker-compose.yaml
│  Dockerfile
│  requirements.txt
│  
└─source
    │  make_master.ipynb
    │  screening.ipynb
    │  
    ├─input
    │  │  data_j.xls
    │  │  screener_sbi.csv
    │  │  
    │  └─charts
    │      ├─long_ボックス下限
    │      │  ├─XXX.png
    │      │  ├─...
    │      ├─...
    │      └─short_高値反落
    │          ├─XXX.png
    │          ├─...
    └─output
```
### 3-1.出来高・株価でスクリーニング
make_master.ipynbを作成します。このプログラムでは、銘柄一覧とyfinanceを使って出来高・株価でスクリーニングした結果を、「output」フォルダ配下にmaster.csvとして出力します。銘柄一覧のデータ量やスクリーニングの条件に依ると思いますが、実行時間は30分程度でした。
:::details make_master.ipynbのコード
```py:make_master.ipynb
import pandas as pd
from datetime import datetime
import yfinance as yf
from tqdm import tqdm 

# # 使用するデータに合わせて、以下のいずれかを実行する------------------------
# SBI証券でスクリーニングした結果を読み込む
df = pd.read_csv("./input/screener_sbi.csv", encoding="utf_8") #投資金額：10万～60万

# 東証上場銘柄一覧を読み込む
# df = pd.read_excel("./input/data_j.xls")
# df = df[df["市場・商品区分"].str.contains("内国株式", na=False)]
# ----------------------------------------------------------------------------

# 使用する列に絞る
df = df[["コード", "銘柄名"]]
df["yf_code"] = df["コード"].astype(str).str.zfill(4) + ".T"

# yfinance で情報取得（30min）
results = []
for _, row in tqdm(df.iterrows(), total=len(df), desc="処理中"):
    code = row["yf_code"]
    name = row["銘柄名"]

    try:
        ticker = yf.Ticker(code)
        info = ticker.info

        # 最新日付の株価（1日のデータ）
        hist = ticker.history(period="1d")

        result = {
            "コード": code,
            "銘柄名": name,
            "始値": hist["Open"].iloc[-1] if not hist.empty else None,
            "高値": hist["High"].iloc[-1] if not hist.empty else None,
            "安値": hist["Low"].iloc[-1] if not hist.empty else None,
            "終値": hist["Close"].iloc[-1] if not hist.empty else None,
            "出来高": hist["Volume"].iloc[-1] if not hist.empty else None,
            "カテゴリ": info.get("industry"),
            "時価総額": info.get("marketCap"),
            "決算日": info.get("nextEarningsDate"),
            "配当金額": info.get("dividendRate"),
            "権利付き最終日": info.get("exDividendDate"),
        }

        results.append(result)

    except Exception as e:
        print(f"{code} 取得失敗: {e}")

# 結果をデータフレームに変換して、投資候補の銘柄に絞る
# 投資金額：10万～60万、出来高：10万以上
result_df = pd.DataFrame(results)
result_df = result_df[
    (result_df["安値"] >= 1000) &
    (result_df["高値"] <= 6000) &
    (result_df["出来高"] >= 100000)
]

# UNIXタイムスタンプを日付に変換
result_df["権利付き最終日"] = pd.to_datetime(
    result_df["権利付き最終日"], 
    unit='s', 
    errors='coerce'  # 無効な値は NaT に
)
# 日付のフォーマットを文字列にする
result_df["権利付き最終日"] = result_df["権利付き最終日"].dt.strftime('%Y-%m-%d')

# 表示と保存
result_df.to_csv("./output/master.csv", index=False)
print(len(result_df))
display(result_df.head())
```
:::
![](/images/20250712_tech_investor_selectcode/step3-1.png =600x)
### 3-2.チャートを取得した銘柄をOCRで一覧にする
screening.ipynbの前半を作成します。まずは、スクショに対して二値化などの前処理を行います。今回のように二値化した結果が、**黒い背景に白字になる場合はOCRの精度が下がることがあるので、白黒反転させます**。次に、チャートが表示される場所（座標）が固定されていることに留意して、チャート上部のヘッダー（銘柄コードと銘柄名の部分）を切り取ってノイズを除去します。その後、OCRで銘柄コードを読み取ります。銘柄コードを読み取る際は**銘柄名はノイズとなるので、OCRを実行する際に数値のみ認識するよう設定**します。
:::details screening.ipynbの前半のコード
```py:screening.ipynb（前半）
import os
import re
from tqdm import tqdm
import numpy as np
import pandas as pd
pd.set_option('display.max_rows', 100)
from datetime import date, timedelta
import yfinance as yf

import cv2
import pytesseract
from pathlib import Path
from PIL import Image, ImageEnhance, ImageFilter


# 関数を定義-----------------------------------------------
def rename_pngs(folder: Path):
    """
    folder配下のpngを、input_{i}.pngにrenameする
    """
    #不要なファイルがあれば削除
    del_files = sorted(folder.glob("*Zone.Identifier"))
    list(map(lambda f: f.unlink(), del_files))

    # ファイル名をrename
    png_files = sorted(folder.glob("*.png"))
    list(map(
        lambda pair: pair[0].rename(folder / f"input_{pair[1]}.png"),
        zip(png_files, range(len(png_files)))
    ))



def trimming_pngs(target_folders: list[str]):
    """
    folder配下の「long」「short」を含むフォルダ配下のpngを、白黒処理・トリミング処理をして、processed1_{i}.pngにrenameする
    """
    # トリミング領域リスト（left, upper, right, lower）
    crop_boxes = [
        (25, 450, 325, 485),  (355, 450, 645, 485),  (675, 450, 975, 485),
        (25, 680, 325, 715),  (355, 680, 645, 715),  (675, 680, 975, 715),
        (25, 910, 325, 945),  (355, 910, 645, 945),  (675, 910, 975, 945),
        (25, 1140, 325, 1175)
    ]

    for folder in target_folders:
        print(f"処理対象フォルダ: {folder.name}")
        
        # input_*.png ファイルを順に処理
        input_files = sorted(folder.glob("input_*.png"))
        
        for input_path in tqdm(sorted(input_files), desc=f"{folder.name} の処理", unit="ファイル"):
            stem = input_path.stem  # 例: input_0

            # 1. PIL → numpy配列 → グレースケール変換
            im_pillow = Image.open(input_path).convert("RGB")
            im_np = np.array(im_pillow)
            gray = cv2.cvtColor(im_np, cv2.COLOR_RGB2GRAY)

            # 2. Otsuによる白黒化
            _, thresh_img = cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)

            # 3. 白黒反転
            inverted = cv2.bitwise_not(thresh_img)

            # 4. 一時的に保存（Pillowで開くため）
            temp_path = folder / f"{stem}_inverted_temp.png"
            cv2.imwrite(str(temp_path), inverted)

            # 5. PILで開いてトリミング
            image = Image.open(temp_path)

            for i, crop_box in enumerate(crop_boxes):
                cropped = image.crop(crop_box)
                output_path = folder / f"{stem.replace('input_', 'processed1_')}_{str(i).zfill(3)}.png"
                cropped.save(output_path)

            # 一時ファイル削除
            temp_path.unlink()



def extract_ocr_results_with_names(target_folders):
    """
    OCRを実行し、フォルダ名とOCR結果リストをタプルで返す。
    Parameters: target_folders (list[Path]): 処理対象フォルダ一覧
    Returns: list[tuple[str, list[str]]]: フォルダ名とOCR結果のリスト
    """
    # OCR設定：数字のみに限定
    custom_config = r'--oem 3 --psm 6 -c tessedit_char_whitelist=0123456789'
    results = []  # [(フォルダ名, [ocr結果...]), ...]

    for folder in target_folders:
        png_files = sorted(folder.glob("processed1_*.png"))
        result = []  # このフォルダ内のOCR結果（先頭4文字を格納用）

        for png_path in tqdm(png_files, desc=f"{folder.name} の処理", unit="ファイル"):
            stem = png_path.stem

            # 画像読み込みと前処理
            image = Image.open(png_path)
            upscale_factor = 5
            image = image.resize((image.width * upscale_factor, image.height * upscale_factor), Image.LANCZOS)  # 解像度調整
            image = image.filter(ImageFilter.MedianFilter(size=5))  # ノイズ除去
            image = ImageEnhance.Contrast(image).enhance(3)  # コントラスト強調
            image = image.point(lambda x: 255 if x > 180 else 0).convert('1')  # 二値化

            # 補正済み画像の保存
            processed_path = folder / f"{stem.replace('processed1_', 'processed2_')}.png"
            image.save(processed_path)

            # OCR実行・補正
            text = pytesseract.image_to_string(image, config=custom_config)
            text = text.replace('「', '').replace('」', '').replace('|', '').replace('[|]', '').replace(']', '')
            text = re.sub(r'[ 　]', '', text).strip()

            # 最大4文字格納
            result.append(text[:4] if len(text) >= 4 else text)

        # このフォルダの結果を (フォルダ名, OCRリスト) で格納
        result = [x for x in result if x.strip() != ""]
        result = [x + ".T" for x in result]
        results.append((folder.name, result))

    return results



def display_result(i:int, results:list[tuple[str, list[str]]]):
    """
    フォルダ名とOCR結果リストを表示する。
    """
    folder_name, ocr_list = results[i]
    master_df = pd.read_csv("./output/master.csv", dtype=str, encoding="utf_8") #投資金額：10万～60万・出来高10万以上
    filtered_df = master_df[master_df["コード"].isin(ocr_list)].sort_values(by="出来高", ascending=False)

    # 1. 欠損・空文字を "0" にし、"." 以降を削除 → カンマ削除 → 整数変換
    filtered_df["出来高"] = (
        filtered_df["出来高"]
        .fillna("0")                            # None → "0"
        .replace("", "0")                       # 空文字 → "0"
        .str.split(".").str[0]                  # 小数点以降を削除
        .str.replace(",", "", regex=False)      # カンマ除去
        .astype(int)                            # 整数に変換
    )

    # 2. 出来高の降順でソート
    filtered_df = filtered_df.sort_values(by="出来高", ascending=False)

    if len(filtered_df)!=0:
        filtered_df.to_csv(f"./output/chart1_{folder_name}.csv", index=False)
    print(f"\n{folder_name} ：{len(filtered_df)}件")
    display(filtered_df)
# -----------------------------------------------------------

# 「./input」配下の「long」または「short」を含むフォルダを取得
base_dir = Path("./input/charts")
target_folders = list(filter(lambda d: d.is_dir() and ('long' in d.name or 'short' in d.name), base_dir.iterdir()))

# input_{i}.pngにrenameする
list(map(rename_pngs, target_folders))
print("ファイル名の変更処理が完了しました。")

# input_{i}.pngに対して、白黒処理・トリミング処理をして、processed1_{i}.pngにrenameする
trimming_pngs(target_folders)
print("トリミング処理が完了しました。")

# processed1_{i}.pngに対して、OCRを実行
results = extract_ocr_results_with_names(target_folders)
print("OCR処理が完了しました。")

# OCRの結果を表示
for folder_name, ocr_list in results:
    print(f"\n{folder_name} （{len(ocr_list)}件）：\n{ocr_list}")
    
print("すべての処理が完了しました。")

```
:::
![](/images/20250712_tech_investor_selectcode/step3-2.png =450x)
### 3-3.3-1と3-2のデータを突合させる
screening.ipynbの後半を作成します。3-1と3-2のデータを突合させて両方に含まれる銘柄コードのみ抜粋します。ついでに、直近2週間のチャートの上ヒゲと下ヒゲが長すぎる銘柄を（値動きに安定感がないため）、投資候補から外します。
:::details screening.ipynbの後半のコード
```py:screening.ipynb（後半）
# 関数を定義-----------------------------------------------
def display_extract_code(i:int, ocr_results:list[tuple[str, list[str]]]):
    """
    髭の長さで銘柄を絞り込み、フォルダ名とOCR結果リストを表示する。
    """
    folder_name, ocr_list = ocr_results[i]
    master_df = pd.read_csv("./output/master.csv", dtype=str, encoding="utf_8") #投資金額：10万～60万・出来高10万以上
    filtered_df = master_df[master_df["コード"].isin(ocr_list)].sort_values(by="出来高", ascending=False)
    
    if filtered_df.empty:
        print(f"[{folder_name}] 該当する銘柄が master.csv に存在しません。スキップします。")
    else:

        # yfinance で情報取得
        yf_results = []
        for _, row in tqdm(filtered_df.iterrows(), total=len(filtered_df), desc="処理中"):
            code = row["コード"]
            name = row["銘柄名"]

            try:
                ticker = yf.Ticker(code)
                info = ticker.info

                # 直近２週間分の株価
                today_date = date.today()
                two_weeks_ago_date = today_date - timedelta(days=14)
                today = today_date.strftime("%Y-%m-%d")
                two_weeks_ago = two_weeks_ago_date.strftime("%Y-%m-%d")
                hist = yf.download(tickers=code, start=two_weeks_ago, end=today, auto_adjust=True)
                if isinstance(hist.columns, pd.MultiIndex):
                    hist.columns = hist.columns.get_level_values(0)


                if hist.empty:
                    continue

                hist = hist.reset_index()  # 日付を列に
                hist["コード"] = code
                hist["銘柄名"] = name
                hist["カテゴリ"] = info.get("industry")
                hist["時価総額"] = info.get("marketCap")
                hist["決算日"] = info.get("nextEarningsDate")
                hist["配当金額"] = info.get("dividendRate")
                hist["権利付き最終日"] = info.get("exDividendDate")

                yf_results.append(hist)
                
            except Exception as e:
                print(f"{code} 取得失敗: {e}")

        # 全ての履歴を結合して整形
        df = pd.concat(yf_results, ignore_index=True)
        df = df.rename(columns={
            "Date": "日付",
            "Open": "始値",
            "High": "高値",
            "Low": "安値",
            "Close": "終値",
            "Volume": "出来高"
        })
        df = df[['コード', '銘柄名', '日付', '高値', '安値', '始値', '終値', '出来高', 'カテゴリ', '時価総額', '決算日', '配当金額', '権利付き最終日']]
        df['始値'] = pd.to_numeric(df['始値'], errors='coerce')
        df['終値'] = pd.to_numeric(df['終値'], errors='coerce')
        df['高値'] = pd.to_numeric(df['高値'], errors='coerce')
        df['安値'] = pd.to_numeric(df['高値'], errors='coerce')

        # 上ヒゲの長さ: 高値 - max(始値, 終値)
        df['上ヒゲ'] = df['高値'] - df[['始値', '終値']].max(axis=1)

        # 下ヒゲの長さ: min(始値, 終値) - 安値
        df['下ヒゲ'] = df[['始値', '終値']].min(axis=1) - df['安値']

        # 終値と始値の差の絶対値
        df['実体'] = (df['終値'] - df['始値']).abs()

        # 投資対象か否か判定する材料
        df['投資flg'] = df.apply(
            lambda row: 0 if (row['上ヒゲ'] - row['実体'] > 0) and (row['下ヒゲ'] - row['実体'] > 0) else 1,
            axis=1
        )

        # 投資対象か否か判定する
        # 上下のひげが「実体」より短いものが6割以上ある銘柄を投資対象とする
        counts = df.groupby('コード')['投資flg'].sum().reset_index()
        valid_codes = counts[counts['投資flg'] >= 6]['コード']
        df = df[df['コード'].isin(valid_codes)]

        # 欠損・空文字を "0" にし、"." 以降を削除 → カンマ削除 → 整数変換
        df["出来高"] = (
            df["出来高"]
            .fillna("0")                                  # None → "0"
            .replace("", "0")                             # 空文字 → "0"
            .astype(str)                                  # 念のため文字列化
            .str.replace(",", "", regex=False)            # カンマ削除
            .str.replace(r"\..*$", "", regex=True)        # 小数点以降を削除（例：123.45 → 123）
            .astype(int)                                  # 整数に変換
        )

        # UNIXタイムスタンプを日付に変換
        df["権利付き最終日"] = pd.to_datetime(
            df["権利付き最終日"], 
            unit='s', 
            errors='coerce'  # 無効な値は NaT に
        )
        df["権利付き最終日"] = df["権利付き最終日"].dt.strftime('%Y-%m-%d')

        # 最新の日付の情報のみ抽出して、出来高の降順でソート
        df = df[df["日付"] == df["日付"].max()]
        df = df.sort_values(by="出来高", ascending=False)
        df = df.drop(columns=['上ヒゲ', '下ヒゲ', '実体', '投資flg'])

        # 出力と表示
        if len(df)!=0:
            df.to_csv(f"./output/chart2_{folder_name}.csv", index=False)
            # 「コード」列だけをコピーして加工（元のdfは変更しない）
            # 加工後のコード列のみを出力（ヘッダーなし）
            code_series = df['コード'].str.replace(r"\.T$", "", regex=True)
            code_series.to_frame().to_csv(f"./output/MATSUI_chart2_{folder_name}.csv", index=False, header=False)

        print(f"\n{folder_name} ：{len(df)}件")
        display(df)
# -----------------------------------------------------------
display_extract_code(0, results)
display_extract_code(1, results)
# ...チャートのスクショを格納しているフォルダの数だけ繰り返す
```
:::
### ここまでの進捗
![](/images/20250712_tech_investor_selectcode/step3.png =800x)


## 4.目視による厳選
松井証券のツールを使用すると、データをインポートして、複数のチャートを少ないクリック数で見れます。また、日足・週足などの時間軸を変えられる点も良いです。**ツールの使用には証券口座の開設が必要です**。
### 4-1.スクリーニングデータの読み込み
[松井証券のツール](https://www.matsui.co.jp/tool/ns-highspeed/)に、スクリーニングしたデータを読み込ませます。
![](/images/20250712_tech_investor_selectcode/inport.png =400x)


### 4-2.チャート形状を確認
損するリスクが低そうな銘柄をチャートの形状をもとに、厳選します。
![](/images/20250712_tech_investor_selectcode/check_charts.png =500x)


### 4-3.当日朝の板情報を確認
当日朝の板情報から売り優勢か買い優勢か見極めて、投資する銘柄をさらに絞り込みます。

### 選定完了！
![](/images/20250712_tech_investor_selectcode/allSteps.png =800x)


## おわりに
このプログラムのおかげで、数時間かけていた銘柄の選定作業が、45分程度（=スクショ取得15分 + チャート形状の目視による絞り込み15分 + 当日朝の板情報による絞り込み15分；プログラミング実行中の待ち時間は含まない）まで抑えることができました。今のところ選定した銘柄で利益を出すことができています。今回した記事が、皆さんの銘柄選定作業の一助となれば幸いです。（プログラムなどの紹介内容の利用に伴う責任は、こちらでは一切負いません。**投資は自己責任でお願いします**。）
