# Introduction to Model Context Protocol — 試験対策ノート

---

## MCP とは何か

**Model Context Protocol（MCP）** は、Claude にコンテキストとツールを提供するための通信レイヤー。

**目的:** ツール定義と実行の負担を、自分のサーバーから**専門化された MCPサーバー**に移す。

### MCP が解決する問題

例: GitHub のデータにアクセスするチャットインターフェースを構築する場合
- **MCP なし**: リポジトリ、PR、イシュー、プロジェクトなど GitHub の膨大な機能全てに対してツールスキーマと関数を手動で書く必要がある
- **MCP あり**: 誰かが既に実装した GitHub MCP サーバーを接続するだけ

### よくある誤解: MCP ≠ ツール使用

| | MCP | ツール使用（Tool Use） |
|--|-----|----------------------|
| 役割 | ツール定義と実装を提供 | Claude がツールを呼び出す仕組み |
| 誰が実装するか | MCP サーバー開発者 | アプリケーション開発者 |
| 目的 | 統合の再利用 | Claude の能力拡張 |

→ **補完的な技術**。MCP はツールを提供し、ツール使用は Claude がそれを呼び出す。

---

## MCP のアーキテクチャ

```
ユーザー
  ↓
あなたのアプリ（MCP クライアントを含む）
  ↓
MCP クライアント ← → MCP サーバー（ツール/リソース/プロンプトを提供）
                          ↓
                     外部サービス（GitHub, DB など）
```

### コンポーネント

- **MCP クライアント**: MCPサーバーと接続するコード（あなたのアプリ内）
- **MCP サーバー**: ツール・リソース・プロンプトを定義して提供する専門サービス
- **Claude**: クライアントから得たツールリストを使って問題を解決する

---

## トランスポートとメッセージ

### トランスポートアグノスティック

MCP はさまざまな通信プロトコルで動作する:

| トランスポート | 説明 | 主な用途 |
|--------------|------|---------|
| **stdio** | 標準入出力 | ローカルサーバー（最一般的） |
| HTTP | HTTPプロトコル | リモートサーバー |
| WebSocket | 双方向通信 | リアルタイム |

最も一般的なのは、**クライアントとサーバーが同じマシン**で `stdio` 通信。

### MCP メッセージタイプ

| メッセージ | 説明 |
|-----------|------|
| `ListToolsRequest / ListToolsResult` | クライアント→サーバー: 利用可能なツールは？ |
| `CallToolRequest / CallToolResult` | クライアント→サーバー: このツールを実行して |
| `ListResourcesRequest` | リソース一覧の取得 |
| `ReadResourceRequest / ReadResourceResult` | リソース内容の取得 |
| `ListPromptsRequest` | プロンプト一覧の取得 |
| `GetPromptRequest / GetPromptResult` | プロンプト内容の取得 |

---

## MCP の3つのプリミティブ

| プリミティブ | 制御者 | 説明 | 用途例 |
|-------------|--------|------|--------|
| **Tools（ツール）** | モデル（Claude） | Claudeが自律的に呼び出す | ファイル読み込み、API呼び出し |
| **Resources（リソース）** | アプリケーション | アプリコードが取得して利用 | オートコンプリート、コンテキスト注入 |
| **Prompts（プロンプト）** | ユーザー | ユーザーが選択・使用 | `/format`, `/summarize` などのコマンド |

**覚え方:**
- Tools → **Model** controlled（Claude が判断して使う）
- Resources → **App** controlled（アプリが取得してClaudeに渡す）
- Prompts → **User** controlled（ユーザーが明示的に選ぶ）

---

## MCP サーバーの実装

### Python SDK のセットアップ

```python
from mcp.server.fastmcp import FastMCP
from pydantic import Field

# サーバーの初期化
mcp = FastMCP("DocumentMCP", log_level="ERROR")

# ドキュメントストア
docs = {
    "report.pdf": "このレポートは...",
    "plan.md": "プロジェクト計画...",
}
```

### ツールの定義（`@mcp.tool()` デコレータ）

```python
@mcp.tool()
def read_doc_contents(
    doc_id: str = Field(description="読み込むドキュメントのID")
) -> str:
    """ドキュメントの内容を返します"""
    if doc_id not in docs:
        raise ValueError(f"ドキュメント {doc_id} が見つかりません")
    return docs[doc_id]

@mcp.tool()
def edit_document(
    doc_id: str = Field(description="編集するドキュメントのID"),
    old_str: str = Field(description="置換前のテキスト（空白含め完全一致）"),
    new_str: str = Field(description="置換後のテキスト")
) -> None:
    """ドキュメントの内容を編集します"""
    if doc_id not in docs:
        raise ValueError(f"ドキュメント {doc_id} が見つかりません")
    docs[doc_id] = docs[doc_id].replace(old_str, new_str)
```

**SDK の利点:**
- JSON スキーマを手動で書かなくてよい
- 型ヒントが自動バリデーションになる
- `Field(description=...)` が Claude への説明として使われる
- デコレータでツール登録が自動化される

---

### リソースの定義（`@mcp.resource()` デコレータ）

#### 静的リソース（固定URI）

```python
@mcp.resource("docs://list")
def list_documents() -> str:
    """利用可能なドキュメントの一覧を返します"""
    return "\n".join(docs.keys())
```

#### テンプレートリソース（パラメータあり）

```python
@mcp.resource("docs://{doc_id}")
def get_document(doc_id: str) -> str:
    """指定したドキュメントの内容を返します"""
    if doc_id not in docs:
        raise ValueError(f"ドキュメント {doc_id} が見つかりません")
    return docs[doc_id]
```

**リソースの使い方（クライアント側）:**

```python
async def read_resource(self, uri: str):
    result = await self.session().read_resource(AnyUrl(uri))
    resource = result.contents[0]
    
    if isinstance(resource, types.TextResourceContents):
        if resource.mimeType == "application/json":
            return json.loads(resource.text)
        return resource.text
    # BlobResourceContents の場合はバイナリ処理
```

**リソースの活用例:**
- `@ドキュメント名` でドキュメントを参照するオートコンプリート
- プロンプトへのコンテキスト自動注入

---

### プロンプトの定義（`@mcp.prompt()` デコレータ）

```python
@mcp.prompt()
def format_document(
    doc_id: str = Field(description="フォーマットするドキュメントのID")
) -> list:
    """ドキュメントをMarkdown形式にフォーマットするためのプロンプト"""
    doc_content = docs.get(doc_id, "")
    return [
        {
            "role": "user",
            "content": f"""以下のドキュメントをMarkdown形式にフォーマットしてください。
見出し・リスト・強調を適切に使用してください。

<document>
{doc_content}
</document>"""
        }
    ]
```

**プロンプトを使う理由:**
- ユーザーが自分でプロンプトを書くより**高品質な結果**が得られる
- MCP サーバー作成者がテスト・評価済みのプロンプトを提供できる
- `/format doc_id` のような**スラッシュコマンド**として提供できる

---

## MCP クライアントの実装

### クライアントアーキテクチャ

```python
from mcp import ClientSession
from mcp.client.stdio import stdio_client

class MCPClient:
    def __init__(self):
        self._session = None
    
    def session(self):
        return self._session
    
    async def connect(self, server_script_path: str):
        # サーバーに接続
        ...
    
    async def get_tools(self):
        """利用可能なツールの一覧を取得"""
        result = await self.session().list_tools()
        return result.tools
    
    async def call_tool(self, tool_name: str, args: dict):
        """ツールを呼び出す"""
        result = await self.session().call_tool(tool_name, args)
        return result.content
    
    async def list_resources(self):
        """利用可能なリソースの一覧を取得"""
        result = await self.session().list_resources()
        return result.resources
    
    async def read_resource(self, uri: str):
        """リソースの内容を取得"""
        result = await self.session().read_resource(AnyUrl(uri))
        ...
    
    async def list_prompts(self):
        """利用可能なプロンプトの一覧を取得"""
        result = await self.session().list_prompts()
        return result.prompts
    
    async def get_prompt(self, prompt_name: str, args: dict[str, str]):
        """プロンプトを取得（変数を埋め込む）"""
        result = await self.session().get_prompt(prompt_name, args)
        return result.messages
```

### アプリケーションフロー

```
ユーザーの質問
  ↓
MCPクライアント.get_tools() でツール一覧取得
  ↓
Claude に質問 + ツール一覧を送信
  ↓
Claude がツール呼び出しを要求
  ↓
MCPクライアント.call_tool() でツール実行
  ↓
結果を Claude に返す
  ↓
Claude が最終回答を生成
  ↓
ユーザーに返答
```

---

## MCP サーバーインスペクター

開発・デバッグ用のブラウザベースのUI。

```bash
# サーバーをインスペクターモードで起動
mcp dev mcp_server.py
# → http://127.0.0.1:6274 にアクセス
```

**使い方:**
1. ブラウザで URL を開く
2. **Connect** ボタンをクリックしてサーバーに接続
3. **Tools** タブ: ツールを一覧表示・テスト実行
4. **Resources** タブ: 静的リソースとテンプレートリソースを確認
5. **Prompts** タブ: プロンプトをテスト

---

## 実践的な統合パターン

### @メンション によるコンテキスト注入

```python
# ユーザーが "@report.pdf を要約して" と入力した場合
if user_input.startswith("@"):
    doc_id = user_input[1:].split()[0]  # "report.pdf" を抽出
    doc_content = await client.read_resource(f"docs://{doc_id}")
    # ドキュメント内容をプロンプトに注入
    augmented_prompt = f"<document>{doc_content}</document>\n{user_input}"
```

**用途:**
- オートコンプリート: `list_resources()` でリソース一覧を取得しUI表示
- コンテキスト注入: リソース内容を自動的にプロンプトに追加（Claude がツールを使わなくて済む）

### /コマンド によるプロンプト活用

```python
if user_input.startswith("/"):
    command = user_input[1:].split()[0]   # "format"
    arg = user_input.split()[1]           # "report.pdf"
    
    # プロンプトを取得して会話に使用
    prompt_messages = await client.get_prompt(command, {"doc_id": arg})
    response = await ask_claude(prompt_messages)
```

---

## MCP まとめ

| 概念 | ポイント |
|------|---------|
| MCPの目的 | ツール定義・実装の再利用と標準化 |
| Transport | stdio（ローカル）が最一般的、HTTP/WSもあり |
| Tools | モデル制御。Claude が自律的に呼び出す |
| Resources | アプリ制御。アプリが取得してClaudeに渡す |
| Prompts | ユーザー制御。事前に用意された高品質な指示 |
| `@mcp.tool()` | 型ヒント+Fieldで自動スキーマ生成 |
| `@mcp.resource()` | 静的URI or テンプレートURI |
| `@mcp.prompt()` | メッセージリストを返す |
| インスペクター | `mcp dev server.py` でデバッグUI起動 |
