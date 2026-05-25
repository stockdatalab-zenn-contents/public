---
title: "【図解】Bob Shell sandbox環境をWindows11＋Dockerで構築する手順"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Bob", "sandbox", "windows", "docker", "python"]
published: true
---
## はじめに
Bob Shellは、IBM Bobをターミナルから利用するためのCLIツールです。Bob ShellはClaude Codeのように、様々な処理を対話形式またはコマンドで依頼できます。ファイル操作やシェルコマンド実行も扱えるため、使い方によってはローカル環境への影響が大きくなります。 そこで重要になるのが **sandboxモード** です。

**sandboxとは、AIツールを隔離された実行環境で動かすことで、AIの操作がホスト環境に与える影響を抑える仕組み**です。（ただし、**完全な安全を保証するものではありません**。）AIに任せる作業範囲が広がるほど、こうした安全装置の重要性は高まります。sandboxを使うメリット・デメリットは以下の通りです。
![](/images/20260523_tech_sandbox_bobshell/sandbox_merideme.png =600x)

| 観点      | メリット            | デメリット                  |
| ------- | --------------- | ---------------------- |
| 安全性     | ホスト環境を守れる       | 設定を間違えると安全性が下がる        |
| ファイル操作  | 影響範囲を限定できる      | 共有・コピーが面倒              |
| コマンド実行  | 危険コマンドの被害を限定できる | 一部コマンドが制限される           |
| ネットワーク  | 外部送信を制御できる      | install/API/gitで詰まりやすい |
| 環境管理    | 再現性が上がる         | 初期構築が面倒                |
| AI活用    | 自律作業させやすい       | 権限設計が必要                |
| パフォーマンス | 環境を使い捨てできる      | VM/コンテナ分のオーバーヘッドあり     |

  

今回は、Windows PC 上で Docker を使って動作する Python アプリを、Bob Shell の sandbox モードで編集できる環境を構築する手順を、順を追って説明します。構築する環境のイメージ図は以下です。
```txt:フォルダ構成
yyyymmdd_pjt_name/
│
├─start-bob.env
├─start-bob.sh
├─stop-bob.sh
│
├─app/
│  │  docker-compose.yaml
│  │  docker-compose.override.yaml
│  │  Dockerfile
│  │  requirements.txt
│  └──source/
│         └─XXXXX.py
│
└─ .bob/
    └─ settings.json
```
![](/images/20260523_tech_sandbox_bobshell/env_all.png =600x)

なお、本手順は2026/5/22時点の[BobShellの公式案内ページ](https://bob.ibm.com/docs/shell)を参考にしたものです。適宜、最新の情報を確認するようにしてください。


## 1. Docker環境を構築する
sandbox環境の土台となるWSL2＋Docker環境 を準備します。記事「[【図解】Windows11でWSL2＋DockerによるPython開発環境を構築する手順](https://zenn.dev/stockdatalab/articles/20250519_tech_env_docker)」の以下の章を参照ください。
- 「 1. WSL2でUbuntuをインストールする」
- 「2. Docker EngineをUbuntuにインストールする」
### ここまでの完成図
![](/images/20260523_tech_sandbox_bobshell/env_01.png =600x)

## 2. WSLにBob Shellをインストールする
Bob Shellは**Node.jsがインストールされていることが前提**です。Node.jsが**WSL上に**インストールされていない場合は、まずは[こちらの手順](https://qiita.com/nekonekonekosan/items/61a6b9d4da6bdfd1d0bb#nodejs-%E3%82%92%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%99%E3%82%8B)を参照してインストールします。インストール後、パスが通っているか確認します。
:::details Node.jsインストールコマンド（PowerShell）
```sh
#wsl起動
wsl ~

# curl コマンドをインストール
sudo apt-get install curl

# curl コマンドでnvm をインストール
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.2/install.sh | bash

#wsl再起動
wsl --shutdown
wsl ~

# nvm のインストールを確認（バージョンが表示されるか確認）
nvm --version

# Node.js の安定版(LTS 版)をインストール
nvm install --lts

# Node.js のインストールを確認（バージョンが表示されるか確認）
node -v
```
:::

Bob Shellを使えるようにセットアップし動作確認します。APIキーがある場合は、APIキーを使って認証できるようにしておくと、都度ブラウザで認証する手間を省けて便利です。
:::details Bob Shellセットアップコマンド（PowerShell）
```sh
# インストール
curl -fsSL https://bob.ibm.com/download/bobshell.sh | bash

# 環境変数（APIキー）設定
echo 'export BOBSHELL_API_KEY="your-api-key-here"' >> ~/.bashrc
source ~/.bashrc

# APIキーで認証してBob Shellを起動できることを確認
mkdir test
cd test
bob --auth-method api-key
```
:::

### ここまでの完成図
![](/images/20260523_tech_sandbox_bobshell/env_02.png =600x)


## 3. Bob Shellをsandboxモードで動かす
sandboxモードでBobを起動することで、sandboxモード用のDockerイメージが自動的にロードされます。
```sh:実行コマンド
# 現時点のDockerイメージを確認する
docker images

# Bobを起動する場所を作成・移動
mkdir test
cd test

# デフォルトのsandboxモードでBobを起動
BOB_SHELL_SANDBOX=docker bob --auth-method api-key

# Ctrl+CでBobを終了し、Dockerイメージを確認する
# node:25-trixieなど、sandboxモードで使用するDockerイメージが増えていることを確認する
docker images
```


プロジェクト単位でsandboxモードを常に有効にしたい場合は、.bob/settings.json に設定します。これにより`bob --auth-method api-key`や`bob`コマンドだけで、自動的にsandboxモードで起動されるようになります。
```json:settings.json
{
  "tools": {
    "sandbox": "docker"
  }
}
```

![](/images/20260523_tech_sandbox_bobshell/startbob.png =800x)

### ここまでの完成図
```txt:フォルダ構成
yyyymmdd_pjt_name/
└── .bob/
     └── settings.json
```
![](/images/20260523_tech_sandbox_bobshell/env_03.png =600x)


## 4. アプリのコンテナ構築のための資材準備する
Bobとは関係なく、以下の資材からアプリが動作する環境であるコンテナを構築・起動・停止できるか確認します。（参考：記事「[【図解】Windows11でWSL2＋DockerによるPython開発環境を構築する手順](https://zenn.dev/stockdatalab/articles/20250519_tech_env_docker)」の「4. python環境用のコンテナを構築する」の章）
- tar.gzなどの既存のDockerイメージをロードして使用する場合
  - dockerImageName.tar.gz
  - docker-compose.yaml
- DockerfileからDockerイメージを作成する場合
  - Dockerfile
  - requirements.txt
  - docker-compose.yaml


### ここまでの完成図
:::details フォルダ構成（tar.gzなどの既存のDockerイメージをロードして使用する場合）
```txt
yyyymmdd_pjt_name/
├─app/
│  │  docker-compose.yaml
│  ├──docker_image/
│  │     └─dockerImageName.tar.gz
│  └──source/
│         └─XXXXX.py
│
└─ .bob/
    └─ settings.json
```
:::
:::details フォルダ構成（DockerfileからDockerイメージを作成する場合）
```txt
yyyymmdd_pjt_name/
├─app/
│  │  docker-compose.yaml
│  │  Dockerfile
│  │  requirements.txt
│  └──source/
│         └─XXXXX.py
│
└─ .bob/
    └─ settings.json
```
:::

![](/images/20260523_tech_sandbox_bobshell/env_04.png =600x)


## 5. ２つのコンテナを連携させる
shファイルや`docker-compose.override.yaml`で、以下の設定を行います。**JupyterLabを使えるようにしておく**と、BobのsandboxモードでもJupyterLabのREST+WebSocketAPIを踏み台にして、アプリコンテナ内のpythonを実行できます。
- **マウント設定**
マウント場所を同じプロジェクトフォルダにすることで、アプリのソースコードを編集できるようにします。
- **ネットワーク設定**
同じネットワーク上に乗せることで、アプリのコンテナを操作し、動作確認などをできるようにします。


:::details start-bob.env
```sh
# start-bob.sh のプロジェクト固有設定
#
# start-bob.sh と同じディレクトリに置くと自動で source される。
# 設定可能な変数の意味は start-bob.sh のヘッダコメント参照。
#
# このプロジェクトでは事前ビルド済みの tar.gz から Docker image を load する。
IMAGE_TAR="${APP_DIR}/docker_image/dockerImageName.tar.gz"
IMAGE_TAG="py3_jupyter"
```
:::
:::details start-bob.sh
```sh
#!/usr/bin/env bash
# Bob Shell sandbox (docker) を WSL 上で起動するスクリプト (共通実装)
#
# 前提:
#   - WSL ディストリビューション内に Bob Shell と Docker Engine がインストール済み
#   - 本スクリプトはプロジェクトルート (app と同階層) から実行する
#   - app コンテナは docker-compose.yml(.yaml) の external network 指定で
#     共有ネットワーク (NETWORK) に参加する
#
# プロジェクト固有設定:
#   本スクリプトと同じディレクトリに `start-bob.env` があれば source する。
#   env ファイルで上書き可能な変数:
#     IMAGE_TAR   - 事前 load する Docker image の tar.gz パス (空なら load 工程スキップ)
#     IMAGE_TAG   - 上記 tar.gz が含む image タグ (IMAGE_TAR とセットで必須)
#     APP_DIR     - docker compose を実行するディレクトリ (default: ${ROOT_DIR}/app)
#     NETWORK     - Bob sandbox と app が共有するネットワーク名 (default: bob-app-net)
#
# 流れ:
#   1. (任意) IMAGE_TAR が設定されていれば、未 load の場合のみ docker load
#   2. 共有ネットワーク (NETWORK) が無ければ作成
#   3. app/ で docker compose up -d
#      (compose 側で external network "${NETWORK}" を参照しているため、
#       起動と同時に共有ネットワークに参加)
#   4. Bob sandbox を共有ネットワークに参加させて起動

set -euo pipefail

ROOT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# --- デフォルト設定 (env ファイルで上書き可能) ---
APP_DIR="${ROOT_DIR}/app"
IMAGE_TAR=""
IMAGE_TAG=""
NETWORK="bob-app-net"

# --- プロジェクト固有設定の読み込み ---
ENV_FILE="${ROOT_DIR}/start-bob.env"
if [[ -f "${ENV_FILE}" ]]; then
    # shellcheck disable=SC1090
    source "${ENV_FILE}"
fi

# [1/4] (任意) 事前ビルド済み Docker image を tar.gz から load
#   IMAGE_TAR が空なら工程ごとスキップ (compose の build: に委譲する想定)
echo "[1/4] Docker image の事前 load を確認"
if [[ -z "${IMAGE_TAR}" ]]; then
    echo "  -> IMAGE_TAR 未設定のためスキップ (compose 側でビルド/取得)"
else
    if [[ -z "${IMAGE_TAG}" ]]; then
        echo "IMAGE_TAR が指定されていますが IMAGE_TAG が未設定です" >&2
        exit 1
    fi
    if docker images --format '{{.Repository}}' | grep -qx "${IMAGE_TAG}"; then
        echo "  -> '${IMAGE_TAG}' は既に load 済み"
    else
        if [[ ! -f "${IMAGE_TAR}" ]]; then
            echo "イメージファイルが見つかりません: ${IMAGE_TAR}" >&2
            exit 1
        fi
        echo "  -> ${IMAGE_TAR} を load します"
        docker load -i "${IMAGE_TAR}"
    fi
fi

# [2/4] 共有ネットワーク "${NETWORK}" が無ければ作成
echo "[2/4] Docker network '${NETWORK}' を確認"
if docker network ls --format '{{.Name}}' | grep -qx "${NETWORK}"; then
    echo "  -> 既に存在"
else
    echo "  -> 作成します"
    docker network create "${NETWORK}" >/dev/null
fi

# [3/4] app コンテナを docker compose で起動
#   compose 側で external network "${NETWORK}" を参照しているため、
#   ここで起動した時点で共有ネットワークに参加済みとなる
echo "[3/4] docker compose で app を起動"
( cd "${APP_DIR}" && docker compose up -d )

# [4/4] Bob sandbox を起動
#   SANDBOX_FLAGS で Bob が docker run する際の追加フラグを渡し、
#   --network "${NETWORK}" で app と同じネットワークに参加させる
echo "[4/4] Bob sandbox を起動 (${NETWORK} に参加)"
export SANDBOX_FLAGS="--network ${NETWORK}"
cd "${ROOT_DIR}"
exec bob --auth-method api-key
```
:::
:::details stop-bob.sh
```sh
#!/usr/bin/env bash
# Bob Shell sandbox 終了後に app コンテナと共有ネットワークを片付けるスクリプト

set -euo pipefail

ROOT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
APP_DIR="${ROOT_DIR}/app"
NETWORK="bob-app-net"

echo "[1/2] docker compose down で app を停止"
( cd "${APP_DIR}" && docker compose down )

echo "[2/2] Docker network '${NETWORK}' を削除 (使用中なら skip)"
if docker network ls --format '{{.Name}}' | grep -qx "${NETWORK}"; then
    if docker network rm "${NETWORK}" >/dev/null 2>&1; then
        echo "  -> 削除しました"
    else
        echo "  -> 使用中のため残します"
    fi
else
    echo "  -> 既に存在しません"
fi
```
:::
:::details docker-compose.override.yaml
```yaml
# docker-compose.override.yaml
#
# docker compose は同ディレクトリの docker-compose.override.yaml を自動で
# 読み込み、ベースの docker-compose.yaml にマージする。
#
# ベースとの差分 (= ここで上書き/追加するもの):
#   - volumes / working_dir を ${APP_MOUNT_PATH:-/root/app} で env 切替可能に
#   - command で jupyter lab を明示起動
#   - networks: bob-app-net (external) に参加
#
# 注意:
#   - volumes は target (コンテナ側パス) をキーにマージされる。
#     APP_MOUNT_PATH 未設定時は target=/root/app となりベースの
#     `.:/root/app` を置き換える。設定時は別 target になる点に注意。
#   - service に networks を指定するとデフォルトネットワークは使われない。

services:
    jupyterlab:
        volumes:
            - .:${APP_MOUNT_PATH:-/root/app}
        working_dir: ${APP_MOUNT_PATH:-/root/app}
        ports:
            - "127.0.0.1:8888:8888"
        command: >
            jupyter lab
            --ip=0.0.0.0
            --port=8888
            --no-browser
            --allow-root
            --ServerApp.token=''
            --ServerApp.password=''
        networks:
            - bob-app-net

networks:
    bob-app-net:
        external: true
        name: bob-app-net
```
:::
:::details フォルダ構成（tar.gzなどの既存のDockerイメージをロードして使用する場合）
```txt
yyyymmdd_pjt_name/
│
├─start-bob.env
├─start-bob.sh
├─stop-bob.sh
│
├─app/
│  │  docker-compose.yaml
│  │  docker-compose.override.yaml
│  ├──docker_image/
│  │     └─dockerImageName.tar.gz
│  └──source/
│         └─XXXXX.py
│
└─ .bob/
    └─ settings.json
```
:::
:::details フォルダ構成（DockerfileからDockerイメージを作成する場合）
```txt
yyyymmdd_pjt_name/
│
├─start-bob.env
├─start-bob.sh
├─stop-bob.sh
│
├─app/
│  │  docker-compose.yaml
│  │  docker-compose.override.yaml
│  │  Dockerfile
│  │  requirements.txt
│  └──source/
│         └─XXXXX.py
│
└─ .bob/
    └─ settings.json
```
:::


## 6. 動作確認する
**使用するコンテナが起動している場合は`decker compose down`で停止した上で**、以下のコマンドを実行して、起動できたら完成です。
```sh:実行コマンド（PowerShell）
# 使用するコンテナが起動している場合は、停止してから実行すること

#プロジェクトフォルダに移動して、Bobをsandboxモードで起動
wsl ~
cd /mnt/c/Users/UserName/Documents/yyyymmdd_pjt_name
./start-bob.sh

# ネットワークが共有できていることを確認
## Bobのチャット欄で以下の通り依頼します。
## 200：疎通 OK
## 302：疎通 OK (ログインページへのリダイレクト)。トークンが要求されるだけで通信自体は成功
## 405：疎通 OK だが HEAD を投げてしまっているケース。-I を外して再実行
## その他（Could not resolve host 等）：bob-sandbox が bob-app-net に参加していない可能性あり。
http://[アプリのコンテナ名]:8888 に -I なしで GET して、返ってきたステータスコードだけ教えて

## 共有ネットワーク上にあるコンテナを表示する
## Bobを起動させたまま、他のWSLセッションで以下を実行
docker network inspect bob-app-net --format '{{range .Containers}}{{.Name}}{{"\n"}}{{end}}'

# マウント構造を表示する
# Bobを起動させたまま、他のWSLセッションで以下を実行
## アプリ・Bobのsandbox用のコンテナ名を確認する
docker ps

## コンテナのマウント構造を表示する
docker inspect [コンテナ名] --format '{{range .Mounts}}{{.Source}} -> {{.Destination}}{{"\n"}}{{end}}'

# Bobを終了する
## Ctrl+C２回でBobを終了
## その後、以下のコマンドでコンテナ・ネットワークを削除する
./stop-bob.sh
```

### 完成図
![](/images/20260523_tech_sandbox_bobshell/env_all.png =600x)


## おわりに
**sandboxモードは安全性を高めリスクを下げるための仕組みですが、万能ではありません**。
また、sandboxモードとあわせて使いたいのが **.bobignore**です。.bobignore は、Bob Shellに読ませたくないファイルやディレクトリを指定するためのファイルです。構文は .gitignore と同じように扱えます。
AIエージェントに開発作業を任せる場面では、**「便利さ」と「安全性」のバランスが重要**です。
Bob Shellを本格的に使う場合は、まずsandboxモードを有効にし、対象プロジェクト・対象ファイル・許可するコマンドを絞るところから始めるのがよいでしょう。