# Claude Code in Action — 試験対策ノート

---

## Claude Code とは

Claude Code はコマンドラインで動作する **AI コーディングアシスタント**。  
言語モデルとツールシステムを組み合わせ、ファイル読み書き・コマンド実行・コード分析を自律的に行う。

---

## コーディングアシスタントの仕組み

コーディングアシスタントがタスクを解決する3ステップ:

1. **コンテキスト収集** — エラー内容を理解し、関連ファイルを特定する
2. **計画策定** — 解決策を決める（コード変更・テスト実行など）
3. **実行** — ファイルを更新し、コマンドを実行する

→ ステップ1と3は**外部環境との相互作用**が必要 = ツール使用が必須

### なぜ Claude Code が強いか

- Claude は **ツール使用が得意**（見たことのないツールでも使いこなせる）
- 強いツール活用能力 → 複雑なタスク対応・拡張性・セキュリティ向上
- コードベースをインデックス化せずにナビゲートできる（コード全体を外部サーバーに送らない）

---

## インストールとセットアップ

### インストール方法

```bash
# macOS / Linux / WSL
curl -fsSL https://claude.ai/install.sh | bash

# Windows PowerShell
irm https://claude.ai/install.ps1 | iex

# macOS (Homebrew)
brew install --cask claude-code
```

### 初回起動

```bash
claude  # ← これだけで起動
```

- 初回起動時にカラーテーマ選択と **claude.ai アカウントでの認証**が必要
- API キーの代わりに claude.ai の認証情報を使用

### サードパーティプロバイダー経由の利用

Amazon Bedrock、Google Cloud Vertex AI、Microsoft Foundry 経由でも利用可能（追加設定が必要）。

---

## コンテキスト管理

### /init コマンド

新しいプロジェクトで最初に実行するコマンド。

```bash
/init
```

**動作:**
1. コードベース全体を分析
2. プロジェクトの目的・アーキテクチャを理解
3. 重要なコマンドとファイルを特定
4. **CLAUDE.md** ファイルに要約を書き込む

→ 以降のリクエストで常にこの情報がコンテキストとして使われる

---

### CLAUDE.md ファイル

プロジェクト固有の情報を Claude に伝えるファイル。

```markdown
# プロジェクト概要
このプロジェクトは...

# 重要なコマンド
- `npm run dev` — 開発サーバー起動
- `npm test` — テスト実行

# アーキテクチャ
データベーススキーマは @prisma/schema.prisma で定義されています。
```

**CLAUDE.md での `@` メンション:**
```markdown
# データベーススキーマは @prisma/schema.prisma を参照してください
```
→ 毎回のリクエストにそのファイルの内容が自動で含まれる

**他ツールとの統合:**
```markdown
@AGENTS.md  ← 他のAIツール用の設定ファイルをインポート
```
→ 重複して書く必要なし

---

### @ メンション（チャット内）

会話中に特定ファイルを参照させる:

```
@src/auth/login.ts のバグを修正してください
```

---

### /memory コマンド

セッションをまたいで記憶させたい情報を保存:

```
/memory コードスタイルは常に TypeScript strict モードを使うこと
```

---

## 変更を加える

### スクリーンショットで視覚的に指示

- UIの特定部分を変更したいとき、スクリーンショットが最も効果的
- **`Ctrl+V`**（Windows/Linux）でスクリーンショットをチャットに貼り付け（Cmd+V ではない）

### /effort コマンド（推論深度の調整）

```
/effort high   ← 高い推論深度
/effort low    ← 低い推論深度（デフォルト）
```

- 複雑なロジック問題・デバッグに使う
- 追加トークンを消費するため コスト注意

### プランモード（Plan Mode）

複雑な変更の前に計画を立てさせる:

```
/plan  ← または Shift+Tab で切り替え
```

**Plan Mode と /effort の使い分け:**

| | プランモード | /effort high |
|--|------------|--------------|
| **向き** | 広いコードベース理解、複数ファイルの変更 | 複雑なロジック、アルゴリズム問題 |
| **得意** | 「何をすべきか」を計画する | 「どうやるか」を深く考える |

→ 組み合わせも可能（追加コスト注意）

---

## 会話フローの制御

### Escape キー — 割り込み

Claude が間違った方向に進んでいるとき停止させる:
- `Esc` を押す → Claude の応答を中断
- 修正指示を入力して再スタート

**使用例:** 複数のテストを書き始めたとき → Esc → 「まず1つだけ書いて」

### ダブルEscape — 入力クリア

入力途中のテキストをクリア（送信前）。

### /compact — 会話を圧縮

コンテキストウィンドウがいっぱいになってきたとき:

```
/compact
```

- 会話履歴を要約して圧縮
- 重要なコンテキストを保持しながら空きスペースを確保

### /clear — 新しい会話を開始

完全に無関係な新しいタスクに切り替えるとき:

```
/clear
```

- 完全に新鮮なコンテキストで開始
- 前の会話を削除するわけではない（`/resume` で戻れる）

### /resume — 以前の会話を再開

```
/resume
```

---

## カスタムコマンド

繰り返す作業を**スラッシュコマンド**として自動化。

### 作成方法

```
プロジェクトディレクトリ/
└── .claude/
    └── commands/
        └── audit.md      ← /audit コマンドになる
```

**audit.md の例:**
```markdown
以下の手順で依存関係のセキュリティ監査を実行してください:
1. `npm audit` を実行して脆弱なパッケージを確認
2. `npm audit fix` で自動修正を適用
3. `npm test` でテストが通ることを確認
```

### 引数付きコマンド

```markdown
$ARGUMENTS で受け取った引数を使ってテストを実行してください:
- テストファイル: $ARGUMENTS
- カバレッジレポートを生成すること
```

```bash
# 使用例
/test src/auth.test.ts
```

**用途:** チームの規約・コーディングスタイルに沿ったボイラープレート生成、デプロイ手順など

---

## MCP サーバーとの統合

### MCP サーバーの追加

```bash
# ターミナル（Claude Code の外）で実行
claude mcp add playwright npx @playwright/mcp@latest
```

→ Claude Code に **Playwright** でブラウザを操作する能力が追加される

### パーミッション管理

初回使用時にツールごとの許可を求められる。  
毎回確認が不要な場合は `settings.json` で設定:

```json
{
  "permissions": {
    "allow": ["mcp__playwright__*"]
  }
}
```

### GitHub Actions での MCP パーミッション

CI/CD 環境では GUI がないため、各MCPサーバーのツールを明示的にリストアップする必要がある:

```yaml
- name: Run Claude Code
  uses: anthropics/claude-code-action@v1
  with:
    allowed_tools: "mcp__playwright__navigate,mcp__playwright__left_click,..."
```

---

## GitHub 統合

### セットアップ

```bash
/install-github-app
```

→ ガイドに従って:
1. GitHub に Claude Code アプリをインストール
2. API キーを追加
3. ワークフローファイルの PR が自動生成される

### デフォルトで追加される GitHub Actions

**1. メンションアクション:**
```
@claude このイシューを修正してください
```
→ Claude が:
- タスク計画を作成
- コードベース全体にアクセスして実装
- PR を作成（またはコメントに実装案を提示）

**2. PR レビューアクション:**
- PR が作成/更新されると Claude が自動でコードレビュー
- 提案をコメントとして投稿

### カスタムインストラクション

`.github/workflows/` 内のファイルにプロジェクト固有の指示を追加できる。

---

## フック（Hooks）

Claude Code のツール実行の**前後にカスタムコマンドを実行**する仕組み。

### フックの種類

| フック | タイミング | 用途 |
|--------|-----------|------|
| **PreToolUse** | ツール実行**前** | ブロック、バリデーション |
| **PostToolUse** | ツール実行**後** | フォーマット、テスト実行 |
| **Notification** | Claude が許可を求めるとき / 60秒アイドル後 | 通知送信 |
| **Stop** | Claude の応答完了時 | 最終チェック |
| **SubagentStop** | サブエージェント完了時 | - |
| **PreCompact** | compact 操作の前 | - |
| **UserPromptSubmit** | ユーザーがプロンプトを送信した直後 | 前処理 |
| **SessionStart** | セッション開始/再開時 | - |
| **SessionEnd** | セッション終了時 | - |

### フックの定義

`.claude/settings.local.json` に定義:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Read",
        "hooks": [
          {
            "type": "command",
            "command": "node $PWD/hooks/read_hook.js"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "node $PWD/hooks/format_hook.js"
          }
        ]
      }
    ]
  }
}
```

### フックスクリプトの仕組み

- **入力**: ツール呼び出しの JSON データが `stdin` から渡される
- **出力**: `stdout` への出力はClaudeへのフィードバックとなる  
- **stderr**: エラーメッセージ（Claude に表示される）
- **終了コード**: `0` = 許可、`2` = ブロック（PreToolUse のみ）

```javascript
// .env ファイルの読み込みをブロックするフック例
process.stdin.setEncoding("utf8");
let input = "";
process.stdin.on("data", (d) => (input += d));
process.stdin.on("end", () => {
    const toolArgs = JSON.parse(input);
    const readPath = toolArgs.tool_input?.file_path || "";
    
    if (readPath.includes(".env")) {
        console.error(".env ファイルは読み込めません");
        process.exit(2);  // ← ブロック
    }
    process.exit(0);  // ← 許可
});
```

**注意:** ツールごとに `tool_input` の形が異なる:
- `Read` → `{file_path: "..."}`
- `Grep` → `{pattern: "...", path: "..."}`
- `Bash` → `{command: "..."}`

### フックのセキュリティ注意事項

- スクリプトのパスは**絶対パス**を使うこと（パスインターセプト攻撃対策）
- `$PWD` プレースホルダーを使うとチーム間で設定ファイルを共有しやすい
- `settings.example.json` + `npm run setup` で `$PWD` を実際のパスに置換するパターンが一般的

### 実用的なフック例

**1. TypeScript 型チェックフック（PostToolUse）**

```javascript
// Write/Edit の後に tsc --noEmit を実行
// → 型エラーを即座に Claude にフィードバック
// → 関数シグネチャ変更時に呼び出し箇所の修正漏れを防ぐ
```

**2. コードレビューフック（PostToolUse）**

```javascript
// 特定ディレクトリのファイル変更後に別の Claude インスタンスでレビュー
// → Agent SDK を使って Claude が Claude のコードをレビューする
// → コスト・時間がかかるため、重要なディレクトリのみに限定
```

---

## Claude Code SDK

Claude Code を**プログラムから呼び出す**ライブラリ。

### インストール

```bash
mkdir sdk-demo
cd sdk-demo
npm init -y
npm install @anthropic-ai/claude-agent-sdk
# ※ @anthropic-ai/claude-code（CLI本体）とは別パッケージ
```

### 基本的な使い方

```javascript
import { query } from "@anthropic-ai/claude-agent-sdk";

const prompt = "現在のディレクトリのファイル一覧を表示してください";

for await (const message of query({ prompt })) {
    console.log(JSON.stringify(message, null, 2));
    // ツール呼び出し、ツール結果、テキストなどが流れてくる
}
```

### ツールの制限

```javascript
for await (const message of query({
    prompt,
    options: {
        allowedTools: ["Read", "Glob"]  // 許可するツールを限定
    }
})) {
    // ...
}
```

### SDK でできること

- カスタムシステムプロンプト
- MCP サーバーの指定
- フックの設定
- サブエージェント
- セッションの再開
- フックからの呼び出し（Claude が Claude をレビューするパターン）

---

## 重要まとめ

| コマンド/機能 | 説明 |
|-------------|------|
| `claude` | Claude Code を起動 |
| `/init` | プロジェクトを分析して CLAUDE.md 生成 |
| `/memory` | 情報を記憶に保存 |
| `@ファイル名` | ファイルをコンテキストに追加 |
| `Ctrl+V` | スクリーンショットを貼り付け |
| `/effort high` | 推論深度を上げる |
| `/plan` | プランモードに切り替え |
| `Esc` | Claude の実行を中断 |
| `/compact` | 会話履歴を圧縮 |
| `/clear` | 新しい会話を開始 |
| `/resume` | 前の会話を再開 |
| カスタムコマンド | `.claude/commands/*.md` ファイルを作成 |
| `claude mcp add` | MCP サーバーを追加 |
| `/install-github-app` | GitHub 統合をセットアップ |
| PreToolUse | ツール実行前フック（exit 2でブロック可） |
| PostToolUse | ツール実行後フック |
| `@anthropic-ai/claude-agent-sdk` | プログラムから呼び出すためのSDK |

### フックの出口コードまとめ

| 終了コード | 意味 | 適用 |
|-----------|------|------|
| `0` | 許可・成功 | PreToolUse / PostToolUse |
| `2` | ブロック | PreToolUse のみ |
| その他 | エラー | - |
