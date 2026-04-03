# Claude Code 説明書

> Claude Code は Anthropic が開発した AI コーディングアシスタントです。ターミナル、IDE、デスクトップアプリ、ブラウザから利用できます。

---

## 目次

1. [概要](#1-概要)
2. [インストール](#2-インストール)
3. [基本的な使い方](#3-基本的な使い方)
4. [スラッシュコマンド一覧](#4-スラッシュコマンド一覧)
5. [キーボードショートカット](#5-キーボードショートカット)
6. [パーミッションモード](#6-パーミッションモード)
7. [MCP サーバー](#7-mcp-サーバー)
8. [ホック（自動化）](#8-ホック自動化)
9. [IDE 統合](#9-ide-統合)
10. [CLAUDE.md（プロジェクト設定）](#10-claudemdプロジェクト設定)
11. [ベストプラクティス](#11-ベストプラクティス)
12. [トラブルシューティング](#12-トラブルシューティング)

---

## 1. 概要

### 主な機能

- コードベース全体の理解・解析
- ファイルの読み取り・編集
- コマンド実行（テスト・ビルド・Lint など）
- Git 統合（コミット・PR 作成など）
- 外部ツールとの連携（MCP）
- 複数エージェントの並列実行
- スケジュール実行

### 対応プラットフォーム

| プラットフォーム | 利用方法 |
|----------------|---------|
| macOS / Linux / Windows | CLI（ターミナル） |
| Mac / Windows | デスクトップアプリ |
| ブラウザ | claude.ai/code |
| VS Code / JetBrains | IDE 拡張機能 |

---

## 2. インストール

### macOS / Linux

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Homebrew を使用する場合：

```bash
brew install --cask claude-code
```

### Windows

```powershell
# PowerShell
irm https://claude.ai/install.ps1 | iex
```

```powershell
# WinGet
winget install Anthropic.ClaudeCode
```

### 認証

インストール後、初回実行時に自動でログイン画面が表示されます。
手動でログインする場合：

```bash
claude auth login
```

対応アカウント：
- Claude Pro / Max / Team / Enterprise
- Claude Console（API 利用）
- Amazon Bedrock / Google Vertex AI / Microsoft Foundry

---

## 3. 基本的な使い方

### インタラクティブモード

```bash
# セッション開始
claude

# 初期プロンプト付きで開始
claude "このコードのバグを修正して"

# 直前のセッションを継続
claude -c

# セッションを再開（選択肢から選ぶ）
claude -r
```

### プリントモード（非インタラクティブ）

```bash
# 単発で実行して終了
claude -p "この関数を説明して"

# パイプで入力を処理
cat error.log | claude -p "エラーを分析して"

# JSON 形式で出力
claude -p "クエリ" --output-format json
```

---

## 4. スラッシュコマンド一覧

セッション中に `/` から始まるコマンドを入力することで実行できます。

### セッション管理

| コマンド | 説明 |
|---------|------|
| `/clear` | 会話履歴をリセット |
| `/rename` | セッションに名前を付ける |
| `/fork` | 現在のセッションを分岐 |
| `/rewind` | 前の状態に戻す |
| `/compact` | コンテキストを圧縮 |
| `/cost` | トークン使用量と費用を表示 |

### 設定・管理

| コマンド | 説明 |
|---------|------|
| `/config` | 設定画面を開く |
| `/memory` | CLAUDE.md を表示・編集 |
| `/hooks` | ホックを管理 |
| `/mcp` | MCP サーバーを管理 |
| `/plugin` | プラグインを管理 |
| `/keybindings` | キーバインディングを設定 |

### モデル・動作

| コマンド | 説明 |
|---------|------|
| `/model` | モデルを選択 |
| `/thinking` | 拡張思考を有効化 |
| `/plan` | プランモードに切り替え |

### 認証・接続

| コマンド | 説明 |
|---------|------|
| `/login` | アカウントにログイン |
| `/logout` | ログアウト |
| `/ide` | IDE に接続 |
| `/help` | ヘルプを表示 |

---

## 5. キーボードショートカット

### グローバル

| ショートカット | 機能 |
|--------------|------|
| `Ctrl+C` | 操作をキャンセル |
| `Ctrl+D` | Claude Code を終了 |
| `Ctrl+L` | 画面を再描画 |
| `Ctrl+T` | タスクリスト表示 / 非表示 |
| `Ctrl+R` | コマンド履歴を検索 |

### チャット入力

| ショートカット | 機能 |
|--------------|------|
| `Enter` | メッセージを送信 |
| `Shift+Enter` | 改行を挿入（送信しない） |
| `Shift+Tab` | パーミッションモードを切り替え |
| `Ctrl+G` | 外部エディタで開く |
| `Ctrl+V` | 画像を貼り付け（Windows: `Alt+V`） |
| `Cmd/Ctrl+P` | モデル選択 |
| `Cmd/Ctrl+T` | 拡張思考の切り替え |
| `Cmd/Ctrl+O` | ファストモードの切り替え |

### カスタムキーバインド

`~/.claude/keybindings.json` を作成して設定できます：

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor"
      }
    }
  ]
}
```

---

## 6. パーミッションモード

Claude Code が実行できる操作の範囲を制御します。`Shift+Tab` でセッション中に切り替えられます。

| モード | 説明 | 用途 |
|-------|------|------|
| `default` | 各操作前に確認を求める | デフォルト・慎重な作業 |
| `acceptEdits` | ファイル編集を自動許可（コマンドは確認） | コード反復作業 |
| `plan` | ファイル読み取りのみ（編集不可） | コードベース分析・計画立案 |
| `auto` | 安全なコマンドを自動実行 | 長時間タスク |
| `bypassPermissions` | すべてのチェックをスキップ | 隔離環境（コンテナ・VM）のみ |

### 起動時にモードを指定

```bash
claude --permission-mode plan
```

### 設定ファイルでデフォルト指定

`.claude/settings.json`:

```json
{
  "permissions": {
    "defaultMode": "acceptEdits",
    "allow": [
      "Bash(npm test)",
      "Bash(npm run lint)"
    ],
    "deny": [
      "Bash(rm -rf *)"
    ]
  }
}
```

---

## 7. MCP サーバー

Model Context Protocol（MCP）を使って、外部ツールやサービスと連携できます。

### MCP サーバーの追加

```bash
# HTTP トランスポートで追加
claude mcp add --transport http <名前> <URL>

# 例：GitHub MCP
claude mcp add --transport http github https://api.githubcopilot.com/mcp/
```

または `.claude/mcp.json` で設定：

```json
{
  "mcpServers": {
    "postgresql": {
      "command": "node",
      "args": ["pg-mcp-server.js"],
      "env": {
        "DATABASE_URL": "postgresql://localhost/mydb"
      }
    }
  }
}
```

### 主な MCP サーバー例

| サーバー | 用途 |
|---------|------|
| GitHub | リポジトリ検索・PR 操作・issue 管理 |
| PostgreSQL | データベースクエリ |
| Slack | メッセージ送信・検索 |
| Google Drive | ドキュメント管理 |
| Sentry | エラーモニタリング |
| Filesystem | ローカルファイルアクセス |

### MCP の管理

インタラクティブモードで `/mcp` コマンドを使用するか：

```bash
# セッション中に MCP を管理
/mcp
```

---

## 8. ホック（自動化）

ホックは Claude Code のライフサイクル内で自動実行されるスクリプトです。

### ホックイベント

| イベント | タイミング |
|---------|---------|
| `SessionStart` | セッション開始時 |
| `UserPromptSubmit` | プロンプト送信時 |
| `PreToolUse` | ツール実行前（ブロック可能） |
| `PostToolUse` | ツール実行成功後 |
| `PostToolUseFailure` | ツール実行失敗後 |
| `Notification` | 通知送信時 |
| `SessionEnd` | セッション終了時 |

### 設定例

`.claude/settings.json` に記述します。

**ファイル編集後に Prettier を実行：**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
          }
        ]
      }
    ]
  }
}
```

**macOS 通知を送信：**

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude Code の操作が完了しました\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

### 設定ファイルの場所とスコープ

| ファイル | スコープ | Git 共有 |
|---------|---------|---------|
| `~/.claude/settings.json` | 全プロジェクト | なし |
| `.claude/settings.json` | プロジェクト単位 | 可能 |
| `.claude/settings.local.json` | プロジェクト単位（個人） | なし |

---

## 9. IDE 統合

### VS Code

**インストール：**

1. `Cmd+Shift+X`（Mac）/ `Ctrl+Shift+X`（Windows/Linux）
2. 「Claude Code」を検索してインストール

**主なショートカット：**

| ショートカット | 機能 |
|--------------|------|
| `Cmd/Ctrl+Esc` | エディタと Claude を切り替え |
| `Cmd/Ctrl+Shift+Esc` | 新規タブで会話開始 |
| `Option/Alt+K` | @-メンション参照を挿入 |

### JetBrains IDE

**インストール：**

1. JetBrains Marketplace から「Claude Code」プラグインをインストール
2. IDE を再起動

**主なショートカット：**

| ショートカット | 機能 |
|--------------|------|
| `Cmd/Ctrl+Esc` | Claude Code を開く |
| `Cmd+Option/Alt+Ctrl+K` | ファイル参照を挿入 |

### IDE から Claude Code を起動

```bash
# IDE の統合ターミナルから実行
claude

# 外部ターミナルから IDE に接続
/ide
```

---

## 10. CLAUDE.md（プロジェクト設定）

`.claude/CLAUDE.md` にプロジェクト固有の指示を記述することで、Claude Code の動作をカスタマイズできます。

### 書き方の例

```markdown
# プロジェクト概要
このプロジェクトは React + TypeScript で構築された EC サイトです。

# コードスタイル
- ES モジュール構文（import/export）を使用
- 関数は arrow function で記述
- 型には interface より type を優先

# ワークフロー
- コード変更後は `npm run type-check` を実行
- テストは `npm test` で実行
- Lint は `npm run lint` で実行

# アーキテクチャ
- API は REST 仕様に従う
- URL は kebab-case（例：/user-profiles）
- JSON プロパティは camelCase
```

### チームで共有

```bash
git add .claude/CLAUDE.md
git commit -m "Add Claude Code project instructions"
```

---

## 11. ベストプラクティス

### 具体的なプロンプトを書く

**悪い例：**
```
このバグを修正してください
```

**良い例：**
```
ログアウト後に再ログインすると「Token not found」エラーが発生します。
src/auth/tokenRefresh.ts のトークンリフレッシュ処理を確認して修正してください。
修正後にテストを実行して確認してください。
```

### コンテキスト管理

```bash
# コンテキストを圧縮（容量が多くなってきたら）
/compact

# 別タスクを始めるときはリセット
/clear

# 前の状態に戻す
/rewind
```

### 並列実行

```bash
# 複数のセッションを並列実行
claude -w feature-auth    # 認証機能
claude -w feature-search  # 検索機能

# スクリプトで複数ファイルを処理
for file in src/**/*.ts; do
  claude -p "TypeScript 5.0 対応にマイグレーション: $file"
done
```

### 検証を含めたプロンプト

```
validateEmail 関数を実装してください。
以下のテストケースを作成し、すべてパスすることを確認してください：
- user@example.com → true
- invalid → false
- user@.com → false
```

---

## 12. トラブルシューティング

| 問題 | 対処法 |
|------|--------|
| インストールできない | ネットワーク接続を確認して再実行 |
| ログインできない | `claude auth login` を手動実行 |
| コンテキストが満杯 | `/compact` で圧縮、または `/clear` で新セッション開始 |
| ホックが動かない | `/hooks` で確認、JSON 構文チェック、`chmod +x` で実行権限付与 |
| MCP が接続できない | サーバーの起動確認、URL や接続設定の確認 |
| 応答が遅い | `/compact` でコンテキスト削減、または `/clear` でリセット |

---

## セキュリティ上の注意

- `.env` ファイルなどの機密ファイルはパーミッション設定で保護する
- `bypassPermissions` モードは隔離環境（Docker コンテナ・VM）でのみ使用する
- `auto` モードは Team / Enterprise プランが必要
- 本番環境への直接実行は避ける
- 外部から取得したコンテンツはプロンプトインジェクションに注意する

---

*このドキュメントは Claude Code を使用して作成されました。*
