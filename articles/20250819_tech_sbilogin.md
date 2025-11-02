---
title: "Dockerコンテナ上でSeleniumを動かしてSBI証券に自動ログインする（Chrome・デバイス認証対応）"
emoji: "🦁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Docker", "Selenium", "Chrome", "SBI", "認証"]
published: true
---
## はじめに
証券会社のログインは、セキュリティ対策として「デバイス認証」が導入されている場合が多く、単純にSeleniumでスクリプト化しても、端末の識別情報がリセットされると毎回認証が求められ、認証のたびに手作業が必要になるのが課題です。本記事では、**Dockerコンテナ上にChromeをインストールし、Seleniumで自動操作を行いつつ、認証済みプロファイルを再利用することで「初回のみのデバイス認証」で済む仕組み**を紹介します。

## 1. Docker環境を構築する
Dockerでpythonを扱える環境を構築します。手順は、記事「[【図解】Windows11でWSL2＋DockerによるPython開発環境を構築する手順](https://zenn.dev/stockdatalab/articles/20250519_tech_env_docker)」を参照ください。Dockerfileなどの資材は以下を使用しました。

ポートなどの設定をしない限り、**Seleniumは環境（例：ローカル環境・Linux環境・Dockerコンテナ環境）を跨いでブラウザを操作することができません**。今回はDockerコンテナ上でSeleniumを動かすので、コンテナにChromeをインストールします。

#### 環境イメージ
![](/images/20250819_tech_sbilogin/env.png =400x)


```sh:Dockerfile
#ベースイメージ
FROM jupyter/base-notebook:python-3.10.10

# rootユーザーに切り替え（apt使用のため）
USER root


# 必要ユーティリティ
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates curl wget gnupg unzip \
 && rm -rf /var/lib/apt/lists/*

# --- Google Chrome リポジトリを keyrings 方式で追加（apt-key は使わない） ---
RUN install -m 0755 -d /etc/apt/keyrings \
 && curl -fsSL https://dl.google.com/linux/linux_signing_key.pub \
    | gpg --dearmor -o /etc/apt/keyrings/google-linux.gpg \
 && chmod a+r /etc/apt/keyrings/google-linux.gpg \
 && echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/google-linux.gpg] http://dl.google.com/linux/chrome/deb/ stable main" \
    > /etc/apt/sources.list.d/google-chrome.list

# Google Chrome 本体をインストール
RUN apt-get update && apt-get install -y --no-install-recommends \
    google-chrome-stable \
 && rm -rf /var/lib/apt/lists/*

# パスを固定
RUN ln -sf /usr/bin/google-chrome /usr/bin/chrome

# システム依存パッケージのインストール（日本語フォント用）
RUN apt-get update && \
    apt-get install -y --no-install-recommends \

    fonts-ipaexfont \
    fontconfig \

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
#requirements.lockを使用する場合は、requirements.txtをrequirements.lockに置換して同様の記載でdockerfileを実行
COPY requirements.txt ./
RUN pip3 install --no-cache-dir -r requirements.txt
```
```sh:requirements.txt
selenium
beautifulsoup4
webdriver_manager
```
```sh:docker-compose.yamlの例
services:
  jupyterlab:
    build: 
      context: .
      dockerfile: ./Dockerfile
    image: zenn0
    container_name: zenn0_c
    ports:
      - "127.0.0.1:8888:8888"   #jupyter用
      - "127.0.0.1:5000:5000"   #mlflow用
      - "127.0.0.1:8501:8501"   #pygwalker用
    working_dir: /home/jovyan/work
    volumes:
      - ./source:/home/jovyan/work    #ローカルとコンテナ環境をマウントする
    environment:
      TZ: "Asia/Tokyo"
    tty: true
    restart: always
```

次に、**デバイス認証済みのプロファイルを保存する場所を作ります**。Docker＋Selenium は標準では空の一時プロファイルで起動するため、端末識別に使われる情報が毎回リセットされます。SBI証券のデバイス認証は、端末判定にログイン用プロファイル情報を使います。これらが毎回リセットされるとサイト側からは「初見の端末」に見え、毎回デバイス認証が必要になります。プロファイルを保存することで、初回だけデバイス認証を実施すれば良いようになります。
**作成したフォルダは、gitコミット対象外にしておきましょう**。ログイン処理を自動化するとともにセキュリティーリスクを最小限に抑えることができます。
```sh:Ubuntu-24.04アプリ（Bash）
mkdir -m 777 ChromeProfileforSelenium
```
```txt:フォルダ構成
work
│  docker-compose.yaml
│  Dockerfile
│  requirements.txt
│  
└─source（マウント場所）
      ChromeProfileforSelenium
      master.ipynb
```


## 2. デバイス認証する（初回のみ）
デバイス認証やログインの処理を実装します。.envファイルにログインパスワードなどを記載し、gitコミット対象外にしておくことで、ログイン処理を自動化するとともにセキュリティーリスクを最小限に抑えることができます。
```sh:.env
CHROME_BIN=/usr/bin/chrome
SE_PROFILE=./ChromeProfileforSelenium
SBI_USER=12301234567 # 自分のユーザーネーム
SBI_PASS=ABCDE! # 自分のログインパスワード
```
デバイス認証の流れは下図の通りです。2-1の処理完了後から2-3の処理開始までは、**2-2でメールのリンク先のページにデバイス認証のためのコードを入力するため、長めに45秒程待機するようにします**。2-2で入力するコードは、2-1の処理完了時に取得したスクショを参照します。なお、その他ページ遷移のたび短時間待機しているのは、サイトのサーバーに過度に負荷をかけないようにするためです。
![](/images/20250819_tech_sbilogin/login.png =800x)
:::message
2025/10/25にログイン処理の仕様が変更されました。それに伴い、URLやhtmlの要素名の変更・UAの指定などが必要となりました。UAで指定するブラウザのバージョンは最新のものにします。
:::
:::details master.ipynb（2025/10/25以降）
```py:master.ipynb
import os, getpass, time
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.common.by import By

options = Options()

# コンテナ内の Chromium 実行ファイルを指定
chrome_bin = os.environ.get("CHROME_BIN", "/usr/bin/chromium")
options.binary_location = chrome_bin

# ヘッドレス + ルート起動向けの安定化フラグ
options.add_argument("--headless=new")
options.add_argument("--no-sandbox")
options.add_argument("--disable-dev-shm-usage")
options.add_argument("--disable-gpu")
options.add_argument("--window-size=1920,1080")

#SBI側の仕様変更により、2025/10/25以降指定必須（ブラウザのバージョンは最新のものを指定）
options.add_argument("--user-agent=Mozilla/5.0 (X11; Linux x86_64) "
                        "AppleWebKit/537.36 (KHTML, like Gecko) "
                        "Chrome/142.0.0.0 Safari/537.36")  


# デバイス認証済みのプロファイルが保存される場所を読み込ませる
se_profile = os.getenv("SE_PROFILE")
options.add_argument(f'--user-data-dir={se_profile}')


# ドライバは webdriver_manager に任せる（ブラウザ本体は上で用意済み）
service = Service(ChromeDriverManager().install())
driver = webdriver.Chrome(service=service, options=options)

# UAを確認
ua = driver.execute_script("return navigator.userAgent;")
print("Current User-Agent:", ua)


# SBI証券のページへ遷移
url = "https://login.sbisec.co.jp/login/entry"
driver.get(url)
time.sleep(3) # 画面が描画されるまで待機

# ユーザーネームを入力
username_value = os.getenv("SBI_USER")
username = driver.find_element(By.NAME, "username")
username.send_keys(username_value)

# パスワードを入力
password_value = os.getenv("SBI_PASS")
password = driver.find_element(By.NAME, "password")
password.send_keys(password_value)

# ログインボタンをクリック
driver.find_element(By.ID, "pw-btn").click()
time.sleep(5) # 画面が描画されるまで待機

# ------------------初回ログイン時のみ実行（ここから）---------------------------------
# デバイス認証（メール送信）
driver.find_element(By.ID, "sendEmailButton").click()
time.sleep(2) # 画面が描画されるまで待機
driver.save_screenshot('./selenium_screenshot_code.png') # デバイス認証のための入力コードが記載されている画面

# デバイス認証（チェックボタン押下・ボタン押下）
driver.find_element(By.ID, "authCheck").click() # 認証完了確認のチェック
time.sleep(45) # 画面が描画される （ボタンが活性化して押下できるようになる）・メールから認証完了するまで待機
driver.find_element(By.ID, "otpRegisterButton").click() # 認証完了確認のボタン押下
time.sleep(15) # 自動で画面遷移するのを待つ
# ------------------初回ログイン時のみ実行（ここまで）---------------------------------

# ログアウトボタンをクリック
driver.find_element(By.LINK_TEXT, "ログアウト").click()
time.sleep(5)

driver.quit()
```
:::
:::details master.ipynb（2025/10/25以前）
```py:master.ipynb
import os, getpass, time
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.common.by import By

options = Options()

# コンテナ内の Chromium 実行ファイルを指定
chrome_bin = os.environ.get("CHROME_BIN", "/usr/bin/chromium")
options.binary_location = chrome_bin

# ヘッドレス + ルート起動向けの安定化フラグ
options.add_argument("--headless=new")
options.add_argument("--no-sandbox")
options.add_argument("--disable-dev-shm-usage")
options.add_argument("--disable-gpu")
options.add_argument("--window-size=1920,1080")


# デバイス認証済みのプロファイルが保存される場所を読み込ませる
se_profile = os.getenv("SE_PROFILE")
options.add_argument(f'--user-data-dir={se_profile}')


# ドライバは webdriver_manager に任せる（ブラウザ本体は上で用意済み）
service = Service(ChromeDriverManager().install())
driver = webdriver.Chrome(service=service, options=options)


# SBI証券のページへ遷移
url = "https://www.sbisec.co.jp/ETGate"
driver.get(url)
time.sleep(3) # 画面が描画されるまで待機

# ユーザーネームを入力
username_value = os.getenv("SBI_USER")
username = driver.find_element(By.NAME, "user_id")
username.send_keys(username_value)

# パスワードを入力
password_value = os.getenv("SBI_PASS")
password = driver.find_element(By.NAME, "user_password")
password.send_keys(password_value)

# ログインボタンをクリック
driver.find_element(By.NAME, "ACT_login").click()
time.sleep(5) # 画面が描画されるまで待機

# ------------------初回ログイン時のみ実行（ここから）---------------------------------
# デバイス認証（メール送信）
driver.find_element(By.CLASS_NAME, "seeds-button-lg").click()
time.sleep(2) # 画面が描画されるまで待機
driver.save_screenshot('./selenium_screenshot_code.png') # デバイス認証のための入力コードが記載されている画面

# デバイス認証（チェックボタン押下・ボタン押下）
driver.find_element(By.ID, "check-panel").click() # 認証完了確認のチェック
time.sleep(45) # 画面が描画される （ボタンが活性化して押下できるようになる）・メールから認証完了するまで待機
driver.find_element(By.ID, "device-auth-otp").click() # 認証完了確認のボタン押下
time.sleep(15) # 自動で画面遷移するのを待つ
# ------------------初回ログイン時のみ実行（ここまで）---------------------------------

# ログアウトボタンをクリック
driver.find_element(By.LINK_TEXT, "ログアウト").click()
time.sleep(5)

driver.quit()
```
:::

## 3. ログイン・ログアウトする（２回目以降）
２回目以降は、デバイス認証することなくログインすることができます。
:::details master.ipynb（前述のmain.pyから不要な部分をコメントアウト）
```sh:master.ipynb
import os, getpass, time
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.common.by import By

options = Options()

# コンテナ内の Chromium 実行ファイルを指定
chrome_bin = os.environ.get("CHROME_BIN", "/usr/bin/chromium")
options.binary_location = chrome_bin

# ヘッドレス + ルート起動向けの安定化フラグ
options.add_argument("--headless=new")
options.add_argument("--no-sandbox")
options.add_argument("--disable-dev-shm-usage")
options.add_argument("--disable-gpu")
options.add_argument("--window-size=1920,1080")

#SBI側の仕様変更により、2025/10/25以降指定必須（使用する環境に合わせて指定）
options.add_argument("--user-agent=Mozilla/5.0 (X11; Linux x86_64) "
                        "AppleWebKit/537.36 (KHTML, like Gecko) "
                        "Chrome/142.0.0.0 Safari/537.36")  


# デバイス認証済みのプロファイルが保存される場所を読み込ませる
se_profile = os.getenv("SE_PROFILE")
options.add_argument(f'--user-data-dir={se_profile}')


# ドライバは webdriver_manager に任せる（ブラウザ本体は上で用意済み）
service = Service(ChromeDriverManager().install())
driver = webdriver.Chrome(service=service, options=options)

# UAを確認
ua = driver.execute_script("return navigator.userAgent;")
print("Current User-Agent:", ua)


# SBI証券のページへ遷移
url = "https://login.sbisec.co.jp/login/entry"
driver.get(url)
time.sleep(3) # 画面が描画されるまで待機

# ユーザーネームを入力
username_value = os.getenv("SBI_USER")
username = driver.find_element(By.NAME, "username")
username.send_keys(username_value)

# パスワードを入力
password_value = os.getenv("SBI_PASS")
password = driver.find_element(By.NAME, "password")
password.send_keys(password_value)

# ログインボタンをクリック
driver.find_element(By.ID, "pw-btn").click()
time.sleep(5) # 画面が描画されるまで待機

# # ------------------初回ログイン時のみ実行（ここから）---------------------------------
# # デバイス認証（メール送信）
# driver.find_element(By.ID, "sendEmailButton").click()
# time.sleep(2) # 画面が描画されるまで待機
# driver.save_screenshot('./selenium_screenshot_code.png') # デバイス認証のための入力コードが記載されている画面

# # デバイス認証（チェックボタン押下・ボタン押下）
# driver.find_element(By.ID, "authCheck").click() # 認証完了確認のチェック
# time.sleep(45) # 画面が描画される （ボタンが活性化して押下できるようになる）・メールから認証完了するまで待機
# driver.find_element(By.ID, "otpRegisterButton").click() # 認証完了確認のボタン押下
# time.sleep(15) # 自動で画面遷移するのを待つ
# # ------------------初回ログイン時のみ実行（ここまで）---------------------------------

# ログアウトボタンをクリック
driver.find_element(By.LINK_TEXT, "ログアウト").click()
time.sleep(5)

driver.quit()
```
:::



## おわりに
今回紹介した方法を使えば、Dockerコンテナ環境でも一度デバイス認証を通せば、以降は安定してSeleniumによるログイン・ログアウトを自動化できます。開発環境の再現性を保ちつつ、ブラウザ操作の自動化を実現できる点は、データ収集や分析を効率化する上で大きなメリットです。

ただし、**自動ログインやスクレイピングの実行は各サイトの規約や利用ルールに必ず従うことが大前提**です。便利な仕組みも、ルールを守ってこそ安心して活用できます。自動化を取り入れる際は、リスクとメリットを踏まえて、適切に運用してください。

:::details おまけ：SBI証券のBRISKの全板ページの見方メモ
詳細は、[こちらの公式ページ](https://static.brisk.jp/assist-v2/fullitapanelhelp.html)を参照してください。
![](/images/20250819_tech_sbilogin/board.png =700x)
:::
