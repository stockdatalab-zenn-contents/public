---
title: "Windows11にWSL＋DockerでPython環境を構築する方法：初心者向け"
emoji: "🛠"	
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "環境構築", "Docker", "WSL", "VSCode"]
published: false
---
## はじめに
こんにちは、SE出身のデータサイエンティスト「マチ」です。データ分析や機械学習を始める際に、最初につまずきやすいのが「開発環境の構築」ですよね。今回は、**Windows PCにWSL2（Windows Subsystem for Linux 2）を導入し、Dockerを使ってPython環境を構築する方法**を、手順を追って説明します。構築する環境のイメージ図は以下です。
![](/images/20250519_tech_env_docker/env_all.png =400x)

## 手順
### 1. WSL2でのUbuntsuのインストール
#### 1-1. WSLを動作するようにする
WSLとは、Windows上でLinuxを実行するためのシステムです。Windows11では標準搭載されています。DockerEngineはLinuxで動作する環境なので、WSLでLinux環境を用意します。コントロールパネルから「プログラム」→「Windowsの機能の有効化または無効化」→「Linux用Windowsサブシステム＆仮想マシンプラットフォーム」をチェックして有効化します。PCの再起動が必要になることがあります。
![](/images/util/dummy_black.png =150x)

#### 1-2. Ubuntsuをインストールする
https://learn.microsoft.com/ja-jp/windows/wsl/install に従って“Windows PowerShell”を操作します。
コマンドでWSLを更新します。
```sh:Windows PowerShell
wsl --update
```
Ubuntu-24.04 を入れます。Linuxユーザー名とパスワードを聞かれるので設定してください。（忘れないようにしてください！）
```sh:Windows PowerShell
wsl --install -d Ubuntu-24.04
```
Ubuntuに自動ログインするので一度ログアウトコマンドを実行してください。
```sh:Windows PowerShell
logoutまたはexit
```
![](/images/util/dummy_black.png =150x)

#### 1-3. WSLの設定
以下のコマンドを使って、インストール確認とデフォルト設定の保存をします。
今回は「Ubuntu-24.04」のversion「2」に「＊」がついていれば良いです。
WSLのverについて
https://qiita.com/omu_kato/items/f9a6b5a02e25f5f2a487
```sh:Windows PowerShell
wsl -l -v
wsl --set-default <DistributionName>
wsl --set-default-version <Version#> 
```
一旦、ここまできたらPowerShellを再起動しましょう。
PowerShellで“wsl ~”と入力することで、先ほど設定した既定の Linux ディストリビューションを開くことができます。（”exit”で閉じます。）
また、Windowsメニューに「Ubuntu24.04LTS」が入っていることを確認します。（以降はこちらのBashから操作します。）
上記を確認したら、PowerShellを閉じます。
その他の変更（日本語化等）は各自自由に設定してください。
Tips
WSL上のUbuntuから/mntを見るとWindows内のファイルにアクセスできます。
が、パフォーマンスの観点から、
WSLを使用するプロジェクトフォルダはWSL(Ubuntu)上に作ることが推奨されているので、アクセスすることは稀です。
WindowsからもUbuntu内のファイルにアクセスできます。
エクスプローラーで\\wsl$”やLinuxと入力する。
![](/images/util/dummy_black.png =150x)
#### ここまでの完成図
![](/images/20250519_tech_env_docker/env1.png =400x)





### 2. DockerEngineのインストール
https://docs.docker.com/engine/install/ubuntu/ に従って、Ubuntuにdockerをinstallします。
以降のコマンドはUbuntuアプリから起動するBashで実行します。
以下のコマンドでdockerのaptリポジトリを登録します。
```sh:Windows PowerShell
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update 
```
以下のコマンドでDockerEngineをinstallします。
```sh:Windows PowerShell
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
以下のコマンドでDockerEngineが正しくinstallされたことを確認します。
“Hello from Docker!”が出力されれば、install成功です。
```sh:Windows PowerShell
sudo docker run --rm hello-world
```
以下のコマンドで、現在のユーザーをdockerグループに登録し、sudo無でdockerコマンドを実行できるようにしておきます。
```sh:Windows PowerShell
sudo usermod -aG docker $USER
newgrp docker
docker run --rm hello-world
```
ついこの間まで、Windows及びMacOSのマシンにおいて、Docker環境を構築する際のメジャーなソフトフェアは「Docker Desktop」でしたが、2021-2022に有償化されています。


（ついでに）以下のコマンドでUbuntuにGitをinstallします。
```sh:Windows PowerShell
sudo apt install git
```
#### ここまでの完成図
![](/images/20250519_tech_env_docker/env2.png =400x)





### 3. VSCodeやpythonのインストール
Microsoft StoreからVisual Studio Code をinstallします。

Visual Studio Code を起動し、「表示 -> 拡張機能」から以下を検索し、インストールします。
Dev Containers
WSL
Japanese Language Pack for Visual Studio Code（任意）

Ubuntuで右記のコードを実行して作業用ディレクトリを用意します。
本ガイドでは、~(/home/[username]/)下に作成しますが、場所は任意とします。

作業用ディレクトリ移動した後、”code .”コマンドを実行し、VS Codeを起動します。
初回起動時、自動的にUbuntuにVS Code Serverがインストールされ、Ubuntuやコンテナ上で動かすプログラムにも、ホスト（Windows）側のVS Codeアプリから簡単に接続、操作できるようになります。
#### ここまでの完成図
![](/images/20250519_tech_env_docker/env3.png =400x)





### 4. python環境用のコンテナの構築
Dockerfile（+ requirements.lock）からDockerImageをつくります。
Ubuntu内のDockerfileを配置したディレクトリで以下のコマンドを実行します。

#### 完成図
![](/images/20250519_tech_env_docker/env4.png =400x)


## さいごに

