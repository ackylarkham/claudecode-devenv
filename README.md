# Claude Code 開発コンテナー・テンプレート

VS Code の開発コンテナー（Dev Container）を使って **Claude Code** による開発環境をすぐに構築できるテンプレートです。このリポジトリをテンプレートとして GitHub リポジトリを作成し、VS Code で開くだけで、ホスト環境に依存しない統一された AI 支援開発環境が立ち上がります。

---

## 特徴

- **Claude Code をすぐに使える** — コンテナー起動後すぐに `claude` コマンドが利用可能
- **認証情報の永続化** — Docker/Podman の Named Volume に Claude の認証情報を保持するため、コンテナーを再作成しても再ログイン不要
- **日本語対応** — タイムゾーン（Asia/Tokyo）・ロケール（ja_JP.UTF-8）・日本語フォント（Noto CJK）を設定済み
- **PlantUML 対応** — IBM Semeru JDK 21 + Graphviz をインストール済みで、PlantUML による図表作成が可能
- **開発向け VS Code 拡張機能を同梱** — GitLens、TODO Tree、Markdown All in One など実用的な拡張機能を自動インストール
- **Claude Code のパーミッション設定済み** — ファイル編集・Git 操作を自動許可しつつ、`.env` や外部ネットワーク送信は制限

---

## 動作確認環境

| ソフトウェア | バージョン |
| --- | --- |
| Podman | 5.8.2 |
| Podman Desktop | 1.27.1 |
| VS Code | 最新安定版 |
| Dev Containers 拡張機能 | 最新安定版 |

> Docker Desktop でも同様に動作します。

---

## 前提条件

以下をホスト環境にインストールしてください。

1. **VS Code** — [公式サイト](https://code.visualstudio.com/) からインストール
2. **Dev Containers 拡張機能** — VS Code 拡張機能 `ms-vscode-remote.remote-containers` をインストール
3. **Docker または Podman** — コンテナーランタイムのいずれか
   - Docker: [Docker Desktop](https://www.docker.com/products/docker-desktop/)
   - Podman: [Podman Desktop](https://podman-desktop.io/)

---

## 使い方

### 1. このリポジトリをテンプレートとして新しいリポジトリを作成

GitHub の **「Use this template」** ボタンをクリックして、自身の GitHub アカウントに新しいリポジトリを作成します。

### 2. リポジトリをクローン

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

### 3. VS Code で開く

```bash
code .
```

VS Code が起動したら、右下に表示される通知 **「Reopen in Container」** をクリックします。
通知が表示されない場合は、コマンドパレット（`Ctrl+Shift+P` / `Cmd+Shift+P`）から以下を実行します。

```text
Dev Containers: Reopen in Container
```

### 4. コンテナーのビルドと起動

初回は Docker イメージのビルドが行われます（数分かかる場合があります）。
完了すると、VS Code のターミナルに以下のようなメッセージが表示されます。

```text
========================================
✅ Claude Code開発環境 起動完了
----------------------------------------
🕐 タイムゾーン : JST +0900
🌐 ロケール    : ja_JP.UTF-8
🟢 Node.js     : v24.x.x
🤖 Claude Code : x.x.x
========================================
```

### 5. Claude Code にログイン

ターミナルで以下を実行し、Claude アカウントでログインします。

```bash
claude
```

初回ログイン後、認証情報は Named Volume に保存されます。コンテナーを再作成しても再ログインは不要です。

---

## ディレクトリ構成

```text
.
├── .cluade/
│   ├── CLAUDE.md          # Claude Code へのプロジェクト指示ファイル
│   ├── settings.json      # Claude Code のパーミッション設定
│   └── settings.local.json
├── .devcontainer/
│   ├── Dockerfile         # コンテナーイメージの定義
│   └── devcontainer.json  # Dev Container の設定
├── doc/                   # ドキュメント格納ディレクトリ
├── src/                   # ソースコード格納ディレクトリ
└── README.md
```

---

## コンテナー構成の詳細

### ベースイメージ

`node:lts-bookworm-slim`（Node.js LTS + Debian 12 Bookworm slim）

### インストール済みソフトウェア

| ソフトウェア | 用途 |
| --- | --- |
| Node.js (LTS) | JavaScript ランタイム・npm |
| Claude Code | AI 支援開発 CLI |
| IBM Semeru JDK 21 | Java ランタイム（PlantUML 実行用） |
| Graphviz | PlantUML によるダイアグラム描画 |
| Git / curl / vim 等 | 基本開発ツール |

### ポートフォワード

| ポート | 用途 |
| --- | --- |
| 3000 | フロントエンド開発サーバー（例: React, Next.js） |
| 8080 | バックエンド開発サーバー（例: Express, FastAPI） |

### VS Code 拡張機能

| 拡張機能 | 用途 |
| --- | --- |
| Claude Code | AI 支援開発 |
| GitLens / Git History | Git 強化ツール |
| Markdown All in One / markdownlint | Markdown 編集・Lint |
| PlantUML | UML 図表作成 |
| TODO Tree | TODO コメント管理 |
| Better Comments | コメント強調表示 |
| indent-rainbow | インデント可視化 |
| Code Spell Checker | スペルチェック |
| vscode-icons | アイコンテーマ |
| 日本語言語パック | UI 日本語化 |

---

## Claude Code のパーミッション設定

`.cluade/settings.json` で以下のパーミッションを設定しています。

### 許可（自動承認）

- プロジェクト内のファイルの読み取り・編集
- `npm run *`、`npm test` の実行
- `git` コマンドの実行

### 拒否

- `curl` / `wget` による外部への送信
- `.env`、`.env.local`、`./secrets/**` の読み取り

必要に応じて `.cluade/settings.json` を編集してパーミッションをカスタマイズしてください。

---

## ANTHROPIC_API_KEY の設定（オプション）

独自アプリケーションに Claude API を組み込む場合は、`.devcontainer/devcontainer.json` の以下の行のコメントを外してください。

```json
"ANTHROPIC_API_KEY": "${localEnv:ANTHROPIC_API_KEY}"
```

その後、ホスト環境に環境変数 `ANTHROPIC_API_KEY` を設定してからコンテナーを再作成します。

---

## ライセンス

MIT
