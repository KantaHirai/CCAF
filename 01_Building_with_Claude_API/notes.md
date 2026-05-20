# Building with the Claude API — 試験対策ノート

---

## セクション1: API の基礎

### Claude モデルファミリー

| モデル | 特徴 | 用途 |
|--------|------|------|
| Claude Opus | 最高の知性・推論力 | 複雑なタスク、研究、分析 |
| Claude Sonnet | バランス重視（速度・性能） | 本番アプリ、コーディング |
| Claude Haiku | 最速・最低コスト | 軽量エージェント、頻繁呼び出し |

- モデルIDは `claude-opus-4-7`, `claude-sonnet-4-6`, `claude-haiku-4-5-20251001` など
- 最新モデルを使用するのが原則

---

### API リクエストのライフサイクル（5ステップ）

```
① クライアント → サーバー（あなたのアプリ）
② サーバー → Anthropic API（APIキーで認証）
③ Anthropic でモデル処理（トークナイズ→埋め込み→文脈化→生成）
④ Anthropic API → サーバー（レスポンス返却）
⑤ サーバー → クライアント（生成テキストを表示）
```

**なぜ直接クライアントから呼んではいけないか:**  
APIキーがクライアントコードに露出すると第三者に悪用される → 必ずサーバー経由。

**Claude の内部処理 4 ステップ:**

| ステップ | 内容 |
|--------|------|
| **Tokenization** | 入力テキストをトークン（単語の一部など）に分割 |
| **Embedding** | 各トークンを意味の数値ベクトル（embedding）に変換 |
| **Contextualization** | 周辺語に基づいて各 embedding を文脈に合わせて調整 |
| **Generation** | 次のトークンの確率分布から選んで生成（max_tokens / stop_sequence で終了）|

**生成終了条件:** max_tokens 到達 / end-of-sequence トークン生成 / stop_sequence に到達

---

### API へのアクセスと認証

```python
# パッケージインストール
%pip install anthropic python-dotenv

# .env ファイルに API キーを保存（コードに直接書かない）
ANTHROPIC_API_KEY="sk-ant-..."

# Python での初期化
import anthropic
from dotenv import load_dotenv
load_dotenv()

client = anthropic.Anthropic()  # 環境変数から自動読み込み
```

**API キーの取得手順（console.anthropic.com）:**
1. console.anthropic.com にアクセス
2. ダッシュボード右上の「Get API Keys」をクリック
3. 「Create Key」をクリック
4. ワークスペース・名前を設定して作成
5. キーを安全な場所に保存（一度しか表示されない）

---

### リクエストの作成

```python
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "量子コンピュータを1文で説明してください"}
    ]
)

# レスポンスのテキスト取得
print(message.content[0].text)
```

**重要なパラメータ:**
- `model`: 使用するモデルID
- `max_tokens`: 生成する最大トークン数（必須）
- `messages`: 会話履歴の配列

**レスポンスに含まれるもの:** 生成テキスト（content）、トークン使用量（usage）、終了理由（stop_reason）

---

### マルチターン会話

**ポイント: Claude API はステートレス**

- Claude はリクエスト間で会話履歴を保持しない
- 毎回のリクエストに **完全な会話履歴** を送る必要がある

```python
messages = []

def add_user_message(messages, text):
    messages.append({"role": "user", "content": text})
    return messages

def add_assistant_message(messages, text):
    messages.append({"role": "assistant", "content": text})
    return messages

def chat(messages):
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        messages=messages
    )
    return response.content[0].text

# 会話フロー
messages = add_user_message(messages, "こんにちは！")
reply = chat(messages)
messages = add_assistant_message(messages, reply)
messages = add_user_message(messages, "続けて教えてください")
reply2 = chat(messages)
```

---

### システムプロンプト

- Claude の **役割・行動指針** を定義する文字列
- `create()` の `system` パラメータに渡す

```python
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system="あなたは数学の家庭教師です。答えを直接教えず、ヒントを与えてください。",
    messages=[{"role": "user", "content": "2次方程式の解き方は？"}]
)
```

**システムプロンプトの用途:**
- キャラクター・ペルソナの設定
- 回答フォーマットの指定
- 行動制約の設定（「簡潔に答える」「日本語のみ」など）

---

### 温度（Temperature）

Claude のテキスト生成はトークンの確率分布からサンプリングする仕組み。

| 温度 | 挙動 | 用途 |
|------|------|------|
| 0.0 | 確定的・一貫性が高い | 事実確認、コード生成 |
| 0.5〜0.7 | バランス | 一般的な質問応答 |
| 1.0 | 創造的・多様 | ブレインストーミング、創作 |

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    temperature=0.0,   # 低温度 → 一貫した回答
    messages=[{"role": "user", "content": "映画のアイデアを教えて"}]
)
```

- 温度が高くても同じ回答が出ることはある（確率を変えるだけ）
- 拡張思考（Extended Thinking）とは**互換性なし**

---

### レスポンスストリーミング

大きなレスポンスを**チャンクごとにリアルタイム送信**することでUXを改善。

```python
with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=1000,
    messages=messages
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)  # チャンクをリアルタイム送信
    
    # 完全なメッセージオブジェクトを取得（DB保存用など）
    final_message = stream.get_final_message()
```

- 応答が10〜30秒かかる場合にUXが大幅に改善
- `stream.text_stream`: テキストチャンクのイテレータ
- `stream.get_final_message()`: 完全なレスポンスオブジェクト

---

### 構造化データの出力

Claude にJSONやコードなどを**説明文なしで**出力させる技法。

**テクニック: アシスタントメッセージの事前入力（Prefilling）+ ストップシーケンス**

```python
# JSON出力の例
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "ユーザー情報をJSONで生成してください"},
        {"role": "assistant", "content": "{"}  # アシスタントメッセージを事前入力
    ],
    stop_sequences=["}"]  # JSONの終わりで停止
)

# クリーンなJSONを取得
import json
clean_json = json.loads(("{" + response.content[0].text + "}").strip())
```

**応用:**
- Pythonコードなら ` ```python` で開始・終了
- 箇条書きリスト、CSV など構造化コンテンツ全般に使える

---

## セクション2: プロンプト評価（Evals）

### プロンプト評価とは

プロンプトを**数値的に測定**してシステマチックに改善するアプローチ。

- **プロンプトエンジニアリング**: 良いプロンプトを書く技術
- **プロンプト評価**: プロンプトの品質を客観的に測定する技術

### 典型的な評価ワークフロー（5ステップ）

1. **プロンプトを下書き**（初期バージョンを作成）
2. **テストデータセットを作成**（代表的な入力例を用意）
3. **Claude をデータセットで実行**（全テストケースで出力生成）
4. **出力を採点**（グレーダーで評価）
5. **スコアで比較**（プロンプトバージョン間の優劣を数値で判断）

### テストデータセットの生成

Claude 自身でテストケースを生成できる。`generate_dataset()` はデータセット仕様を渡すと Claude がテストケースを自動生成し、prefilling + stop_sequences で JSON をパースして返す。

```python
dataset = evaluator.generate_dataset(
    task_description="Write a compact 1 day meal plan for an athlete",
    prompt_inputs_spec={
        "height": "Athlete's height in cm",
        "weight": "Athlete's weight in kg",
        "goal": "Goal of the athlete",
        "restrictions": "Dietary restrictions"
    },
    output_file="dataset.json",
    num_cases=3  # 開発中は少数に留める
)
```

### 評価の実行

```python
results = evaluator.run_evaluation(
    run_prompt_function=run_prompt,
    dataset_file="dataset.json",
    extra_criteria="""
    The output should include:
    - Daily caloric total
    - Macronutrient breakdown
    - Meals with exact foods, portions, and timing
    """
)
```

### グレーダーの種類

| グレーダー | 説明 | 適用例 |
|-----------|------|--------|
| **コードベースグレーダー** | Pythonで文法・形式を検証 | JSON/Python/Regex の構文チェック |
| **モデルベースグレーダー** | Claude が 1〜10 で採点 | 主観的品質評価（完全性・役立ち度） |
| **人間グレーダー** | 人が手動でレビュー | 総合品質・深さ・簡潔さ（最も柔軟だが時間かかる）|

---

### モデルベースグレーダーの実装

スコアだけを聞くと中間値（6点前後）に偏る → **強み・弱み・推論** も一緒に求めると精度が上がる。

```python
def grade_by_model(test_case, output):
    eval_prompt = """
You are an expert code reviewer. Evaluate this AI-generated solution.
Task: {task}
Solution: {solution}
Provide your evaluation as a structured JSON object with:
- "strengths": An array of 1-3 key strengths
- "weaknesses": An array of 1-3 key areas for improvement
- "reasoning": A concise explanation of your assessment
- "score": A number between 1-10
"""
    messages = []
    add_user_message(messages, eval_prompt)
    add_assistant_message(messages, "```json")   # prefilling
    eval_text = chat(messages, stop_sequences=["```"])
    return json.loads(eval_text)
```

---

### コードベースグレーダーの実装

```python
def validate_json(text):
    try:
        json.loads(text.strip())
        return 10
    except json.JSONDecodeError:
        return 0

def validate_python(text):
    try:
        ast.parse(text.strip())
        return 10
    except SyntaxError:
        return 0

def validate_regex(text):
    try:
        re.compile(text.strip())
        return 10
    except re.error:
        return 0

# モデルスコアとコードスコアを平均
score = (model_score + syntax_score) / 2
```

テストケースには期待する出力形式（`"format": "python"` など）を含め、グレーダーが正しいバリデーター関数を選べるようにする。

コード出力をクリーンにするためのプレフィリング例:
```python
add_assistant_message(messages, "```code")  # Python/JSON/Regex を問わず使える
```

---

## セクション3: プロンプトエンジニアリング技法

### プロンプトエンジニアリングの反復サイクル

1. ゴールを設定
2. 初期プロンプトを作成
3. 評価（スコア確認）
4. プロンプトエンジニアリング技法を適用
5. 再評価して改善を確認
→ 3〜5 を繰り返す

最初のスコアが 2〜3 / 10 でも正常。改善を追跡することが目的。

---

### 1. 明確・直接的に書く

- プロンプトの**最初の行が最重要**
- 曖昧な表現を避け、シンプルな言葉でタスクを明示
- 悪い例: 「例の件について色々と...」
- 良い例: 「以下の製品説明を50文字以内で要約してください」

---

### 2. 具体的に書く（2種類のガイドライン）

**① 出力品質ガイドライン** — 出力が持つべき性質をリスト:
- 長さ・フォーマット・含むべき要素・トーン

```
Guidelines:
1. Include accurate daily calorie amount
2. Show protein, fat, and carb amounts
3. Specify when to eat each meal
4. Use only foods that fit restrictions
5. List all portion sizes in grams
```
→ スコアが 3.92 → 7.86 に向上した事例あり（品質ガイドライン追加のみで 2倍以上改善）。

**② プロセスステップ** — 問題をどう考えるかの手順を指示:
- 複雑な問題・意思決定・批判的思考に有効
- Claude が複数の観点を検討してから回答するよう誘導

```
1. Brainstorm three talents that would create dramatic tension
2. Pick the most interesting talent
3. Outline a pivotal scene that reveals the talent
```

---

### 3. XML タグで構造化

大量コンテンツを含むプロンプトに特に有効。

```xml
<instructions>
以下の売上記録を分析して、月別トップ3商品を特定してください。
</instructions>

<sales_records>
{sales_data}
</sales_records>

<output_format>
JSON形式でresult配列として返してください。
</output_format>
```

**利点:**
- Claude が指示とデータを明確に区別できる
- 複数のデータソースを扱うときに特に有効

---

### 4. 例を提供する（Few-Shot Prompting）

**ゼロショット**: 例なしで直接指示  
**ワンショット**: 例を1つ提供  
**マルチショット**: 複数の例を提供（最も効果的）

```python
prompt = """
以下の例を参考に感情分析を行ってください:

入力: "今日の映画は最高だった！"
出力: positive

入力: "最悪な一日だった..."
出力: negative

入力: "まあまあかな"
出力: neutral

入力: "{user_tweet}"
出力:"""
```

- **皮肉や文脈依存**の判断に特に効果的
- 例はエッジケースをカバーするよう選ぶ

---

## セクション4: ツール使用（Tool Use）

### ツール使用とは

Claude のトレーニングデータに含まれない**最新情報や外部システム**にアクセスさせる仕組み。

**なぜ必要か:** Claude は訓練時のデータしか知らず、リアルタイムデータ（天気、株価、DB など）にアクセスできない。

**ツール使用の基本フロー:**
1. ツールスキーマ（JSON Schema）を Claude に渡す
2. Claude がツール呼び出しを要求
3. アプリケーションがツールを実行
4. 結果を Claude に返す
5. Claude が最終回答を生成

---

### プロジェクト例: リマインダーシステム

Claude の3つの限界をツールで補う:

| 問題 | 解決ツール |
|------|-----------|
| 正確な現在時刻を知らない | `get_current_datetime` |
| 日付計算が不正確 | `add_duration_to_datetime` |
| リマインダー設定機能がない | `set_reminder` |

→ ツールは「Claude にできないことをツールで補う」という原則の実例。

---

### ツールスキーマ（JSON Schema）

```python
get_weather_schema = {
    "name": "get_weather",
    "description": "指定した都市の現在の天気情報を取得します",
    "input_schema": {
        "type": "object",
        "properties": {
            "city": {
                "type": "string",
                "description": "天気を取得する都市名（例: Tokyo）"
            },
            "unit": {
                "type": "string",
                "enum": ["celsius", "fahrenheit"],
                "description": "温度の単位"
            }
        },
        "required": ["city"]
    }
}
```

**JSON Schemaの3要素:**
- `name`: ツールの名前（関数名）
- `description`: 何をするツールか（Claude がいつ使うか判断するための説明）
- `input_schema`: パラメータの型・説明・必須項目

---

### ツール関数のベストプラクティス

```python
def get_current_datetime(date_format="%Y-%m-%d %H:%M:%S"):
    if not date_format:
        raise ValueError("date_format cannot be empty")
    return datetime.now().strftime(date_format)
```

- **わかりやすい名前**: 関数名・パラメータ名で目的を明示
- **入力のバリデーション**: 必須パラメータが空でないか確認
- **明確なエラーメッセージ**: Claude はエラーを見て修正した引数で再試行できる

---

### ツール呼び出しのリクエスト

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=[get_weather_schema],
    messages=[{"role": "user", "content": "東京の天気は？"}]
)
```

---

### レスポンスのメッセージブロック

ツール使用が有効な場合、レスポンスは**複数のブロック**を持つ:

```python
# レスポンスの構造例
response.content = [
    TextBlock(type="text", text="東京の天気を確認します"),
    ToolUseBlock(
        type="tool_use",
        id="toolu_01...",
        name="get_weather",
        input={"city": "Tokyo", "unit": "celsius"}
    )
]

# stop_reason が "tool_use" の場合はツール呼び出しが必要
if response.stop_reason == "tool_use":
    tool_use = next(b for b in response.content if b.type == "tool_use")
    # ツールを実行
    result = get_weather(**tool_use.input)
```

---

### ツール結果の送信

```python
# ツール結果を会話に追加
messages.append({
    "role": "user",
    "content": [{
        "type": "tool_result",
        "tool_use_id": tool_use.id,  # ツール呼び出しIDと一致させる
        "content": "東京: 25°C、晴れ",
        "is_error": False  # エラーの場合は True
    }]
})

# 再度リクエスト（ツールスキーマも必ず含める）
final_response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=[get_weather_schema],  # ← 必ず再度含める
    messages=messages
)
```

**重要:** フォローアップリクエストでも `tools` パラメータを含めること。

---

### マルチターン・マルチツール会話ループ

複数ツールを連続で呼ぶ場合のループパターン:

```python
from anthropic.types import Message

def add_user_message(messages, message):
    user_message = {
        "role": "user",
        "content": message.content if isinstance(message, Message) else message
    }
    messages.append(user_message)

def text_from_message(message):
    return "\n".join([block.text for block in message.content if block.type == "text"])

def run_conversation(messages):
    while True:
        response = chat(messages, tools=[...])
        add_assistant_message(messages, response)
        if response.stop_reason != "tool_use":
            break
        tool_result_blocks = run_tools(response)
        add_user_message(messages, tool_result_blocks)
    return messages
```

複数ツールを追加する際のルーター:
```python
def run_tool(tool_name, tool_input):
    if tool_name == "get_current_datetime":
        return get_current_datetime(**tool_input)
    elif tool_name == "add_duration_to_datetime":
        return add_duration_to_datetime(**tool_input)
    elif tool_name == "set_reminder":
        return set_reminder(**tool_input)
```

---

### 細粒度のツール制御（Fine-Grained Tool Calling）

ストリーミング時のツール引数 JSON イベント: `InputJsonEvent`
- `partial_json`: 生成中の JSON チャンク
- `snapshot`: 累積された JSON 全体

通常: Anthropic API はトップレベルの key-value ペアが完成するまでバッファリングしてから送信（JSON バリデーション付き）。

**fine_grained=True** を有効にするとバリデーションをスキップし、生成されるたびにチャンクを即時送信:
```python
run_conversation(messages, tools=[save_article_schema], fine_grained=True)
```

注意: `fine_grained=True` 時は不正 JSON が来る可能性があるため、エラーハンドリングが必要:
```python
try:
    parsed_args = json.loads(chunk.snapshot)
except json.JSONDecodeError:
    print("Received invalid JSON, continuing...")
```

---

### 組み込みツール① テキスト編集ツール（Text Editor Tool）

Claude はファイル操作の JSON スキーマを**自分で知っている**（実装は開発者が提供）。

**できること:** ファイル/ディレクトリの表示・行範囲表示・テキスト置換・新規ファイル作成・行挿入・変更の undo

**スキーマスタブ（モデルによって異なる）:**
```python
def get_text_edit_schema(model):
    if model.startswith("claude-3-7-sonnet"):
        return {"type": "text_editor_20250124", "name": "str_replace_editor"}
    elif model.startswith("claude-3-5-sonnet"):
        return {"type": "text_editor_20241022", "name": "str_replace_editor"}
```

→ スタブを渡すと Claude が内部でフルスキーマに展開する。実装（ファイル読み書き等）は開発者が担当。

---

### 組み込みツール② Web 検索ツール（Web Search Tool）

```python
web_search_schema = {
    "type": "web_search_20250305",
    "name": "web_search",
    "max_uses": 5,           # 1リクエスト中の最大検索回数
    "allowed_domains": ["nih.gov"]  # 検索ドメインを制限（任意）
}
```

**事前設定が必要:** console.anthropic.com の Settings > Privacy で Web Search を有効にする。

**レスポンスのブロック構造:**
- `TextBlock`: Claude の説明テキスト
- `ServerToolUseBlock`: 実際の検索クエリ
- `WebSearchToolResultBlock`: 検索結果まとめ
- `WebSearchResultBlock`: 個別の検索結果（タイトル・URL）
- `CitationBlock`: Claude が引用した具体的なテキスト

---

### tool_choice オプション

- `{"type": "auto"}`: 自動選択（デフォルト）
- `{"type": "any"}`: 少なくとも1つのツールを必ず使う
- `{"type": "tool", "name": "..."}`: 指定ツールを強制使用
- `{"type": "none"}`: ツールを使わない

---

## セクション5: RAG（Retrieval Augmented Generation）

### RAGとは

大きなドキュメントを**チャンクに分割**し、質問に関連する部分だけを取得してClaudeに渡す技術。

**なぜ必要か:**
- 800ページの文書を全部プロンプトに入れると → 高コスト・遅い・コンテキスト上限超過
- RAGで関連部分だけを渡す → 効率的・安価・スケーラブル

### RAGの基本フロー（6ステップ）

```
① ドキュメントをチャンクに分割
② 各チャンクを埋め込みベクトルに変換
③ ベクトル DB に保存（前処理完了・待機）
④ ユーザーの質問を埋め込みベクトルに変換
⑤ DB で類似ベクトルを検索（コサイン類似度）
⑥ 関連チャンクをプロンプトに挿入して Claude に渡す
```

---

### テキストチャンキング戦略

| 戦略 | 特徴 | 適用シーン |
|------|------|-----------|
| **固定長（サイズ基準）** | 最もシンプル・どんな文書でも動作 | 本番のフォールバックに最適 |
| **構造基準** | セクション・見出しで分割・最高品質 | フォーマットが保証されている文書（Markdown等）|
| **文単位** | 文末で分割・重複オーバーラップあり | 一般テキスト文書 |
| **意味的** | NLP で文間の関連度を計算 | 高品質が必要・計算コスト許容の場合 |

**コード例（固定長チャンク）:**
```python
def chunk_by_char(text, chunk_size=150, chunk_overlap=20):
    chunks = []
    start_idx = 0
    while start_idx < len(text):
        end_idx = min(start_idx + chunk_size, len(text))
        chunks.append(text[start_idx:end_idx])
        start_idx = end_idx - chunk_overlap if end_idx < len(text) else len(text)
    return chunks
```

**コード例（構造基準・Markdown）:**
```python
def chunk_by_section(document_text):
    pattern = r"\n## "
    return re.split(pattern, document_text)
```

**コード例（文単位）:**
```python
def chunk_by_sentence(text, max_sentences_per_chunk=5, overlap_sentences=1):
    sentences = re.split(r"(?<=[.!?])\s+", text)
    chunks = []
    start_idx = 0
    while start_idx < len(sentences):
        end_idx = min(start_idx + max_sentences_per_chunk, len(sentences))
        chunks.append(" ".join(sentences[start_idx:end_idx]))
        start_idx += max_sentences_per_chunk - overlap_sentences
    return chunks
```

---

### テキスト埋め込み（Embeddings）

- テキストを**数値ベクトル**に変換
- 意味的に類似したテキストは近いベクトル空間に配置される
- 各次元が「何を表すか」は学習済みで人間には解釈できない

**VoyageAI（推奨プロバイダー）:**  
Anthropic 自体は現在 embedding を提供していない → VoyageAI を使う。

```python
import voyageai
client = voyageai.Client()  # VOYAGE_API_KEY を環境変数に設定

def generate_embedding(text, model="voyage-3-large", input_type="query"):
    result = client.embed([text], model=model, input_type=input_type)
    return result.embeddings[0]
```

**コサイン類似度:**
- ベクトル間の角度のコサインで類似度を計算
- 範囲: -1 〜 1（1 に近いほど類似）
- **コサイン距離** = 1 - コサイン類似度（0 に近いほど類似）

---

### RAG 実装コード

```python
# ① チャンク分割
with open("./report.md", "r") as f:
    text = f.read()
chunks = chunk_by_section(text)

# ② 埋め込み生成（バッチ処理）
embeddings = generate_embedding(chunks)

# ③ ベクトル DB に格納（テキストも一緒に保存）
store = VectorIndex()
for embedding, chunk in zip(embeddings, chunks):
    store.add_vector(embedding, {"content": chunk})

# ④ クエリ埋め込み生成
user_embedding = generate_embedding("What did software engineering dept do?")

# ⑤ 類似検索（上位 k 件）
results = store.search(user_embedding, 2)
for doc, distance in results:
    print(distance, doc["content"][:200])
```

---

### BM25 語彙的検索

キーワードの完全一致に強い検索手法。特定の ID や固有名詞の検索で意味検索より優れる。

**BM25 アルゴリズムの仕組み:**
1. クエリをトークン化（例: "INC-2023-Q4-011" → 2つのトークン）
2. 各トークンが文書中に出現する頻度をカウント
3. 出現頻度の低いトークン（レアな用語）を高く重み付け
4. 重み付きスコアが高い文書を返す

```python
store = BM25Index()
for chunk in chunks:
    store.add_document({"content": chunk})
results = store.search("What happened with INC-2023-Q4-011?", 3)
```

---

### マルチインデックス RAG パイプライン（ハイブリッド検索）

意味検索（VectorIndex）+ 語彙検索（BM25Index）を組み合わせる。

**Reciprocal Rank Fusion（RRF）で結果を統合:**
```
RRF_score(d) = Σ(1 / (k + rank_i(d)))
```
- `k`: 定数（一般的には 60）
- 複数インデックスで上位に来る文書が高スコアになる

```python
class Retriever:
    def __init__(self, *indexes: SearchIndex):
        self._indexes = list(indexes)

    def add_document(self, document):
        for index in self._indexes:
            index.add_document(document)

    def search(self, query_text, k=1, k_rrf=60):
        # 各インデックスから結果取得 → RRF で統合
        ...

# 使い方
retriever = Retriever(VectorIndex(), BM25Index())
```

同じ `add_document()` / `search()` インターフェースで実装すれば、新しい検索手法の追加が容易。

---

## セクション6: Claude の機能

### 拡張思考（Extended Thinking）

Claude が**最終回答を出す前に考えを整理する**機能。

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000  # 思考に使えるトークン数の上限
    },
    messages=[{"role": "user", "content": "複雑な数学の問題..."}]
)

# レスポンスに思考ブロックと回答ブロックが含まれる
for block in response.content:
    if block.type == "thinking":
        print("思考:", block.thinking)
    elif block.type == "text":
        print("回答:", block.text)
```

**制限事項:**
- 温度（temperature）と**互換性なし**
- アシスタントメッセージの事前入力と**互換性なし**
- コスト・レイテンシが増加
- まず通常のプロンプトエンジニアリングを最適化し、それでも不十分な場合に使う

---

### 画像サポート

```python
import base64

with open("image.jpg", "rb") as f:
    image_data = base64.standard_b64encode(f.read()).decode("utf-8")

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/jpeg",
                    "data": image_data
                }
            },
            {"type": "text", "text": "この画像を説明してください"}
        ]
    }]
)
```

**画像の制限:**
- 1リクエストで最大 **100枚**
- 1枚あたり最大 **5MB**
- 1枚送信時: 最大 **8000px**（幅・高さ）
- 複数枚送信時: 最大 **2000px**（幅・高さ）
- トークン数: `(幅px × 高さpx) / 750`

**画像でも Prompt Engineering が重要:** 単純な質問より、分析ステップを明示したプロンプトが大幅に精度向上。

---

### PDF サポート

```python
with open("document.pdf", "rb") as f:
    file_bytes = base64.standard_b64encode(f.read()).decode("utf-8")

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "document",    # 画像は "image"、PDF は "document"
                "source": {
                    "type": "base64",
                    "media_type": "application/pdf",  # PDF の media_type
                    "data": file_bytes,
                }
            },
            {"type": "text", "text": "このPDFを1文で要約してください"}
        ]
    }]
)
```

Claude は PDF 内のテキスト・画像・グラフ・表・構造を理解できる。

---

### 引用（Citations）

Claude が回答の根拠となる文書の箇所を**引用付きで返す**機能。

```python
# PDF + Citations
{
    "type": "document",
    "source": {"type": "base64", "media_type": "application/pdf", "data": file_bytes},
    "title": "earth.pdf",
    "citations": {"enabled": True}   # ← これで有効化
}

# プレーンテキスト + Citations
{
    "type": "document",
    "source": {"type": "text", "media_type": "text/plain", "data": article_text},
    "title": "earth_article",
    "citations": {"enabled": True}
}
```

**Citations が返す情報:**
- `cited_text`: Claude が引用した実際のテキスト
- `document_index`: 複数文書がある場合のインデックス
- `document_title`: 文書のタイトル
- `start_page_number` / `end_page_number`: 引用箇所のページ（PDF の場合）
- プレーンテキストの場合: ページ番号の代わりに文字位置

---

### プロンプトキャッシング

同じコンテンツを繰り返し送る場合に**計算を再利用**してコストと速度を改善。

**仕組み:** 最初のリクエストでキャッシュ書き込み → フォローアップで読み込み。TTL は **1時間**。

**キャッシュブレークポイントの追加:**
```python
# テキストブロックの longhand 形式が必要
{
    "type": "text",
    "text": very_long_system_prompt,
    "cache_control": {"type": "ephemeral"}  # ← キャッシュポイント
}
```

**重要ルール:**
- `cache_control` は shorthand 形式（単純文字列）では使えない（longhand 形式が必要）
- キャッシュポイント以前のコンテンツ全体がキャッシュされる
- キャッシュポイントまでの内容が1文字でも変わるとキャッシュ無効
- **最小キャッシュ可能サイズ: 1024 トークン**（それ以下はキャッシュされない）
- 最大 **4つ**のキャッシュブレークポイントを設定可能

**キャッシュ処理順序（内部）:** ツール → システムプロンプト → メッセージ

**ツールスキーマのキャッシュ実装:**
```python
if tools:
    tools_clone = tools.copy()
    last_tool = tools_clone[-1].copy()
    last_tool["cache_control"] = {"type": "ephemeral"}
    tools_clone[-1] = last_tool
    params["tools"] = tools_clone
```

**レスポンスのキャッシュ確認:**
```
初回リクエスト:   cache_creation_input_tokens=1772
フォローアップ: cache_read_input_tokens=1772
```

**効果的な対象:** ツールスキーマ（変化しない）、長いシステムプロンプト、反復文書分析

---

### コード実行と Files API

**Files API:**
- ファイルを事前にアップロードし、`file_id` で参照
- 同じファイルを複数リクエストで再送しなくてよい
- `file_metadata = upload('streaming.csv')` → `file_metadata.id` を使う

**コード実行ツール:**
```python
tools=[{"type": "code_execution_20250522", "name": "code_execution"}]
```
- 隔離された Docker コンテナで Python コードを実行
- **ネットワークアクセスなし**（外部 API 呼び出し不可）
- Claude が複数回コードを実行して反復的に分析可能
- 生成されたファイル（グラフ等）は Files API でダウンロード可能

**Files API + コード実行の連携:**
```python
add_user_message(messages, [
    {"type": "text", "text": "Analyze this CSV and generate a churn analysis plot"},
    {"type": "container_upload", "file_id": file_metadata.id},  # ← ファイルをコンテナに渡す
])
```

**レスポンスブロック:**
- `TextBlock`: Claude の説明
- `server_tool_use` ブロック: Claude が実行したコード
- `code_execution_tool_result` ブロック: 実行結果
- `code_execution_output` ブロック: 生成されたファイルの ID

---

## セクション7: Model Context Protocol（APIコース内）

詳細は `02_Introduction_to_MCP/notes.md` も参照。

### MCP とは（APIコースでの説明）

MCP はツール定義と実装の**負担を外部の MCP サーバーに移す**仕組み。

**MCP が解決する問題:**  
GitHub チャットボットを作る場合、リポジトリ・PR・イシュー・プロジェクト…と膨大なツールをすべて自前で実装する必要がある。  
→ MCP なら、誰かが作った GitHub MCP サーバーに接続するだけ。

**よくある誤解: MCP ≠ ツール使用**
- MCP: ツール定義・実装を提供（サーバー側）
- ツール使用: Claude がツールを呼び出す仕組み（クライアント側）
- → **補完的な技術**。MCP はツール使用の土台を提供する。

---

### MCP クライアントの通信フロー

```
① ユーザーがクエリを送信
② サーバーが MCP クライアントに ListToolsRequest を送る
③ MCP サーバーが ListToolsResult を返す
④ Claude にツール一覧 + クエリを送信
⑤ Claude が CallToolRequest を返す
⑥ MCP クライアントが MCP サーバーに CallToolRequest を送る
⑦ MCP サーバーが外部 API（GitHub等）を呼び出し結果を返す
⑧ ツール結果を Claude に返し、最終回答を生成
```

**トランスポートアグノスティック（通信方式を選ばない）:**
- `stdio`（標準入出力）: 同一マシン上の最も一般的な方式
- HTTP / WebSocket: リモート接続にも対応

---

### MCP サーバーの実装（Python SDK）

**初期化:**
```python
from mcp.server.fastmcp import FastMCP
from pydantic import Field

mcp = FastMCP("DocumentMCP", log_level="ERROR")

# ドキュメントをメモリに保存
docs = {
    "deposition.md": "...",
    "report.pdf": "...",
}
```

**ツール定義（`@mcp.tool`）:**
```python
@mcp.tool(
    name="read_doc_contents",
    description="Read the contents of a document and return it as a string."
)
def read_document(
    doc_id: str = Field(description="Id of the document to read")
):
    if doc_id not in docs:
        raise ValueError(f"Doc with id {doc_id} not found")
    return docs[doc_id]

@mcp.tool(
    name="edit_document",
    description="Edit a document by replacing a string in the documents content."
)
def edit_document(
    doc_id: str = Field(description="Id of the document that will be edited"),
    old_str: str = Field(description="The text to replace. Must match exactly."),
    new_str: str = Field(description="The new text to insert.")
):
    if doc_id not in docs:
        raise ValueError(f"Doc with id {doc_id} not found")
    docs[doc_id] = docs[doc_id].replace(old_str, new_str)
```

**SDK の利点:** 型ヒントから JSON スキーマを自動生成、`Field` の description が Claude への説明になる。

---

### リソース定義（`@mcp.resource`）

**直接リソース（固定 URI）:**
```python
@mcp.resource("docs://documents", mime_type="application/json")
def list_docs() -> list[str]:
    return list(docs.keys())
```

**テンプレートリソース（パラメータあり）:**
```python
@mcp.resource("docs://documents/{doc_id}", mime_type="text/plain")
def fetch_doc(doc_id: str) -> str:
    if doc_id not in docs:
        raise ValueError(f"Doc with id {doc_id} not found")
    return docs[doc_id]
```

**Resources はツール使用を介さずコンテキストに直接注入できる**（`@document名` メンション機能に活用）。

---

### プロンプト定義（`@mcp.prompt`）

```python
from mcp.server.fastmcp import base

@mcp.prompt(
    name="format",
    description="Rewrites the contents of the document in Markdown format."
)
def format_document(
    doc_id: str = Field(description="Id of the document to format")
) -> list[base.Message]:
    prompt = f"""
Your goal is to reformat a document to be written with markdown syntax.
The id of the document you need to reformat is: {doc_id}
Use the 'edit_document' tool to edit the document.
"""
    return [base.UserMessage(prompt)]
```

プロンプトはサーバー作者が**テスト・評価済みの高品質な指示**を提供できる。ユーザーが `/format report.pdf` のようなスラッシュコマンドとして使える。

---

### MCP クライアントの実装

```python
from mcp import ClientSession
from mcp.client.stdio import stdio_client
from pydantic import AnyUrl
import json

class MCPClient:
    def __init__(self):
        self._session = None

    def session(self):
        return self._session

    async def list_tools(self) -> list:
        result = await self.session().list_tools()
        return result.tools

    async def call_tool(self, tool_name: str, tool_input: dict):
        return await self.session().call_tool(tool_name, tool_input)

    async def list_resources(self) -> list:
        result = await self.session().list_resources()
        return result.resources

    async def read_resource(self, uri: str):
        result = await self.session().read_resource(AnyUrl(uri))
        resource = result.contents[0]
        if isinstance(resource, types.TextResourceContents):
            if resource.mimeType == "application/json":
                return json.loads(resource.text)
            return resource.text

    async def list_prompts(self) -> list:
        result = await self.session().list_prompts()
        return result.prompts

    async def get_prompt(self, prompt_name: str, args: dict[str, str]):
        result = await self.session().get_prompt(prompt_name, args)
        return result.messages
```

クライアント接続例:
```python
async with MCPClient(command="uv", args=["run", "mcp_server.py"]) as client:
    tools = await client.list_tools()
```

---

### サーバーインスペクター

開発・デバッグ用のブラウザ UI:
```bash
mcp dev mcp_server.py   # ポート 6277 でブラウザ UI が起動
```

使い方: Connect → Tools タブ → List Tools → ツール選択 → パラメータ入力 → Run Tool

**開発ループ:** コード変更 → インスペクターでテスト → アプリに接続せず単体で検証

---

### MCP の3プリミティブまとめ

| プリミティブ | 制御者 | 定義方法 | 典型的な用途 |
|------------|--------|---------|------------|
| **Tools** | モデル（Claude） | `@mcp.tool()` | ファイル読み書き・API呼び出し |
| **Resources** | アプリケーション | `@mcp.resource()` | @メンション・コンテキスト注入 |
| **Prompts** | ユーザー | `@mcp.prompt()` | `/format` などのスラッシュコマンド |

**Claude Code + MCP 連携:**
```bash
claude mcp add [server-name] [command-to-start-server]
claude mcp add documents uv run main.py
```

人気の MCP インテグレーション: `sentry-mcp`（バグ発見・修正）、`playwright-mcp`（ブラウザ自動化）、`figma-context-mcp`（デザイン参照）、`slack-mcp`（チーム通知）

---

## セクション8: Claude Code & Anthropic アプリ

→ Claude Code専用コースのノートを参照（`03_Claude_Code_in_Action/notes.md`）

**Anthropic アプリの概要:**
- **Claude.ai**: Webインターフェース
- **Claude Code**: ターミナルベースのコーディングアシスタント（開発者向け）
- **Computer Use**: デスクトップ操作・ブラウザ操作ができるツールセット
  - Web サイト閲覧・デスクトップアプリ操作・視覚的 UI ナビゲーション

これらは**エージェントの実例**として有用:  
ツール統合・マルチステップ実行・環境との相互作用・自律的問題解決を実装している。

---

## セクション9: エージェントとワークフロー

### ワークフロー vs エージェント（詳細比較）

| | ワークフロー | エージェント |
|--|-------------|-------------|
| **定義** | 手順が事前にわかっている場合の定義済みフロー | ゴールとツールを与えて Claude に計画させる |
| **メリット** | 高精度・テスト容易・予測可能 | 柔軟・未知タスク対応・ユーザー対話的 |
| **デメリット** | 特定タスクに限定・入力を事前に決める必要あり | タスク完了率が低い・評価・テストが難しい |
| **使う場面** | 手順が明確な定型タスク | 不規則・多様なユーザーリクエスト |

**原則: まずワークフローを実装し、必要な場合のみエージェントを使う**  
ユーザーが求めるのは信頼性のある動作であり、アーキテクチャの種類ではない。

---

### 並列化ワークフロー

複数の独立したサブタスクを**同時に実行**してから集約するパターン。

```python
import asyncio

async def analyze_aspect(aspect, content):
    response = await client.messages.create(...)
    return response

# 並列実行
results = await asyncio.gather(
    analyze_aspect("強度", material_data),
    analyze_aspect("コスト", material_data),
    analyze_aspect("環境負荷", material_data)
)

# 結果を集約
final_decision = aggregate(results)
```

**利点:**
- 各側面に集中した専用プロンプトで**精度向上**
- 各プロンプトを**独立してテスト・改善**できる
- 新しい側面の追加が**容易**

---

### チェーニングワークフロー

複数のサブタスクを**順次実行**し、前のステップの出力を次のステップに渡すパターン。

```python
# ステップ1: 初稿生成
draft = generate_article(topic)

# ステップ2: 品質改善（AIっぽい表現を除去、絵文字除去など）
final = revise_article(draft, """
以下を修正してください:
1. AIが書いたと分かる表現を削除
2. 絵文字を全て削除
3. 陳腐な表現を技術ライター風に修正
""")
```

---

### ルーティングワークフロー

入力を**分類**し、専門化されたパイプラインに振り分けるパターン。

```python
# Step 1: 入力を分類
category = client.messages.create(
    ...
    messages=[{"role": "user", "content": f"分類してください: {user_input}"}]
)

# Step 2: カテゴリに応じたプロンプトで処理
if category == "技術系":
    response = technical_pipeline(user_input)
elif category == "エンタメ系":
    response = entertainment_pipeline(user_input)
```

---

### エージェントとツール

エージェントはツールを使って**自律的にタスクを完了**する:

```python
# エージェントループ
while response.stop_reason == "tool_use":
    # ツールを実行
    tool_result = execute_tool(response.tool_use)
    # 結果を返して続きを生成
    response = client.messages.create(
        tools=tools,
        messages=[...tool_result...]
    )
# stop_reason == "end_turn" で完了
```

**ツールは抽象的に設計する:**  
Claude Code の例: `bash`, `read`, `write`, `edit`, `glob`, `grep` という**汎用ツール**を持つ。  
「コードをリファクタリングする」「依存関係をインストールする」などの特化ツールは持たない。  
→ 汎用ツールを組み合わせることで無数のシナリオに対応できる。

---

### 環境インスペクション（Environment Inspection）

エージェントが**アクションの結果を観察・確認**できる仕組みを設計することが重要。

**なぜ重要か:**  
Claude はボタンをクリックした結果（新しいページに移動したか、メニューが開いたか等）を見ないと次の判断ができない。

**実践パターン:**
- **ファイル操作**: 変更前に必ず現在の内容を読む（Read before Write）
- **UI 操作**: アクション後にスクリーンショットを取る
- **API 呼び出し**: レスポンスに期待データが含まれるか確認
- **生成物**: whisper.cpp で字幕生成・FFmpeg でスクリーンショット抽出して品質確認

**システムプロンプトで環境インスペクションを指示:**
```
Use bash to run whisper.cpp and verify dialogue placement.
Use FFmpeg to extract screenshots at regular intervals to visually inspect the output.
```

これにより Claude がエラーを検出・修正し、品質を自律的に保証できる。

---

## 重要まとめ

| トピック | キーポイント |
|---------|------------|
| API基礎 | ステートレス、毎回完全な会話履歴を送る |
| リクエスト処理 | Tokenize → Embed → Contextualize → Generate |
| システムプロンプト | `system` パラメータで役割・制約を定義 |
| 温度 | 0=確定的, 1=創造的; 拡張思考と非互換 |
| ストリーミング | `client.messages.stream()` でリアルタイム送信 |
| 構造化データ | アシスタント事前入力 + ストップシーケンス |
| Evals | 5ステップサイクル; モデル採点=強み・弱み・推論を要求 |
| コードグレーダー | validate_json/python/regex; スコア = (model + syntax) / 2 |
| ツール使用 | スキーマ→Claude要求→実行→結果返送→最終回答 |
| マルチツールループ | stop_reason == "tool_use" の間ループ継続 |
| Fine-grained tools | fine_grained=True でバリデーションスキップ・即時ストリーミング |
| テキスト編集ツール | スキーマスタブのみ提供・実装は開発者担当 |
| Web検索ツール | max_uses・allowed_domains 設定可能 |
| プロンプトキャッシュ | TTL=1時間、最小1024トークン、最大4ブレークポイント |
| 拡張思考 | budget_tokens, 温度・事前入力と非互換 |
| 画像 | 最大100枚/req, 5MB/枚; tokens=(w×h)/750 |
| PDF | type="document", media_type="application/pdf" |
| Citations | cited_text / document_index / page numbers |
| Files API | 事前アップロード → file_id 参照; container_upload でコード実行に渡す |
| RAG | チャンク分割→埋め込み→類似検索→渡す |
| BM25 | レア用語を高重み付け・完全一致に強い |
| Hybrid RAG | Vector + BM25 → Reciprocal Rank Fusion で統合 |
| ワークフロー | 並列化・チェーニング・ルーティングの3パターン |
| エージェント | 汎用ツールを自律的に組み合わせる・環境インスペクション必須 |
| ワークフロー優先 | 可能な限りワークフロー; エージェントは柔軟だが信頼性低い |
