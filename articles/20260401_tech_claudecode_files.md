---
title: "Claude Code フォルダ構成ガイド：.claude/ を使うメリットと各ファイルの役割"
emoji: "🗂"
type: "tech"
topics: ["claudecode", "フォルダ構成", "入門", "初学者", "環境構築"]
published: true
---

## はじめに

以前から Claude Code を使っていましたが、もっと活用したいと思い、今更ながら `.claude/` をはじめとする推奨フォルダ構成や関連ファイルについて学び直しました。この記事では、その内容を自分なりに咀嚼し整理してまとめています。

:::details フォルダ構成を 整備しないと起きる問題 と 整備するメリット
Claude Code は、特別なフォルダ構成を用意しなくても、他の AI ツールと同様に会話するだけで利用できます。では、なぜあえて `.claude/` フォルダや `CLAUDE.md` を整備するのでしょうか。

---
### 整備しないと起きる問題

フォルダ構成を整備せずに使い続けると、次のような問題が積み重なります。

- **毎回同じ説明が必要になる**
「このプロジェクトは Python で書いてください」「コメントは日本語で」といった指示を、会話を始めるたびに繰り返さなければならない
- **チームメンバーとルールがずれる**
Claude に依頼したコードのスタイルが、人によってばらついてしまう
- **危険な操作を止められない**
`rm -rf` や本番 DB への書き込みなど、意図しない破壊的操作が実行されるリスクがある
- **作業の再利用ができない**
「コミットメッセージを毎回いい感じに作ってほしい」という手順を、毎回その場で説明しなければならない
- **会話をまたいで記憶が引き継がれない**
前回の会話で伝えた好みや方針が、次の会話では失われている
---
### 整備することで得られるメリット

一方で、`.claude/` フォルダや各種設定ファイルを整備しておくと、上記の問題を減らせます。
![](/images/20260401_tech_claudecode_files/merit.png =500x)

| 設定ファイル | 解決する問題 |
|---|---|
| `CLAUDE.md` | 毎回の説明が不要になる。チームでルールを統一できる |
| `.claude/rules/` | ファイル種別ごとに細かいルールを自動適用できる |
| `.claude/skills/` | 繰り返し作業をカスタムコマンド化して再利用できる |
| `.claude/settings.json` | 危険な操作を事前にブロックできる |
| `.claude/memory/` | 会話をまたいで好みや作業スタイルを記憶できる |

本記事では、**Claude Code プロジェクトのおすすめフォルダ構成**を紹介しながら、各ファイル・フォルダが「何のためにあるのか」を解説します。
:::

### おすすめのフォルダ構成の全体像

Claude Code プロジェクトのおすすめ構成は次のとおりです。大きく分けると、**Claude Code の動き方を決めるもの**（`CLAUDE.md`、`.claude/`、`.mcp.json` など）と、**Claude Code が作業するときに読み書きする材料**（`app/`、`docs/` など）の 2 種類があります。以下で各ファイル・フォルダを順に見ていきます。

```
yyyymmdd_pjt_name/
├── CLAUDE.md                  ← プロジェクト全体の指示書
├── AGENTS.md                  ← サブエージェント向けの指示書
├── .mcp.json                  ← 外部ツール連携の設定
│
├── .claude/                   ← Claude Code の設定フォルダ（中心）
│   ├── settings.json          ←   権限・hooks・環境変数
│   ├── rules/                 ←   パス別・言語別の細かいルール
│   ├── skills/                ←   繰り返し作業の手順書
│   ├── agents/                ←   専門サブエージェントの定義
│   ├── agent_memory/          ←   サブエージェント用 memory の保存先
│   └── memory/                ←   auto memory の保存先
│
├── scripts/                   ← hooks・statusline スクリプト等
│   ├── statusline.py
│   └── hooks/
│
├── app/                       ← 実装コード一式
│   ├── source/                ←   Python スクリプト
│   │   ├── XXXXX.py
│   │   └── config/            ←   設定値（パス・パラメータ）
│   │        └── XXXXX.py
│   ├── data/                  ←   データファイル
│   │   ├── input/
│   │   └── output/
│   ├── log/                   ←   実行ログ
│   └── tests/                 ←   テストコード
│
└── docs/                         ← ドキュメント（コンテキスト含む）
     ├── 00_read_me/              ←   プロジェクト説明・機能対応表
     ├── 01_user_provided_rules/  ←   詳細ルール原本（参照専用）
     ├── 02_skills/               ←   skill の入出力ファイル置き場
     └── 10_source/               ←   コードと1対1対応するドキュメント
```



## 1. CLAUDE.md：プロジェクト全体の指示書

`CLAUDE.md` は、Claude Code に対する**プロジェクト全体の方針書**です。会話を始めるたびに自動で読み込まれるため、毎回同じ説明を繰り返す必要がなくなります。

- 何を書くか
  - 返答言語（「日本語で返答してください」など）
  - 命名規則の基本
  - build / test コマンド
  - チーム共通のワークフロー

:::details 最小構成の例
```markdown
# プロジェクト名

返答は日本語で行ってください。

## 方針
- 既存のファイル構成に合わせる
- 大きな変更の前に方針を短く説明する
- シークレット・認証情報は読まない・出力しない
```
:::



## 2. .claude/settings.json：権限・hooks・環境変数

`settings.json` は Claude Code の動作設定を一元管理するファイルです。主な項目には、危険なコマンドを禁止できる **permissions**、保存時や実行前後に自動処理を走らせる **hooks**、環境変数を定義する **env**、ステータスバー表示をカスタマイズする **statusLine** があります。

:::details 各設定項目の詳細

#### permissions（権限設定）

特定のコマンドを許可・禁止できます。`rm -rf` などの危険な操作を事前にブロックしておくと、意図しないファイル削除を防げます。

```json
{
  "permissions": {
    "deny": ["Bash(rm -rf *)"]
  }
}
```

---

#### hooks（自動処理）

ファイル保存時やコマンド実行前後に、自動で処理を走らせられます。

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [{ "type": "command", "command": "python scripts/hooks/post_write_log.py" }]
      }
    ]
  }
}
```

---

#### env（環境変数）

Claude Code プロセスが参照する環境変数を定義します。なお、API キーなどの秘密情報はここに書かず、`.env` や OS の環境変数を使いましょう。

```json
{
  "env": {
    "PYTHONPATH": "script",
    "LOG_LEVEL": "INFO",
    "DEFAULT_ENCODING": "utf-8"
  }
}
```

よく使う例:
- `PYTHONPATH` : `import` のパス解決。`script` 配下のモジュールをどこからでも参照できる
- `LOG_LEVEL` : スクリプト側でこの変数を参照し、ログ出力量を一元管理する

---

#### statusLine（ステータスバー）

ターミナル最下部のステータスバーに表示する情報をカスタマイズできます。コンテキスト使用量や現在のブランチなどを表示すると、作業状況が把握しやすくなります。

```json
{
  "statusLine": "python scripts/statusline.py"
}
```

`scripts/statusline.py` に表示したい情報を返すスクリプトを置くと、その出力がステータスバーに反映されます。

```python
# scripts/statusline.py
import subprocess

def get_branch():
    result = subprocess.run(["git", "branch", "--show-current"],
                            capture_output=True, text=True)
    return result.stdout.strip() or "no-branch"

print(f"branch:{get_branch()}")
```

コンテキスト残量を表示しておくと、Compact（会話の圧縮）を実行するタイミングが分かりやすくなります。

:::



## 3. .claude/rules/：パス別・言語別の細かいルール

`CLAUDE.md` はプロジェクト全体に効きますが、`rules/` は**特定のパス・ファイル種別にだけ適用したいルール**を置く場所です。`CLAUDE.md` が肥大化してきたときの分割先としても有効です。

:::details 詳細

```
rules/
├── XXXXX.md
└── YYYYY.md
```

ファイルの先頭に `paths:` を書くと、そのルールが効く対象ファイルを絞れます。`paths:` 指定がないファイルは、すべてのファイルに常時適用されます。

```markdown
---
paths:
  - "app/source/**/*.py"
---

# Python コード規約
pd.read_csv() には必ず dtype=str を指定すること。
str.contains() には regex=False, na=False を指定すること。
```
:::



## 4. .claude/skills/：繰返作業の手順書（カスタムコマンド）

### 4-1. 概要

`skills/` は、**何度も繰り返す作業手順をカスタムコマンド化**して置く場所です。`/<skill名>` で呼び出せます。

```
skills/
├── commit/                     → /commit でコミットメッセージを自動生成
├── meeting_insights_analyzer/  → /meeting_insights_analyzer で会議録の分析
└── skill-creator/              → /skill-creator で新しいスキルを作成
```

### 4-2. スキルフォルダの内部構成

各スキルは `skills/<skill_name>/` 配下に以下のファイル・フォルダを置けます。

```
skills/create_zenn_article/
├── SKILL.md          ← 必須。手順・フロントマター・呼び出し条件を記述
├── README.md         ← 任意。使い方・引数・制約の説明（人間向け）
├── assets/           ← 任意。スキルが参照するテンプレート・スタイルガイド
│   └── article_style_guide.md
├── references/       ← 任意。外部仕様や長いリファレンスドキュメント
│   └── api_spec.md
├── scripts/          ← 任意。スキルが呼び出す補助スクリプト
│   └── sanitize.py
└── evals/            ← 任意。スキルの品質評価用テストケース
    └── evals.json
```

| フォルダ / ファイル | 役割 |
|---|---|
| `SKILL.md` | **唯一の必須ファイル**。Claude がこれを読んでスキルを実行する |
| `README.md` | 人間が読む使い方ガイド。`SKILL.md` に書くには長すぎる補足を置く |
| `assets/` | スキルが参照するテンプレート・スタイルガイド・雛形など |
| `references/` | 外部 API 仕様や長いリファレンスドキュメント（必要時だけ読み込む） |
| `scripts/` | スキルの手順から呼び出す補助スクリプト |
| `evals/` | スキルの動作品質を測るテストケース（`skill-creator` で使用） |

`SKILL.md` さえあれば動くので、シンプルなスキルなら 1 ファイルで十分です。作業が複雑になるにつれて `assets/` や `references/` を追加していくのがおすすめです。

:::details SKILL.md の最小構成

以下の `SKILL.md` を `.claude/skills/commit/SKILL.md` に置くだけで、`/commit` コマンドが使えるようになります。

```markdown
---
name: commit
description: 変更をコミットする。「コミットして」と言われたときに使う。
---

## 手順
1. git status で変更を確認する
2. 変更内容を分析してコミットメッセージを作成する
3. git add して git commit する
```
:::

### 4-3. おすすめ skills

- **[anthropics/skills](https://github.com/anthropics/skills/tree/main/skills)**
Anthropic 公式のスキル集。PDF・Word・Excel の生成、フロントエンド設計、API 連携など 17 種類を収録。公式のため品質・互換性の面で安心して使える。
- **[claude-skill-sanitize-log](https://github.com/okazaky/claude-skill-sanitize-log)**
Claude Code の会話ログからメールアドレス・API キー・パスワードなどの個人情報を自動マスクするスキル。ログを外部に共有する前のサニタイズに便利。
- **[awesome-claude-skills](https://github.com/BehiSecc/awesome-claude-skills)**
コミュニティ運営のスキルカタログ。開発ツール・データ分析・セキュリティなど 14 カテゴリ・100 以上のスキルが分類されており、スキルを探すときの索引として使える。
- **[notebooklm-py](https://github.com/teng-lin/notebooklm-py)**
Google NotebookLM を Python から操作できる非公式ライブラリ。ノートブック作成・ソース一括インポート・ポッドキャスト/スライド/クイズの自動生成など、Web UI にはない機能もカバー。Claude Code のスキルとしても組み込める。



## 5. .claude/agents/：専門サブエージェントの定義

`agents/` は、**特定の役割に特化した小型 AI エージェント**を定義する場所です。サブエージェントは**独立したコンテキスト**で動くため、メインの会話を汚さずに分業できます。「レビューは別の AI に任せる」「並列で 2 つの作業を走らせる」といった使い方が可能です。

最初のうちは使わなくても問題ありません。作業が複雑になってきたら導入を検討してみてください。
```
agents/
├── plan_reviewer.md         ← 実装計画のレビュー専門
├── safe_bash_researcher.md  ← 安全なBash調査専門
└── setup_skills_parallel.md ← 複数スキルを並列セットアップ
```





## 6. .claude/memory/ と agent_memory/：auto memory の保存先

Claude Code には **auto memory** 機能があります。会話の中で学んだユーザーの好みや作業スタイルを自動保存し、**次の会話でも記憶を引き継ぎます**。`settings.json` で `"autoMemoryEnabled": true` にすると有効になります。

auto memory には **2 種類の保存先** があります。メインの Claude とサブエージェントで記憶が分かれているため、互いの記憶が混ざって混乱するのを防げます。

| | `memory/` | `agent_memory/` |
|---|---|---|
| **書き込む主体** | メイン会話の Claude | サブエージェント<br>（`.claude/agents/` 配下） |
| **読み込む<br>タイミング** | メイン会話の開始時 | サブエージェント起動時 |
| **用途** | ユーザーの好み・プロジェクト方針・<br>フィードバック | エージェント固有の知識・手順の<br>チューニング結果 |


memory はプレーンな Markdown ファイルなので、**人間が直接読み書きできます**。確認・編集の方法は主に 2 つあります。Claude が誤った記憶を保存してしまった場合や、古くなった情報を整理したいときは、遠慮なく手動で修正してください。

- **`/memory` コマンド** : Claude Code 内で実行すると、memory ファイルの一覧確認や内容の編集ができる
- **テキストエディタで直接編集** : memory ファイルは通常の Markdown なので、VS Code などで開いて修正・削除しても問題ない


:::details memory ファイルの書き方

各記憶ファイルは YAML フロントマター付きのマークダウンで保存されます。`type` には `user`（ユーザー情報）、`feedback`（指摘・修正履歴）、`project`（プロジェクト状況）、`reference`（外部リソースへのポインタ）の 4 種類があります。以下にそれぞれの例を挙げます。

```markdown
---
name: ユーザーの役割
description: データ加工担当エンジニア。説明は具体例ベースが伝わりやすい。
type: user
---

データ加工を主に担当するエンジニア。
Python（pandas）歴3年。Claude Code は今回が初めて。
説明は具体例ベースの方が伝わりやすい。
```
```markdown
---
name: 出力スタイルに関するフィードバック
description: 箇条書きだけで終わらせず、前後に説明文を入れる。
type: feedback
---

箇条書きだけの説明だとやや機械的に見えるため、
導入文と補足文を入れて、読みやすい文章にすること。
コード例を出すときは、何を意図した例なのかを先に一文で説明する。
```
```markdown
---
name: 売上集計バッチ改修プロジェクト
description: pandas ベースの既存処理を整理し、保守しやすい構成へ改善中。
type: project
---

対象は売上CSVを集計する社内バッチ処理。
現在は app/source/ 配下のスクリプト整理と、設定値の config 分離を進めている。
今後はログ出力の統一と、テストコードの追加を予定している。
```
```markdown
---
name: 参照ドキュメント一覧
description: 実装時によく参照する外部ドキュメントへのリンク集。
type: reference
---

- pandas公式ドキュメント
- 社内のCSV仕様書
- ログ出力ルールの設計メモ
- バッチ実行手順書
```
:::



## 7. AGENTS.md：サブエージェント向けの指示書

`AGENTS.md` は `CLAUDE.md` と似ていますが、**サブエージェントが起動したときに追加で読み込まれる指示書**です。通常の会話では `CLAUDE.md` だけが読まれ、サブエージェントが起動すると `AGENTS.md` も参照されます。最初のうちは `CLAUDE.md` だけで十分です。



## 8. .mcp.json：外部ツール連携の設定

**MCP（Model Context Protocol）** は、Claude Code と外部ツールをつなぐオープン規格です。Claude Code の **built-in tools**（Bash・Read・Edit など）だけでは足りない操作を、**MCP server** 経由で追加できます。

#### built-in tools と custom tools（MCP）の違い

| | built-in tools | custom tools（MCP） |
|---|---|---|
| **追加方法** | 不要（最初から使える） | `.mcp.json` に MCP server を登録 |
| **例** | Bash, Read, Edit, Grep, WebFetch | DB 接続, GitHub API, ブラウザ操作 |
| **自作** | できない | MCP SDK で自作可能 |

:::details .mcp.json の設定例

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}" }
    }
  }
}
```
:::

使わない場合は空ファイルのままで構いません。まずは built-in tools だけで始めて、「この操作を自動化したい」という具体的なニーズが出てきてから導入するのがおすすめです。



## 9. app/ と docs/：実装コードとドキュメント

| フォルダ | 内容 |
|---|---|
| `app/source/` | Python スクリプト（実装コード） |
| `app/data/` | 入出力データ（CSV、JSON など） |
| `app/tests/` | pytest 等のテストコード |
| `docs/00_read_me/` | プロジェクト全体の説明書 |
| `docs/01_user_provided_rules/` | 長い詳細ルール原本（常時読み込まず必要時だけ参照） |
| `docs/02_skills/` | skill の入出力ファイル置き場 |
| `docs/10_source/` | コードファイルと 1 対 1 対応するドキュメント |

`CLAUDE.md` に書くには長すぎるルールは `docs/01_user_provided_rules/` に置き、必要なときだけ `@docs/01_user_provided_rules/xxx.md` と指定して参照させる運用が効率的です。



## Appendix 1. 拡張機能の全体像：tool・skill・hook・MCP・plugin の違い

### 全体像
ポイントは、**MCP は Claude Code の内蔵機能ではなく、Claude Code が外部ツール接続に利用するオープン規格**だということ。そして **plugin はこれらをまとめて配布する箱**だということです。
![](/images/20260401_tech_claudecode_files/overall.png =400x)

### tool と skill の違い

ここが**最も混乱しやすいポイント**です。

- **tool = 能力**（Claude が実行に使う「手」）
- **skill = その能力の使い方をまとめたレシピ**（Claude に渡す「手順書」）

skill は新しい tool を増やすものではありません。`SKILL.md` に「どう進めるか」を書いておくと、Claude がその手順を読み込み、既存の tools を組み合わせて作業します。

| | tool | skill |
|---|---|---|
| **性質** | 実行機能そのもの | 実行手順の定義 |
| **例** | Read でファイルを読む、<br>Bash でコマンドを実行する | 「git status → 差分分析 → <br>コミットメッセージ生成 → commit」 |
| **増やし方** | MCP server を接続 | `SKILL.md` を作成 |
| **使い分け** | 新しく外部 API や DB を触りたい | 毎回同じ手順を再利用したい |

### 各概念のまとめ

| 概念 | 一言で言うと | 対応するファイル・フォルダ |
|---|---|---|
| **tool**（built-in） | Claude が使う標準実行手段 | Claude Code 本体に内蔵 |
| **tool**（custom） | MCP で追加した外部実行手段 | `.mcp.json` で接続 |
| **skill** | Claude に渡す手順書・作業レシピ | `.claude/skills/` |
| **hook** | イベント駆動の自動処理 | `.claude/settings.json` + `scripts/hooks/` |
| **MCP** | 外部ツール接続のオープン規格 | `.mcp.json` |
| **plugin** | 上記をまとめる配布パッケージ | skills / agents / hooks / MCP servers を内包 |

**実務での使い分けの目安:**

- 社内 API や DB を Claude から触りたい → **MCP** server を作る・つなぐ
- 定型作業を毎回同じ流れでやらせたい → **skill**（`SKILL.md` を作る）
- 保存時チェック・イベント連動を自動化したい → **hook**
- チームに設定をまとめて配りたい → **plugin**
- まず何も足さずに始める → **built-in tools** で十分



## Appendix 2. どこに何を書くか早見表

| 書きたいこと | 置き場所 |
|---|---|
| プロジェクト全体の方針・言語設定 | `CLAUDE.md` |
| 特定パスだけに効かせたいルール | `.claude/rules/` |
| 繰り返し使う作業手順（カスタムコマンド） | `.claude/skills/` |
| ツール許可・hooks・環境変数 | `.claude/settings.json` |
| 専門役割のサブエージェント定義 | `.claude/agents/` |
| 会話をまたぐ記憶・好みの保存 | `.claude/memory/` |
| 長い詳細ルール原本（参照専用） | `docs/01_user_provided_rules/` |
| 全プロジェクト共通の設定 | `~/.claude/` |
| MCP サーバー設定 | `.mcp.json` |

**最初に整備する優先順位:**

1. `CLAUDE.md`（自作が面倒なら`/init` で自動生成）
2. `.claude/settings.json`（permissions で危険操作をブロック）
3. `.claude/skills/`（よく使う作業をコマンド化）
4. `.claude/rules/`（`CLAUDE.md` が肥大化してきたら分割）



## Appendix 3. プロンプト・コンテキスト・ハーネス

Claude Code を使い込んでいくと、「プロンプトをどう書くか」だけでなく、「**Claude を取り巻く環境をどう整えるか**」が成果に大きく影響することに気づきます。

### プロンプト・コンテキスト・ハーネス の関係

この 3 つは、**内側から外側へ広がる入れ子の関係**です。
![](/images/20260401_tech_claudecode_files/PCH.png =200x)

| 概念 | 一言で言うと | 具体例 |
|---|---|---|
| **プロンプト** | モデルへの指示文 | 「失敗しているテストを直して」 |
| **コンテキスト<br>エンジニアリング** | モデルに何を見せるか<br>の設計 | 失敗ログ・関連ファイル・規約・差分を選んで渡す |
| **ハーネス** | モデルを動かし続ける<br>実行枠組み | テスト実行 → 探索 → 修正 → 再テスト → <br>保存 → 次セッションに引き継ぎ |

- **プロンプト** 
「こうして」と伝える言葉
- **コンテキストエンジニアリング** 
「どんな情報配置ならモデルが望ましい振る舞いをしやすいか」を設計すること。prompt だけでなく、ツール定義・MCP・外部データ・会話履歴なども含む
- **ハーネス** 
prompt や context をどう使うかだけでなく、**実行フロー自体をどう組むか**まで含む外枠

Anthropic は context engineering を prompt engineering の自然な発展形として、harness design を長時間エージェントの性能を左右する鍵として、それぞれ位置づけています。

### Claude Code におけるハーネス

Claude Code の文脈では、**ハーネス = Claude Code が動く土台となる設定・ルール・ツール・メモリの総体**です。本記事で紹介してきたファイル群がまさにハーネスにあたります。**毎回同じプロンプトを書いているなら、それはハーネスに移すべきサイン**です。

| | プロンプト | ハーネス |
|---|---|---|
| **対象** | 会話中の指示・質問の書き方 | Claude を取り巻く環境の設計 |
| **効果の範囲** | そのセッション内 | 全セッション・チーム全体 |
| **例** | 「ステップバイステップで考えて」 | `CLAUDE.md` にルールを書く、<br>permissions で危険操作をブロック |
| **蓄積性** | 会話が終われば消える | ファイルとして残り、次回以降も有効 |


本記事で紹介してきたフォルダ構成は、ほとんどハーネスの構成要素です。つまり、**フォルダ構成を整備する = ハーネスを整備する**ということです。プロンプトを磨くだけでなくハーネスを整えることで、Claude Code の出力品質と再現性が大きく向上します。

| ファイル | ハーネスとしての役割 |
|---|---|
| `CLAUDE.md` | プロジェクト方針のハーネス化 |
| `.claude/rules/` | コーディング規約のハーネス化 |
| `.claude/skills/` | 定型作業のハーネス化 |
| `.claude/settings.json` | 安全性・自動処理のハーネス化（permissions / hooks） |
| `.claude/memory/` | 学習・好みのハーネス化 |
| `.claude/agents/` | 作業の分業・並列化のハーネス化 |
| `.mcp.json` | 外部ツール接続のハーネス化 |


> **文章で何度も言うより、設定で縛る方が得。** 危険コマンドの禁止のような「守らせたいこと」は、`CLAUDE.md` に書くより `settings.json` の permissions に置く方が、コンテキストを消費せず確実に効きます。



## Appendix 4. コンテキストの消費と節約

Claude Code は会話のやり取りを**コンテキストウィンドウ**に蓄積しながら動きます。このコンテキストには上限があり、使い切ると過去のやり取りが自動的に圧縮（Compaction）されます。圧縮されると細かいニュアンスが失われることがあるため、**コンテキストの消費を意識した運用**が重要です。

### コンテキスト管理の考え方

コンテキストとは「前提情報・短期記憶」です。1 セッションで扱える情報量に限りがあるため、**長期的に残したい情報は memory に、一時的な作業指示はコンテキストに**という使い分けが基本です。
![](/images/20260401_tech_claudecode_files/context.png =450x)

また、Claude Code 本体も内部で圧縮を行っています。会話履歴を要約し、最近アクセスしたファイルを残す Compaction が自動的に走ります。つまり、**ユーザー側のフォルダ構成は「何を最初から読ませるか」の最適化**、**内部の Compaction は「膨らんだ履歴をどう縮めるか」の最適化**という役割分担です。


### Claude Code のフォルダ構成自体が「節約の仕組み」

実は、本記事で紹介してきたフォルダ構成そのものが、コンテキスト節約を意識した設計になっています。基本思想は **「共通知識は薄く、詳細は遅延ロード、重い作業は別コンテキスト」** です。

| ファイル・フォルダ | ロードのタイミング | 節約のポイント |
|---|---|---|
| `CLAUDE.md`（ルート） | **毎回**自動読み込み | 短く保つ（200 行以内が目安） |
| `CLAUDE.md`（サブディレクトリ） | 配下のファイルを<br>読んだ時に**遅延ロード**&emsp; | 関係ないディレクトリの説明は<br>読み込まれない |
| `.claude/rules/`（`paths:` なし） | **毎回**読み込み | 共通ルールのみ。短く |
| `.claude/rules/`（`paths:` あり） | 一致するファイルを<br>読んだ時だけ | Python 規約は<br> `.py` 編集時だけ適用される |
| `.claude/skills/` | **説明文だけ**常駐。<br>本文は呼び出し時 | 長い手順書は<br>ここに逃がすとお得 |
| `.claude/agents/` | 定義は読まれるが、<br>**実行は別コンテキスト** | 重い調査を<br>メインから切り離せる |
| `memory/`（`MEMORY.md`） | 先頭 200 行のみ | 詳細は別の topic ファイルに分離 |
| `settings.json` | 設定として処理<br>（コンテキスト外） | permissions 等は文章で書くより<br>設定で縛る方が得 |

:::details 何がコンテキストを消費するか

| 消費するもの | 説明 |
|---|---|
| **会話の往復** | ユーザーの指示と Claude の応答が積み重なる |
| **読み込んだファイル** | Read ツールで開いたファイルの内容 |
| **CLAUDE.md / rules** | セッション開始時に自動読み込みされる |
| **MCP server の定義** | 有効な MCP server が多いほど、ツール定義でコンテキストを圧迫する |
| **memory** | `MEMORY.md` の先頭 200 行が毎回読み込まれる |
| **skill の説明文** | 各スキルの `description` が常時コンテキストに入る<br>（本文は呼び出し時のみ） |

MCP をたくさん有効にすると、ツール定義や出力でコンテキストが大きく圧迫されることがあるようです。有効化は 10 個以下、ツール合計は 80 個以下を目安とした方が良さそうです。
:::
:::details 節約のためにユーザーができること

- **`/compact`**
会話を手動で圧縮し、要点だけ残す。長い作業の途中で定期的に実行すると効果的。
- **`/context`**
現在のコンテキスト使用量を確認する。statusLine に表示しておくと便利。
- **rules の分割**
`CLAUDE.md` にすべてを書かず、`paths:` 指定付きの `.claude/rules/` に分ける。関係ないファイルを編集中はそのルールが読み込まれない。
- **長いルール原本を分離**
`docs/01_user_provided_rules/` に置き、必要なときだけ `@` で参照する（ただし `@` import は起動時に展開されるため、常時 import すると節約にならない点に注意）。
- **subagent の活用**
サブエージェントは独立したコンテキストで動くため、メインのコンテキストを消費しない。
- **MCP の取捨選択**
使わない MCP server は無効化しておく。
- **`claudeMdExcludes`**
モノレポで他チームの `CLAUDE.md` が拾われてしまう場合、`settings.local.json` の `claudeMdExcludes` で除外できる。
- **HTML コメントの活用**
`CLAUDE.md` 内の HTML コメント（`<!-- -->`）はコンテキスト注入前に除去されるため、保守者向けメモを残してもトークンを消費しない。
:::





## Appendix 5. 参考リンク

- [ハッカソン優勝者の Claude Code 設定集「everything-claude-code」を読み解く](https://zenn.dev/ttks/articles/a54c7520f827be)
agents/skills/rules/hooks/MCP の実践的な設計思想と、MCP を有効化しすぎるとコンテキストが縮小する注意点まで解説
- [Claude Code で AI マルチエージェント「multi-agent-shogun」を作った話](https://zenn.dev/shio_shoppaize/articles/5fee11d03a11a1)
tmux + Claude Code で将軍・家老・足軽の階層型 AI チームを構築し、人間は承認だけの運用に至った経験報告
- [Claude Code の Agent Teams をたった 2 つのプロンプトで試す方法](https://x.com/masa_oka108/status/2019759475973083187)
Agent Teams 機能を最速で導入するための手順紹介
- [Anthropic が Claude Code 16 体チームで C コンパイラを作らせた知見の整理](https://x.com/__satoshissss__/status/2020145601678303607)
Opus 4.6 × 約 2,000 セッションで Rust 製 C コンパイラを構築した事例から、普段の Claude Code 運用に転用できる Tips を整理
- [Claude Code に長期記憶を持たせたら壁打ちの質が変わった](https://zenn.dev/noprogllama/articles/7c24b2c2410213)
SQLite + 日本語特化ベクトル検索で自作した記憶エンジン「sui-memory」の設計と、1,942 セッション分の運用結果



## おわりに
最初は「フォルダを整備しなくても動くなら、そのままでいいのでは？」と思っていたのですが、実際に `CLAUDE.md` と `.claude/skills/` を整えてみると、毎回の説明コストがぐっと減り、Claude との会話がスムーズになりました。この記事が「どこに何を置けばいいか分からない」という方の参考になれば幸いです。