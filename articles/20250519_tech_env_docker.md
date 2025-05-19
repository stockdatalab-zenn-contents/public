---
title: "図解：Windows11でWSL2＋DockerによるPython開発環境を構築する手順

"
emoji: "🛠"	
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "環境構築", "Docker", "WSL", "VSCode"]
published: true
---
## はじめに
こんにちは、SE出身のデータサイエンティスト「マチ」です。データ分析や機械学習を始める際に、最初につまずきやすいのが「開発環境の構築」ですよね。今回は、**Windows11 PCにWSL2（Windows Subsystem for Linux 2）を導入し、Dockerを使ってPython環境を構築する方法**を、手順を追って説明します。構築する環境のイメージ図は以下です。
![](/images/20250519_tech_env_docker/env_all.png =400x)

## 1. WSL2でUbuntuをインストールする
### 1-1. WSL機能を有効化する
WSL[^1]とは、Windows上でLinuxを実行するためのシステムです。Windows11には標準搭載されており、WSL2を利用することでLinuxベースのDocker環境を構築できます。以下の手順でWSLを有効にします。
&emsp;1-1-1. 「コントロールパネル→プログラム→Windowsの機能の有効化または無効化」を開く。
&emsp;1-1-2. 以下にチェックを入れる。
&emsp;&emsp;・Linux用Windowsサブシステム
&emsp;&emsp;・Virtual Machine Platform（仮想マシンプラットフォーム）
&emsp;1-1-3. PCを再起動する（必要に応じて）。
![](/images/20250519_tech_env_docker/setting1.png =400x)

### 1-2. Ubuntuをインストールする
「[WSLを使用してWindowsにLinuxをインストールする方法](https://learn.microsoft.com/ja-jp/windows/wsl/install)」 に従って、PowerShell（**管理者モード**）でWSLのインストール・PCの再起動をします。念のため、以下のコマンドでWSLの更新を行い、今回はUbuntu-24.04 をインストールします。インストール後、自動的にUbuntuが起動し、**ユーザー名とパスワードの設定を求められます。忘れないようメモしておきましょう**。
```sh:PowerShell（管理者モード）
wsl --install # 実行後PCを再起動
wsl --update
wsl --install -d Ubuntu-24.04
```
![](/images/20250519_tech_env_docker/setting2.png =800x)
![](/images/20250519_tech_env_docker/setting3.png =800x)
Ubuntuに自動的にログインするので、以下のコマンドで一度ログアウトします。
```sh:PowerShell（管理者モード）
exit
```

### 1-3. WSLのバージョンと設定を確認する
WSL2にUbuntuが正しく登録されているか、PowerShell上で以下のコマンドで確認します。今回は「Ubuntu-24.04」のversion「2」に「＊」がついていれば問題ありません。
```sh:PowerShell
wsl -l -v
```
想定通りの設定になっていなければ、以下のコマンドで設定し、改めて確認します。
```sh:PowerShell
wsl --set-default <DistributionName>   #今回は<DistributionName>を「Ubuntu-24.04」に置換して実行
wsl --set-default-version <Version#>   #今回は<Version#>を「2」に置換して実行
wsl -l -v
```
![](/images/20250519_tech_env_docker/setting4.png =800x)
PowerShellで以下を実行することで、先ほど設定した既定の Linux ディストリビューションを開くことができます。（”exit”で閉じます。）ここまで確認したら、PowerShellを閉じます。
```sh:PowerShell
wsl ~
```
### ここまでの完成図
![](/images/20250519_tech_env_docker/env1.png =400x)





## 2. Docker EngineをUbuntuにインストールする
### 2-1. リポジトリを登録する
Windowsメニューから「Ubuntu24.04LTS」のBashを起動します。
![](/images/20250519_tech_env_docker/setting5.png =500x)

[Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)に従って、UbuntuにDocker[^2]をインストールします。まずは、aptパッケージマネージャーのパッケージ情報を最新化し、HTTPS 通信のために必要な証明書群などをインストールします。
```sh:Ubuntu-24.04アプリ（Bash）
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
```
次に、GPG鍵[^3]を保存するディレクトリを作成し、権限（所有者が読み書き実行、その他は読み実行）を設定します。次に、Docker公式の GPG鍵をダウンロードし、権限を設定します。
```sh:Ubuntu-24.04アプリ（Bash）
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
![](/images/20250519_tech_env_docker/setting6.png =800x)
Dockerリポジトリを aptのソースに追加します。最後に、追加された Docker リポジトリを含めて、aptパッケージリストを更新します。
```sh:Ubuntu-24.04アプリ（Bash）
# Add the repository to apt sources:
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update 
```

### 2-2. Docker Engineのインストールと動作確認を行う
以下のコマンドでDocker Engineをインストールします。
```sh:Ubuntu-24.04アプリ（Bash）
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
以下のコマンドでDockerEngineが正しくインストールされたことを確認します。
“Hello from Docker!”が出力されれば、インストール成功です。
```sh:Ubuntu-24.04アプリ（Bash）
sudo docker run hello-world
```
![](/images/20250519_tech_env_docker/setting7.png =800x)

### 2-3. sudo不要でdockerを使えるようにする
以下のコマンドで現在のユーザーをDockerグループに登録し、sudoせずにDockerコマンドを実行できるようにします。
```sh:Ubuntu-24.04アプリ（Bash）
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
```
![](/images/20250519_tech_env_docker/setting8.png =800x)

### 2-4. （ついでに）UbuntuにGitをインストールする
```sh:Ubuntu-24.04アプリ（Bash）
sudo apt install git
```

### ここまでの完成図
![](/images/20250519_tech_env_docker/env2.png =400x)





## 3. VSCodeやpythonをインストールする
### 3-1. VS Code本体をインストールする
Microsoft StoreからVisual Studio Code をインストールします。意図せず環境が変化してしまわないよう、VSCodeが自動更新されないようにすることをおすすめします。（[参考記事](https://1103slabo.com/blog/2022/11/282/)）

### 3-2. 拡張機能を導入する
Visual Studio Code を起動し、「表示→拡張機能」から以下を検索し、インストールします。
- Dev Containers
- WSL
- Japanese Language Pack for Visual Studio Code（任意）
![](/images/20250519_tech_env_docker/setting9.png =300x)

### 3-3. 作業用ディレクトリとVSCodeを連携させる
まず、Ubuntu内に作業ディレクトリを作成し、そこに移動します。次に、VSCodeをUbuntu上で起動させます。（本ガイドでは、~(/home/[username]/)下に作業用ディレクトリを作成しています。）
初回起動時、自動的にUbuntuにVSCode Serverがインストールされ、Ubuntuやコンテナ上で動かすプログラムにも、Windows側のVSCodeからシームレスに接続・操作できるようになります。
```sh:Ubuntu-24.04アプリ（Bash）
mkdir zenn_work
cd zenn_work
code .
```
Ubuntu上で起動させたVSCodeに拡張機能「Docker」をインストールします。
![](/images/20250519_tech_env_docker/setting10.png =500x)

### ここまでの完成図
![](/images/20250519_tech_env_docker/env3.png =400x)





## 4. python環境用のコンテナを構築する
※Dockerについては、記事「[Dockerによる開発環境構築のための概念理解と方法解説](https://qiita.com/S4nTo/items/977d28b0eac316915702)」が参考になりました。実際に環境構築する前に一読いただくと、より理解が深まると思います。

### 4-1. DockerImageを作成する
Dockerfile（+ requirements.txt または requirements.lock）からDockerImageをつくります。Ubuntu内の「dockerfile」を配置したディレクトリで以下のコマンドを実行します。本ガイドではベースイメージとして、DockerHubの「[jupyter/base-notebook:python-3.10.10](https://hub.docker.com/layers/jupyter/base-notebook/python-3.10.10/images/sha256-ec1988a6a88770b7fcb0e8ffe191c2b54ca7994fac41484e072ac6bc6ac5126b?context=explore)」を使用します。

```sh:Ubuntu-24.04アプリ（Bash）
docker build -t zenn0 .
docker images
```
```sh:コマンド実行場所（zenn_workフォルダ）配下のフォルダ構成
zenn_work             # 直下のファイルは環境構築用
│  Dockerfile
│  requirements.txt
│  docker-compose.yaml
│  
└─source              # コンテナ上で動かすプログラムファイル格納用
```
```sh:Dockerfileの例
#ベースイメージ
FROM jupyter/base-notebook:python-3.10.10

#ローカルのrequirements.txtを、コンテナ内にコピーしてライブラリをインストール
COPY requirements.txt ./
RUN pip3 install --no-cache-dir -r requirements.txt
```
```sh:requirements.txtの例
setuptools
numpy
pandas

ydata-profiling
autoviz
sweetviz
pygwalker

seaborn
matplotlib
scikit-learn

lightgbm
graphviz

Prophet
datetime
holiday
jpholiday

optuna
optuna-integration[lightgbm]
mlflow
```
### 4-2. コンテナを作成する
Ubuntu内の「docker-compose.yaml」（ポートやマウントの設定も含む）を配置したディレクトリで以下のコマンドを実行します。
```sh:Ubuntu-24.04アプリ（Bash）
docker compose up -d
docker ps -a
```
```sh:docker-compose.yamlの例
services:
  jupyterlab:
    build: 
      context: .
      dockerfile: .
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
![](/images/20250519_tech_env_docker/setting12.png =800x)

### 完成図
環境は完成しました。あと一息です。
![](/images/20250519_tech_env_docker/env4.png =400x)





## 5. 構築した環境でpythonを動かしてみる
### 5-1. コンテナ上のファイルをVSCodeで編集できるようにする
VSCodeをUbuntu上で起動させます。
```sh:Ubuntu-24.04アプリ（Bash）
code .
```
「左端のDockerマーク→コンテナ(イメージ)名を右クリック→Visual Studio Codeをアタッチする→上部に表示されるコンテナ名を左クリック」します。
![](/images/20250519_tech_env_docker/setting13.png =800x)
左下にコンテナ名が表示されたVSCodeのウィンドウが新しく開き、コンテナ内のソースを編集できるようになります。「フォルダを開く」でローカルとマウントしているコンテナ内の場所を指定します。
![](/images/20250519_tech_env_docker/setting14.png =800x)
コンテナ上のVSCodeでファイルを作成して保存する[^4]と、Windowsのエクスプローラー（マウント先）から保存したファイルを参照できます。
![](/images/20250519_tech_env_docker/setting15.png =800x)

### 5-2. pythonの動作確認をする
ipynbファイルのセルに以下を入力して、ctrl+Enterで実行しようとすると、右下や上部にメッセージが表示されます。表示されたメッセージに従って、必要なものをコンテナ上にインストールします。
```py:zenn.py
print('Hello python')
```
![](/images/20250519_tech_env_docker/setting16.png =800x)
再度、ipynbファイルのセル上でctrl+Enterで実行します。セルの下にエラーが出ることなく「Hello」と表示されればOKです。お疲れ様でした。
![](/images/20250519_tech_env_docker/setting17.png =500x)
<br>
なお、作成・起動したコンテナは、以下のコマンドで停止または削除できます。
```sh:Ubuntu-24.04アプリ（Bash）
docker compose stop   #コンテナの停止のみ
または
docker compose down   #コンテナの停止と削除
```





## さいごに
今回は、Windows上にWSL2とDockerを用いたPython開発環境を整える手順を紹介しました。ローカル環境での構築は難しそうに見えますが、WSLとDockerを使うことで、トラブルの少ない柔軟な開発が可能になります。Pythonによるデータ分析や機械学習を始める方の参考になれば幸いです。


[^1]: WSLについての補足
・バージョンの違いついて
記事「[WSLには3つの「バージョン」がある](https://qiita.com/omu_kato/items/f9a6b5a02e25f5f2a487)」が参考になりました。
・WSL上のUbuntuから/mnt/c/を見るとWindows内のファイルにアクセスできます。
・Windowsからもエクスプローラーで"\\\wsl$”やLinuxと入力すると、
&emsp;Ubuntu内のファイルにアクセスできます。

[^2]: 数年前まで、Windows及びMacOSのマシンにおいて、
Docker環境を構築する際のメジャーなソフトフェアは「Docker Desktop」でしたが、
2021-2022に有償化されています。

[^3]: GPG鍵とは、ソフトウェアの信頼性を検証するために使われる暗号鍵です。

[^4]:保存できない場合は、記事[Dockerが作成したWSL上のファイルをVSCodeで編集・保存する方法
](https://zenn.dev/conbrio/scraps/d1fb667b2e00a4)を参考にしてみてください。