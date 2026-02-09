---
title: "【図解】Windows11でWSL2＋Docker+VSCode+ClaudeCodeによるPython開発環境を構築する手順"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["WSL2", "Docker", "VSCode", "ClaudeCode", "python"]
published: true
---
## はじめに
この記事では、Windows 11 に WSL2 を導入し、Docker上で **Claude Code（VS Code拡張）** を使える開発環境を作る手順を説明します。
`.devcontainer/devcontainer.json` を置き、**コンテナ起動時に Claude Code を自動でセットアップ**できるようにすることで、今後コンテナを作り直しても、拡張機能を手動インストールし直す手間を減らします。
![](/images/20260209_tech_env_claude/env.png =400x)


## 1. Docker環境を構築する
まずは、ClaudeCode利用の土台となるWSL2＋Docker環境 を準備します。記事「[【図解】Windows11でWSL2＋DockerによるPython開発環境を構築する手順](https://zenn.dev/stockdatalab/articles/20250519_tech_env_docker)」を参照ください。

### ここまでの完成図
![](/images/20260209_tech_env_claude/env1.png =400x)


## 2. Claudeのアカウントを作る
[こちら](https://claude.ai/)からClaudeのアカウントを作ります。Claude Code（VS Code拡張機能）は無料でインストールできますが、実際に使うには **Claudeの有料プラン（Pro/Maxなど）でログインする**か、**APIキーで利用する**必要があります。個人利用で迷う場合は、まずは **Pro/Maxなどの有料プラン**が運用しやすいです。
- APIキー管理が不要で、VS Code上でログインして使える
- 利用コストの見通しが立てやすい（従量課金のブレが出にくい）

一方、APIキー運用は以下のようなケースで向きます。
- チームで請求や上限をまとめて管理したい
- 自動化・バッチ処理などで「プログラムから叩く」前提が強い
:::details アカウント作成時のイメージ
![](/images/20260209_tech_env_claude/create_acc_1.png =600x)
![](/images/20260209_tech_env_claude/create_acc_2.png =600x)
![](/images/20260209_tech_env_claude/create_acc_3.png =600x)
![](/images/20260209_tech_env_claude/create_acc_4.png =600x)
![](/images/20260209_tech_env_claude/create_acc_5.png =600x)
![](/images/20260209_tech_env_claude/create_acc_6.png =600x)
:::

## 3. コンテナにClaude Codeを入れる（devcontainer.json 作成）
`.devcontainer/devcontainer.json` は、ざっくり言うと 「**開発環境のレシピ（設定メモ）**」です。これを用意しておくと、コンテナを開き直したタイミングで以下を **自動で導入**でき、毎回手で入れ直す手間が減ります。

- 必要な機能（例：Node.js、Claude Code CLI）
- VS Code拡張（Claude Code）


```txt:フォルダ構成
zenn_work  
│  docker-compose.yaml
│  Dockerfile
│  requirements.txt
│  
├─.devcontainer
│      devcontainer.json
│      
└─source
        XXXXX.py（任意のソース）
```
```json:devcontainer.json
{
    "name": "Existing Docker Compose (Extend)",

    "dockerComposeFile": [
        "../docker-compose.yaml"
    ],

    // service は docker-compose.yaml のサービス名に合わせてください
    "service": "jupyterlab",

    "workspaceFolder": "/home/jovyan/work",

    "features": {
        "ghcr.io/devcontainers/features/node:1": {},
        "ghcr.io/anthropics/devcontainer-features/claude-code:1.0.5": {}
    },

    "customizations": {
        "vscode": {
        "extensions": ["anthropic.claude-code"]
        }
    }
}
```

:::details 参考：devcontainer.json（解説付き）
```json:devcontainer.json
// For format details, see https://aka.ms/devcontainer.json. For config options, see the
// README at: https://github.com/devcontainers/templates/tree/main/src/docker-existing-docker-compose
{
    "name": "Existing Docker Compose (Extend)",

    // Update the 'dockerComposeFile' list if you have more compose files or use different names.
    // The .devcontainer/docker-compose.yml file contains any overrides you need/want to make.
    "dockerComposeFile": [
        "../docker-compose.yaml"
    ],

    // The 'service' property is the name of the service for the container that VS Code should
    // use. Update this value and .devcontainer/docker-compose.yml to the real service name.
    "service": "jupyterlab",

    // The optional 'workspaceFolder' property is the path VS Code should open by default when
    // connected. This is typically a file mount in .devcontainer/docker-compose.yml
    "workspaceFolder": "/home/jovyan/work",

    // Features to add to the dev container. More info: https://containers.dev/features.
    // Node.js 実行環境を追加で入れる（Claude Codeが Node を前提にしていることが多いので保険として入れる）
    // Claude Code cliを追加で入れる
    // 参考：https://github.com/anthropics/devcontainer-features/pkgs/container/devcontainer-features%2Fclaude-code
    "features": {
        "ghcr.io/devcontainers/features/node:1": {},
        "ghcr.io/anthropics/devcontainer-features/claude-code:1.0.5": {}
    },

    // Configure tool-specific properties.
    // VS Code側で Claude Code 拡張を自動インストールする
    "customizations": {
        "vscode": {
        "extensions": ["anthropic.claude-code"]
        }
    }
}
```
:::

ワークディレクトリに移動し、WSL上で VS Code を起動します。
```sh:Ubuntu-24.04アプリ（Bash）またはPowerShell
cd zenn_work
code .
```
次に、VS Code のコマンドパレット（Ctrl+Shift+P）から「Dev Containers: **Reopen in Container**」を選びます。これにより、devcontainer.json の内容が反映された状態でコンテナが再構築・再接続されます。
![](/images/20260209_tech_env_claude/extention_1.png =800x)

Reopen後のコンテナ上で、Claude Code 拡張と Claude アカウントを紐づけます。紐づけが完了すると、VS Code 上で Claude Code を使えるようになります。

![](/images/20260209_tech_env_claude/extention_2.png =800x)
![](/images/20260209_tech_env_claude/extention_3.png =800x)
![](/images/20260209_tech_env_claude/extention_4.png =800x)
![](/images/20260209_tech_env_claude/extention_5.png =800x)
![](/images/20260209_tech_env_claude/extention_6.png =800x)
![](/images/20260209_tech_env_claude/extention_7.png =800x)

### 完成図
![](/images/20260209_tech_env_claude/env.png =400x)

## 4. おまけ
Claude Code はコマンドで利用モデルを変更できます。
![](/images/20260209_tech_env_claude/cf1.png =800x)
![](/images/20260209_tech_env_claude/cf2.png =800x)

## おわりに
本記事のポイントは、Claude Code を「手動で入れる」のではなく、devcontainer.json によって 環境のレシピ化 することです。これにより、新しいコンテナを作っても同じ構成を素早く再現でき、拡張機能の入れ直しや設定漏れを防げます。チーム開発でも個人開発でも、環境差分によるトラブルが減るのでおすすめです。