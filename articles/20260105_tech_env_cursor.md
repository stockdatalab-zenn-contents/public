---
title: "【図解】Windows11でWSL2＋Docker+CursorによるPython開発環境を構築する手順"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Cursor", "Docker", "バイブコーディング", "Vibe Coding"]
published: true
---
## はじめに
最近、AntigravityやcursorなどのAIエディタが流行っています。効率的に開発できたり、従来よりも大量のデータを扱えたりすることが魅力です。

今回は、Windows11 PCにWSL2を導入し、Docker上でcursorを使える環境を構築する方法を、手順を追って説明します。構築する環境のイメージ図は以下です
![](/images/20260105_tech_env_cursor/env.png =400x)


## 1. Docker環境を構築する
まずは、cursor接続の土台となるWSL2＋Docker環境 を準備します。記事「[【図解】Windows11でWSL2＋DockerによるPython開発環境を構築する手順](https://zenn.dev/stockdatalab/articles/20250519_tech_env_docker)」の以下の章を参照ください。
- 「 1. WSL2でUbuntuをインストールする」
- 「2. Docker EngineをUbuntuにインストールする」
- 「4. python環境用のコンテナを構築する」
### ここまでの完成図
![](/images/20260105_tech_env_cursor/env1.png =400x)
**これ以降のコマンドは、Linux上で実行します**。ubuntuのbashを起動させるか、PowerShell上で`wsl ~`でwslを起動させてから後続の作業を進めてください。
![](/images/20260105_tech_env_cursor/setting1-1.png =600x)
![](/images/20260105_tech_env_cursor/setting1-2.png =600x)



## 2. cursorをインストールする
この章では、cursorをWindowsPCにインストールします。
cursor自体はWindowsアプリとして動き、のちほどコンテナに接続して開発します。
### 2-1. cursorにユーザー登録をする
[こちらのcursorのページ](https://cursor.com/api/auth/login)から、サインアップまたはサインインをします。
![](/images/20260105_tech_env_cursor/setting2-1.png =600x)

必要に応じてProプランへアップグレードします（記事執筆時点：20ドル/月）。
AIツールは変化が早いので、**月額**で試す運用がおすすめです。
![](/images/20260105_tech_env_cursor/setting2-2.png =600x)

### 2-2. cursorをインストールする
[こちらのcursorのページ](https://cursor.com/ja/download)から、**ログインした状態で**インストーラー「cursor**User**Setup-x64-2.2.44.exe」（≠cursorSetup-x64-2.2.44.exe）を取得し、インストールを進めます。
![](/images/20260105_tech_env_cursor/setting2-3.png =600x)
:::details インストールする際は全てデフォルト値で問題ありません。
![](/images/20260105_tech_env_cursor/setting2-4.png =500x)
:::

### ここまでの完成図
![](/images/20260105_tech_env_cursor/env2.png =400x)



## 3. cursorをコンテナ環境に導入する
この章では、WSL（Linux）から cursor コマンドでcursorを起動できるようにします。
これができると、WSL内のプロジェクトに対してスムーズにcursorを立ち上げられます。
### 3-1. cursorにPATHを通す
PATH は「コマンドを探しに行く場所のリスト」、alias は「短い別名」を作る設定です。ここでは cursor と打ったら、Windows側にあるCursor実行ファイルを呼ぶようにします。

「\\wsl.localhost\Ubuntu-24.04\home\[#YourUserName#]」配下の「.bashrc」ファイルの末尾に以下を追記します。（**[#YourUserName#]は自分のユーザー名に変更してください**）
```bash:.bashrcファイルの末尾に追記
# cursor
export PATH="$PATH:/mnt/c/Users/[#YourUserName#]/AppData/Local/Programs/cursor/resources/app/bin"
alias cursor="/mnt/c/Users/[#YourUserName#]/AppData/Local/Programs/cursor/resources/app/bin/cursor"
```
![](/images/20260105_tech_env_cursor/setting3-1.png =600x)



### 3-2. 設定内容を反映する
WSL再起動またはsourceコマンドで、前段の変更を反映します。
```bash:設定内容を反映するコマンド
# wsl再起動
wsl --shutdown
wsl ~

# または、更新した「.bashrc」ファイルの読み込み
source ~/.bashrc
```

### 3-3. 動作確認を実施する
以下のコマンドを実行して、cursorが起動するか確認します。
```bash:cursorを起動するコマンド
cursor .
```
以下の画面が表示されれば成功です。
![](/images/20260105_tech_env_cursor/setting3-2.png =600x)

### 完成図
![](/images/20260105_tech_env_cursor/env.png =400x)


## 4. おまけ：cursorの使い方
ここからは、実際に開発するときの基本操作をまとめます。
### 4-1. 開発したいコンテナを起動する
まずは対象のコンテナを起動させます。
```bash:コンテナを起動
cd [#docker-compose.yamlまたはDockerfileが配置されている場所#]
docker compose start
# または
docker start [#コンテナ名#]
```
### 4-2. cursorからコンテナに接続する
cursor上で、以下をクリックして接続先のコンテナを指定します。左下にコンテナ名が表示されれば接続成功です。「Open Folder」からコンテナ内のフォルダを開くことができます。
![](/images/20260105_tech_env_cursor/setting4-1-1.png =600x)
![](/images/20260105_tech_env_cursor/setting4-1-2.png =600x)
![](/images/20260105_tech_env_cursor/setting4-1-3.png =600x)


### 4-3. モードやモデルを選ぶ
cursorでは、用途にあったモードやモデルを指定することができます。特に「Composer1」モデルはレスポンスが通常のAIより早いので、おすすめです。
![](/images/20260105_tech_env_cursor/setting4-2.png =600x)

### 4-4. チャットで、コーディングやエラーの解析をする
右側のチャット欄から、設計・実装・改善をまとめて依頼できます。また、ターミナルで出たエラーはクリックして表示される 「Add to Chat」で引用できるため、解析が楽にできます。
![](/images/20260105_tech_env_cursor/setting4-3.png =600x)

### 4-4. 過去のチャット履歴を参照する
チャット欄右上の時計マークから、過去の会話ログを確認できます。
![](/images/20260105_tech_env_cursor/setting4-4.png =600x)

## おわりに
以上で、WSL2＋Dockerで用意したPython実行環境に、cursorから接続して開発できる状態が整いました。cursorを使うメリットは、単に「AIが使える」だけではなく、実装・修正・デバッグの往復回数を減らし、作業のスピードを上げられる点にあります。

特に、エラーをそのままチャットに渡して原因の切り分けを依頼したり、既存コードを前提に設計方針を提案させたりできるため、手が止まりやすい工程ほど効果が出やすいです。まずは小さめのタスク（例：関数1つ追加、ログ出力追加、例外処理の改善など）から試して、cursorの「強い使いどころ」を掴むのがおすすめです。


