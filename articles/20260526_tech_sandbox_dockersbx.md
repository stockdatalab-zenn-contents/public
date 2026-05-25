---
title: "【図解】Docker Sandboxes（docker/sbx-releases）をWindows11で使う手順"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["sandbox", "windows", "docker", "python"]
published: true
---
## はじめに
AIコーディングエージェントにコード修正や依存関係のインストールを任せると便利ですが、「ローカルPCのファイルを壊さないか」「Dockerデーモンや認証情報に触られないか」という不安もあります。

Docker Sandboxesは、Claude Code・Codexなどの**AIツールを、隔離されたmicroVM上で実行するための仕組み**です。**各サンドボックスには専用のDockerデーモン、ファイルシステム、ネットワークが用意され**、AIツールはコンテナのビルド、パッケージのインストール、ファイル編集などを行えますが、ホスト環境には直接触れにくい構造になっています。

**sandboxとは、AIツールを隔離された実行環境で動かすことで、AIの操作がホスト環境に与える影響を抑える仕組み**です。（ただし、**完全な安全を保証するものではありません**。）AIに任せる作業範囲が広がるほど、こうした安全装置の重要性は高まります。sandboxを使うメリット・デメリットは以下の通りです。
![](/images/20260526_tech_sandbox_dockersbx/sandbox_merideme.png =600x)

| 観点      | メリット            | デメリット                  |
| ------- | --------------- | ---------------------- |
| 安全性     | ホスト環境を守れる       | 設定を間違えると安全性が下がる        |
| ファイル操作  | 影響範囲を限定できる      | 共有・コピーが面倒              |
| コマンド実行  | 危険コマンドの被害を限定できる | 一部コマンドが制限される           |
| ネットワーク  | 外部送信を制御できる      | install/API/gitで詰まりやすい |
| 環境管理    | 再現性が上がる         | 初期構築が面倒                |
| AI活用    | 自律作業させやすい       | 権限設計が必要                |
| パフォーマンス | 環境を使い捨てできる      | VM/コンテナ分のオーバーヘッドあり     |

  

今回は、Windows PC 上で Docker を使って動作する Python アプリを、Docker Sandboxesで編集できる環境を構築する手順を、順を追って説明します。構築する環境のイメージ図は以下です。
![](/images/20260526_tech_sandbox_dockersbx/env_all.png =600x)

なお、本手順は2026/5/24時点のものです（[参考](https://github.com/docker/docs/blob/main/content/manuals/ai/sandboxes/get-started.md)）。適宜、最新の情報を確認するようにしてください。

## 1. Docker環境を構築する
sandbox環境の土台となるWSL2＋Docker環境 を準備します。記事「[【図解】Windows11でWSL2＋DockerによるPython開発環境を構築する手順](https://zenn.dev/stockdatalab/articles/20250519_tech_env_docker)」の以下の章を参照ください。
- 「 1. WSL2でUbuntuをインストールする」
- 「2. Docker EngineをUbuntuにインストールする」
### ここまでの完成図
![](/images/20260526_tech_sandbox_dockersbx/env_01.png =600x)

## 2. WSLにsbx CLIをインストールし、セットアップする
### 2-1. WSL（Ubuntu）にKVMをインストールする
WSLをセットアップしたら、`C:\Users\<ユーザー名>\.wslconfig` でNested Virtualizationを明示して、KVMを使うための環境を整えます。.wslconfig はWSL2 VM全体に適用される設定ファイルで、nestedVirtualization はWSL2内でさらにVMを動かすための設定です。
```txt:.wslconfig
[wsl2]
nestedVirtualization=true
```
.wslconfigを作成・保存したら、WSLを再起動します。
```sh:実行コマンド
wsl --shutdown
wsl ~
```
WSL内でKVMが見えているか確認します。
```sh:実行コマンド
# 正常なら「crw-rw---- 1 root kvm 10, 232 ... /dev/kvm」のように表示される
ls -l /dev/kvm
```

次に、KVMモジュールを確認します。
```sh:実行コマンド
# Intel CPUなら「kvm_intel」「kvm」が表示されれば問題ない
lsmod | grep kvm
```

次に、診断ツールで確認します。
```sh:実行コマンド
# 正常なら「INFO: /dev/kvm exists」「KVM acceleration can be used」のように表示される
sudo apt update
sudo apt install -y cpu-checker
kvm-ok
```

最後に、ユーザーを kvm グループに追加します。/dev/kvm が見えていても、権限がないと使えません。
```sh:実行コマンド
sudo usermod -aG kvm $USER
newgrp kvm
```
WSLを再起動して、ユーザーが kvm グループに追加されていることを確認します。
```sh:実行コマンド
# groupsコマンドの実行結果に「kvm」が含まれていれば、問題ない
wsl --shutdown
wsl ~
groups
```


### 2-2. WSLにsbx CLIをインストールする
sbx CLIをWSL上にインストールし、Dockerアカウントでログインします。UbuntuではKVMが必要です。KVMが使えない環境では sbx は起動しません。また、Docker Desktopは必須ではありません。
```sh:実行コマンド
curl -fsSL https://get.docker.com | sudo REPO_ONLY=1 sh
sudo apt-get install docker-sbx
sbx login
```
`sbx login`ではブラウザでの認証が求められます。**ブラウザが自動で開かない場合は、表示されたURLをコピーして、Windows側のブラウザで手動で開きます**。なお、**事前に認証に使うDockerIDを発行しておく必要があります**。DockerIDの発行方法は、[こちらの「DockerHubにアカウント登録」](https://sukkiri.jp/technologies/dockerhub_with_podman.html)を参考にします。
![](/images/20260526_tech_sandbox_dockersbx/sbx_login.png =800x)
![](/images/20260526_tech_sandbox_dockersbx/sbx_login_docker.png =800x)

### 2-3. sbx CLIをセットアップする
初回ログイン時に、ネットワークポリシーを選んでおきます。最初は **Balanced** が使いやすいです。npm、GitHub、モデルプロバイダーなど、開発でよく使う通信はある程度許可しつつ、それ以外は制限できます。
```sh:実行コマンド
sbx policy ls
```
![](/images/20260526_tech_sandbox_dockersbx/sbx_policy_ls.png =800x)

| ポリシー        | 内容                      |
| ----------- | ----------------------- |
| Open        | すべてのネットワーク通信を許可         |
| Balanced    | デフォルト拒否。ただし一般的な開発サイトは許可 |
| Locked Down | 明示的に許可した通信以外はブロック       |
### ここまでの完成図
![](/images/20260526_tech_sandbox_dockersbx/env_02.png =600x)


## 3. sbx CLIでAIツールを起動する
ClaudeCodeなどのAIツールを使うには、AnthropicなどのAPIキーやOAuth認証が必要になる場合があります。例えば、環境変数（ANTHROPIC_API_KEY）を使う場合は、次のように登録します。
```sh:実行コマンド
echo 'export ANTHROPIC_API_KEY="your-api-key-here"' >> ~/.bashrc
source ~/.bashrc
```

Claude CodeをDocker Sandboxesで起動する場合のコマンドは以下の通りです。
```sh:実行コマンド
cd ~/my-project
sbx run claude
```

### ここまでの完成図
![](/images/20260526_tech_sandbox_dockersbx/env_03.png =600x)


## 4. アプリのコンテナを構築・起動する
サンドボックス内でDockerコンテナを作るには、サンドボックスに入ってから、通常のDockerコマンドを実行します。（参考：記事「[【図解】Windows11でWSL2＋DockerによるPython開発環境を構築する手順](https://zenn.dev/stockdatalab/articles/20250519_tech_env_docker)」の「4. python環境用のコンテナを構築する」の章）
```sh:実行コマンド
cd ~/my-project
sbx run <sandbox-name>
sbx exec -it <sandbox-name> bash
docker build --no-cache -t my-project0 .
docker images
docker compose up -d
```
**不要になったサンドボックスは削除して、ディスク容量を回収しましょう**。
```sh:実行コマンド
sbx stop <sandbox-name>
sbx rm <sandbox-name>
```
### 完成図
![](/images/20260526_tech_sandbox_dockersbx/env_all.png =600x)


## 5. 参考：よく使うsbxコマンド
| コマンド                              | 用途                   |
| --------------------------------- | -------------------- |
| `sbx run <agent>`                 | エージェントをサンドボックス内で起動   |
| `sbx ls`                          | サンドボックス一覧を表示         |
| `sbx run <name>`                 | サンドボックスを起動<br>`sbx run shell`は、AIエージェントなしで<br>sandbox内のBashに入るためのもの |
| `sbx stop <name>`                 | サンドボックスを停止           |
| `sbx rm <name>`                   | サンドボックスを削除           |
| `sbx exec -it <name> bash`        | サンドボックス内でシェルを起動      |
| `sbx cp`                          | ホストとサンドボックス間でファイルコピー |
| `sbx ports`                       | サンドボックス内サービスのポート公開   |
| `sbx policy ls`                   | ネットワークポリシーの確認        |
| `sbx policy allow network <host>` | 特定ホストへの通信を許可         |
| `sbx secret set`                  | APIキーなどのシークレットを登録    |
| `sbx template`                    | 再利用可能なテンプレートを管理      |


## おわりに
Docker Sandboxesを使うと、AIコーディングエージェントにある程度自由な作業を任せながら、ホスト環境、Dockerデーモン、認証情報、ネットワークアクセスを分離・制御できます。

一方、Docker Sandboxesは万能ではありません。重要なのは、**ワークスペースはホストと共有される**という点です。エージェントがワークスペース内のファイルを書き換えると、その変更はホスト側にもリアルタイムで反映されます。

AIエージェントにローカル開発環境を触らせるのが不安な場合、Docker Sandboxesは有力な選択肢になります。まずは小さな検証用の場所で挙動を確認してから、本格的な開発ワークフローに組み込むのがよいでしょう。


