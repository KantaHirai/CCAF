# Claude Code 認定試験 — 英単語・用語集

試験で出やすい順に構成。★が多いほど頻出・重要。

---

## API 基礎

| 英単語 / フレーズ | 読み方・略称 | 日本語訳・説明 |
|-----------------|------------|--------------|
| ★★★ **stateless** | ステートレス | 状態を保持しない。Claude API は毎リクエストが独立しており会話履歴を自動保存しない |
| ★★★ **conversation history** | - | 会話履歴。マルチターンでは毎回完全な履歴を送る必要がある |
| ★★★ **system prompt** | - | システムプロンプト。Claudeの役割・行動指針を定義する文字列（`system`パラメータ） |
| ★★★ **max_tokens** | - | 生成する最大トークン数。必須パラメータ |
| ★★★ **temperature** | テンパラチャー | 温度。0=確定的、1=創造的。トークンのサンプリング確率を調整 |
| ★★★ **streaming** | ストリーミング | レスポンスをチャンクごとにリアルタイム送信する機能 |
| ★★ **token** | トークン | テキストの最小単位（単語や単語の一部）。APIの課金単位 |
| ★★ **tokenization** | トークナイゼーション | テキストをトークンに分割する処理 |
| ★★ **sampling** | サンプリング | 確率分布からトークンを選択するプロセス |
| ★★ **prefilling** / **assistant prefill** | プレフィリング | アシスタントメッセージを事前に入力して出力形式を強制する技法 |
| ★★ **stop sequences** | - | 生成を停止させる文字列。構造化データ出力に使用 |
| ★★ **stop_reason** | - | 生成が停止した理由（`end_turn`, `tool_use`, `max_tokens`など） |
| ★ **model ID** | - | モデルを指定する文字列（例: `claude-sonnet-4-6`） |
| ★ **message object** | - | `{role, content}`の形式を持つメッセージオブジェクト |
| ★ **content block** | - | レスポンスのコンテンツブロック（TextBlock, ToolUseBlockなど） |

---

## プロンプトエンジニアリング

| 英単語 / フレーズ | 読み方・略称 | 日本語訳・説明 |
|-----------------|------------|--------------|
| ★★★ **prompt engineering** | - | プロンプトエンジニアリング。より良い結果を得るためのプロンプト設計技術 |
| ★★★ **few-shot prompting** | フューショット | 複数の入出力例をプロンプトに含める技法 |
| ★★★ **one-shot prompting** | ワンショット | 1つの例を示す技法 |
| ★★★ **zero-shot prompting** | ゼロショット | 例なしでタスクを指示する技法 |
| ★★★ **XML tags** | - | XMLタグ。プロンプトの構造化に使用（`<instructions>`, `<document>`など） |
| ★★ **multishot prompting** | - | few-shot と同義。複数例を示す |
| ★★ **sarcasm** | サーカズム | 皮肉。few-shot でエッジケースをカバーする代表例 |
| ★★ **chain-of-thought** | CoT | 思考の連鎖。段階的に考えさせるプロンプト技法 |
| ★ **prompt injection** | - | 外部コンテンツに悪意ある指示を埋め込み、AIを操作する攻撃 |

---

## プロンプト評価（Evals）

| 英単語 / フレーズ | 読み方・略称 | 日本語訳・説明 |
|-----------------|------------|--------------|
| ★★★ **eval** / **evaluation** | イーバル | 評価。プロンプトの品質を数値で測定する |
| ★★★ **grader** | グレーダー | 採点器。出力の品質を評価するスクリプトや別のClaudeのこと |
| ★★★ **model-based grader** | - | Claudeが別のClaudeの出力を採点するグレーダー |
| ★★★ **code-based grader** | - | Pythonコードで正解と比較するグレーダー |
| ★★ **test dataset** | テストデータセット | 評価に使うテスト用の入力例の集合 |
| ★★ **test case** | テストケース | 個別のテスト入力 |
| ★★ **baseline** | ベースライン | 比較の基準となる初期プロンプト |
| ★★ **prompt versioning** | - | プロンプトのバージョン管理。新旧スコアを数値で比較 |
| ★ **ground truth** | グランドトゥルース | 正解データ |

---

## ツール使用（Tool Use）

| 英単語 / フレーズ | 読み方・略称 | 日本語訳・説明 |
|-----------------|------------|--------------|
| ★★★ **tool use** | - | ツール使用。Claudeが外部システム・APIを呼び出す仕組み |
| ★★★ **tool schema** | - | ツールのJSON Schema定義（name, description, input_schema） |
| ★★★ **JSON Schema** | - | ツールのパラメータを定義するデータ検証仕様 |
| ★★★ **tool_use block** | - | Claudeがツール呼び出しを要求するレスポンスブロック |
| ★★★ **tool_result block** | - | ツールの実行結果をClaudeに返すブロック |
| ★★★ **tool_use_id** | - | ツール呼び出しIDと結果を紐付けるID |
| ★★★ **is_error** | - | tool_resultブロック内のエラーフラグ（`True/False`） |
| ★★ **input_schema** | - | ツールの入力パラメータのスキーマ定義 |
| ★★ **tool_choice** | - | ツール選択制御。`auto`, `any`, `tool`, `none` |
| ★★ **text edit tool** | - | ファイルの差分編集に特化した組み込みツール |
| ★★ **web search tool** | - | Anthropic提供のWeb検索組み込みツール |
| ★ **agentic loop** | エージェントループ | `stop_reason == "tool_use"` の間ループしてツールを実行し続けるパターン |

---

## RAG（Retrieval Augmented Generation）

| 英単語 / フレーズ | 読み方・略称 | 日本語訳・説明 |
|-----------------|------------|--------------|
| ★★★ **RAG** | ラグ | Retrieval Augmented Generation。大きな文書から関連部分だけを検索してClaudeに渡す技術 |
| ★★★ **chunking** | チャンキング | 大きな文書を小さなテキスト片に分割すること |
| ★★★ **embeddings** | エンベディング | テキストを数値ベクトルに変換したもの。意味的類似度の計算に使用 |
| ★★★ **vector search** / **semantic search** | ベクトル検索 | 埋め込みベクトルの類似度（コサイン類似度など）で検索する手法 |
| ★★★ **BM25** | - | キーワードベースの語彙的検索アルゴリズム。ベクトル検索と組み合わせることが多い |
| ★★ **vector database** | ベクトルDB | 埋め込みベクトルを保存・検索するデータベース |
| ★★ **cosine similarity** | コサイン類似度 | ベクトル間の類似度を測る指標（1=完全一致, 0=無関係） |
| ★★ **lexical search** | レキシカルサーチ | 語彙的検索。キーワードの完全一致ベースの検索（BM25など） |
| ★★ **multi-index RAG pipeline** | - | ベクトル検索とBM25を組み合わせた高精度RAGパイプライン |
| ★★ **context window** | - | モデルが一度に処理できるトークン数の上限 |
| ★ **prompt stuffing** | - | 全文書をそのままプロンプトに入れる（非効率な手法、RAGの対比として登場） |
| ★ **rank fusion** | - | 複数の検索結果をスコア統合して最終ランキングを生成する手法 |

---

## Claude の機能

| 英単語 / フレーズ | 読み方・略称 | 日本語訳・説明 |
|-----------------|------------|--------------|
| ★★★ **extended thinking** | - | 拡張思考。最終回答前にClaudeが内部で考えを整理するスクラッチパッド機能 |
| ★★★ **budget_tokens** | - | 拡張思考に使えるトークン数の上限を指定するパラメータ |
| ★★★ **prompt caching** | - | プロンプトキャッシング。同じコンテンツの再計算をスキップしてコスト・速度を改善 |
| ★★★ **cache_control** | - | キャッシュポイントを指定するパラメータ（`{"type": "ephemeral"}`） |
| ★★★ **TTL** | Time To Live | キャッシュの有効期限。プロンプトキャッシングは**1時間** |
| ★★ **citations** | サイテーションズ | 引用機能。回答の根拠となる文書箇所を返す |
| ★★ **thinking block** | シンキングブロック | 拡張思考のスクラッチパッド内容を含むブロック |
| ★★ **Files API** | - | ファイルを事前アップロードし、リクエストごとに再送しない機能 |
| ★★ **base64** | ベース64 | 画像・PDFをAPI経由で送信するためのエンコード形式 |
| ★ **feature compatibility** | - | 機能の互換性。拡張思考は温度・プレフィリングと**非互換** |
| ★ **MIME type** | マイムタイプ | ファイルの種類を示す識別子（例: `application/pdf`, `image/jpeg`） |

---

## MCP（Model Context Protocol）

| 英単語 / フレーズ | 読み方・略称 | 日本語訳・説明 |
|-----------------|------------|--------------|
| ★★★ **MCP** | - | Model Context Protocol。Claudeにツール・リソース・プロンプトを提供する通信プロトコル |
| ★★★ **MCP server** | - | ツール・リソース・プロンプトを定義して提供する専門サービス |
| ★★★ **MCP client** | - | MCPサーバーと接続し、Claudeとの橋渡しをするクライアント実装 |
| ★★★ **tool (MCP primitive)** | - | MCPの3大プリミティブの1つ。**モデル制御**（Claudeが自律的に呼び出す） |
| ★★★ **resource (MCP primitive)** | - | MCPの3大プリミティブの1つ。**アプリ制御**（アプリがデータを取得してClaudeに渡す） |
| ★★★ **prompt (MCP primitive)** | - | MCPの3大プリミティブの1つ。**ユーザー制御**（事前定義された高品質な指示） |
| ★★★ **transport-agnostic** | - | トランスポートに依存しない。stdio/HTTP/WebSocketなど複数の通信方式をサポート |
| ★★★ **stdio** | スタジオ | Standard Input/Output。ローカルMCPサーバーの最一般的な通信方式 |
| ★★★ **ListToolsRequest** | - | クライアントがサーバーに利用可能なツール一覧を問い合わせるメッセージ |
| ★★★ **CallToolRequest** | - | クライアントがサーバーに特定のツールの実行を依頼するメッセージ |
| ★★★ **ReadResourceRequest** | - | クライアントがリソース内容を取得するメッセージ |
| ★★ **FastMCP** | - | Python SDK の MCP サーバー実装クラス（`from mcp.server.fastmcp import FastMCP`） |
| ★★ **@mcp.tool()** | - | Python SDK でツールを定義するデコレータ |
| ★★ **@mcp.resource()** | - | Python SDK でリソースを定義するデコレータ |
| ★★ **@mcp.prompt()** | - | Python SDK でプロンプトを定義するデコレータ |
| ★★ **Field()** | フィールド | Pydanticの型、MCPでパラメータの説明を付けるために使用 |
| ★★ **direct resource** | - | 固定URIを持つ静的リソース（例: `docs://list`） |
| ★★ **templated resource** | - | パラメータを持つ動的リソース（例: `docs://{doc_id}`） |
| ★★ **ClientSession** | - | MCP Python SDK の実際の接続オブジェクト |
| ★★ **MCP Server Inspector** | - | `mcp dev server.py` で起動するブラウザベースのデバッグUI |
| ★ **context injection** | - | リソース内容を自動的にプロンプトに埋め込む技法 |
| ★ **autocomplete** | - | オートコンプリート。リソース一覧をUIに表示する活用例 |

---

## エージェントとワークフロー

| 英単語 / フレーズ | 読み方・略称 | 日本語訳・説明 |
|-----------------|------------|--------------|
| ★★★ **agent** | エージェント | タスクを自律的に完了するシステム。手順が不明・動的な場合に使用 |
| ★★★ **workflow** | ワークフロー | 手順が明確な場合に使う構造化されたパイプライン |
| ★★★ **parallelization workflow** | - | 並列化ワークフロー。独立したサブタスクを同時実行して集約 |
| ★★★ **chaining workflow** | - | チェーニングワークフロー。前ステップの出力を次ステップの入力にする逐次処理 |
| ★★★ **routing workflow** | - | ルーティングワークフロー。入力を分類して専門パイプラインに振り分ける |
| ★★ **environment inspection** | - | エージェントが作業前に環境（ファイル、状態）を調べること |
| ★★ **subagent** | サブエージェント | 親エージェントから呼び出される子エージェント |
| ★ **orchestration** | オーケストレーション | 複数のエージェント/ワークフローを調整・管理すること |

---

## Claude Code

| 英単語 / フレーズ | 読み方・略称 | 日本語訳・説明 |
|-----------------|------------|--------------|
| ★★★ **CLAUDE.md** | クロードエムディー | プロジェクト固有の情報をClaudeに伝えるファイル。`/init`で自動生成 |
| ★★★ **/init** | - | プロジェクトを分析してCLAUDE.mdを生成するコマンド |
| ★★★ **hook** | フック | ツール実行の前後にカスタムコマンドを実行する仕組み |
| ★★★ **PreToolUse** | - | ツール実行**前**のフック。exit 2でツールをブロックできる |
| ★★★ **PostToolUse** | - | ツール実行**後**のフック。フォーマット・テスト実行などに使用 |
| ★★★ **exit code 2** | - | PreToolUseフックでツール呼び出しをブロックするための終了コード |
| ★★★ **Claude Code SDK** | - | `@anthropic-ai/claude-agent-sdk`。Claude Codeをプログラムから呼び出すライブラリ |
| ★★★ **query()** | - | Claude Code SDKのメイン関数。`for await (const msg of query({prompt}))`で使用 |
| ★★ **/compact** | - | 会話履歴を要約圧縮するコマンド |
| ★★ **/clear** | - | 新しい会話を開始するコマンド（履歴は削除しない） |
| ★★ **/resume** | - | 以前の会話を再開するコマンド |
| ★★ **/effort** | - | 推論深度を調整するコマンド（`/effort high`など） |
| ★★ **custom commands** | - | `.claude/commands/*.md`で定義するプロジェクト固有のスラッシュコマンド |
| ★★ **$ARGUMENTS** | - | カスタムコマンドで引数を受け取るプレースホルダー |
| ★★ **allowedTools** | - | 使用を許可するツールを限定するオプション |
| ★★ **/install-github-app** | - | GitHub 統合をセットアップするコマンド |
| ★★ **@claude** | - | GitHub Issues/PRでClaudeを呼び出すメンション |
| ★★ **Stop hook** | - | Claudeの応答完了時に実行されるフック |
| ★★ **UserPromptSubmit hook** | - | ユーザーがプロンプトを送信した直後に実行されるフック |
| ★ **settings.local.json** | - | フックを定義するClause Codeの設定ファイル |
| ★ **$PWD** | - | カレントディレクトリの絶対パス。フック設定で相対パスを避けるために使用 |
| ★ **tsc --noEmit** | - | TypeScriptのコンパイルチェック（出力ファイルを生成しない）。フックで型エラー検出に使用 |

---

## その他・汎用用語

| 英単語 / フレーズ | 読み方・略称 | 日本語訳・説明 |
|-----------------|------------|--------------|
| ★★★ **API key** | - | APIキー。`ANTHROPIC_API_KEY`環境変数で設定。コードにハードコードしてはいけない |
| ★★★ **environment variable** | - | 環境変数。機密情報を安全に管理する手段 |
| ★★★ **.env file** | ドットエンブ | 環境変数を保存するファイル。gitにコミットしてはいけない |
| ★★ **latency** | レイテンシー | 応答遅延。プロンプトキャッシングで改善できる |
| ★★ **context window** | - | モデルが一度に処理できる最大トークン数 |
| ★★ **multimodal** | マルチモーダル | 複数の入力形式（テキスト・画像・PDFなど）を扱える能力 |
| ★★ **sandbox** | サンドボックス | 隔離された安全な実行環境 |
| ★ **async / await** | - | 非同期処理。MCPクライアント実装で多用 |
| ★ **decorator** | デコレータ | Pythonの`@`構文。MCPでツール・リソース・プロンプトを登録するために使用 |
| ★ **JSON** | ジェイソン | JavaScript Object Notation。API通信・ツールスキーマ定義に使用 |
| ★ **schema** | スキーマ | データ構造の定義 |

---

## 間違えやすいペア

| 用語A | 用語B | 違い |
|------|------|------|
| **stateless** (API) | **session** (会話) | APIは状態を持たない。セッション感はアプリが履歴を送ることで実現する |
| **tool (MCP)** | **tool use (API)** | MCPツール=定義済みの機能を提供。ツール使用=Claudeが呼び出す仕組み |
| **workflow** | **agent** | workflow=手順が決まっている。agent=手順をClaudeが自律決定 |
| **PreToolUse** | **PostToolUse** | Pre=前（ブロック可）。Post=後（フィードバック） |
| **temperature=0** | **temperature=1** | 0=確定的・一貫性。1=創造的・多様 |
| **@mcp.tool()** | **@mcp.resource()** | tool=モデルが使う。resource=アプリが取得する |
| **/compact** | **/clear** | compact=圧縮（コンテキスト保持）。clear=完全リセット |
| **exit code 0** | **exit code 2** | 0=許可。2=ブロック（PreToolUseのみ） |
| **budget_tokens** | **max_tokens** | budget_tokens=拡張思考の上限。max_tokens=全体の生成上限 |

---

## よく出るコードパターン

### API リクエストの基本形
```python
client.messages.create(model=..., max_tokens=..., messages=[...])
```

### ツール使用の判定
```python
if response.stop_reason == "tool_use":
```

### プロンプトキャッシングのマーク
```python
{"type": "text", "text": ..., "cache_control": {"type": "ephemeral"}}
```

### フックのブロック
```javascript
process.exit(2);  // ブロック
process.exit(0);  // 許可
```

### Claude Code SDKの使用
```javascript
for await (const message of query({ prompt })) { ... }
```

### MCP ツール定義
```python
@mcp.tool()
def my_tool(param: str = Field(description="...")) -> str:
```

---

*わからない用語が出てきたらここに追記してください。*
