# n8n-nodes-browserbase-secure

[browserbase/n8n-node](https://github.com/browserbase/n8n-node) のセキュリティ強化フォークです。

## Why fork?

Browserbase の公式 n8n ノードは、ブラウザセッションの**録画**と**ログ記録**がデフォルトで有効になっています。これらのデータは Browserbase のサーバーに**最大 30 日間保存**されます。

社内のバックオフィス業務自動化（経費処理、社内ツール操作など）では、ブラウザ上に従業員の個人情報や機密性の高い業務データが表示されます。それらがスクリーンショットや操作ログとして外部サーバーに蓄積されることは、情報セキュリティ上許容できません。

このフォークでは、該当機能をコードレベルで無効化し、n8n の UI からもパラメータを除去することで、運用者が誤って有効化するリスクをゼロにしています。

## 無効化した機能

| 機能 | 公式デフォルト | 本フォーク | 無効化の理由 |
|------|-------------|-----------|-------------|
| **Session Recording** (`recordSession`) | `true` | `false` (ハードコード) | ブラウザ画面の録画が Browserbase サーバーに 30 日間保存される。社内システムの画面情報が外部に残るリスク。 |
| **Session Logging** (`logSession`) | `true` | `false` (ハードコード) | 操作ログ（クリック座標、入力内容、DOM 操作）が記録・保存される。入力データやページ構造から機密情報が推測可能。 |

いずれも n8n UI 上のパラメータ自体を削除しているため、ユーザーが再有効化することはできません。

## Upstream との差分

変更対象は `nodes/Browserbase/Browserbase.node.ts` の 1 ファイルのみです。

1. `browserSettings` で `recordSession: false` / `logSession: false` をハードコード
2. Browser Options の UI パラメータ定義から `Record Session` / `Log Session` を削除
3. 対応する TypeScript 型定義を削除

パッケージ名を `n8n-nodes-browserbase-secure` に変更し、公式パッケージとの競合を回避しています。

---

This is an n8n community node that lets you automate browsers using [Browserbase](https://browserbase.com) powered by [Stagehand](https://stagehand.dev) in your n8n workflows.

[n8n](https://n8n.io/) is a [fair-code licensed](https://docs.n8n.io/sustainable-use-license/) workflow automation platform.

## Installation

Follow the [installation guide](https://docs.n8n.io/integrations/community-nodes/installation/) in the n8n community nodes documentation.

## Development / Testing with Docker

```bash
# Install dependencies
npm install

# Build the node
npm run build

# Run n8n with your node in Docker
docker-compose up --build

# Open http://localhost:5678 and search for "Browserbase" node
```

To rebuild after changes:
```bash
npm run build && docker-compose up --build
```

## How It Works

The Browserbase Agent node is a single, self-contained node that:
1. Creates a browser session
2. Navigates to your starting URL
3. Executes an AI agent to complete your task
4. Closes the session automatically

Just provide a URL and an instruction - the node handles everything else.

## Configuration

### Required Fields

| Field | Description |
|-------|-------------|
| **Starting URL** | The page where the agent begins (e.g., `https://example.com`) |
| **Instruction** | Natural language task for the agent to complete |

### Driver Model

The driver model powers the browser session (navigation, DOM interactions). Choose from:

- `google/gemini-2.5-flash` (Recommended - fast & cheap)
- `google/gemini-2.5-pro`
- `openai/gpt-4o`
- `openai/gpt-4o-mini`
- `anthropic/claude-sonnet-4-5-20250929`

### Agent Mode

| Mode | Description | Best For |
|------|-------------|----------|
| **CUA** | Computer Use Agent - uses vision and coordinates | Complex UIs, visual interactions |
| **DOM** | Uses DOM selectors - works with any LLM | Speed, simple pages |
| **Hybrid** | Combines both approaches | Fallback reliability |

### Agent Models

Models available depend on the selected mode:

**CUA Mode:**
- `google/gemini-2.5-computer-use-preview-10-2025`
- `openai/computer-use-preview`
- `anthropic/claude-sonnet-4-20250514`
- `anthropic/claude-sonnet-4-5-20250929`
- `anthropic/claude-haiku-4-5-20251001`

**DOM Mode:**
- `google/gemini-2.5-flash`
- `google/gemini-2.5-pro`
- `openai/gpt-4o`
- `openai/gpt-4o-mini`
- `anthropic/claude-sonnet-4-5-20250929`

**Hybrid Mode:**
- `google/gemini-3-flash-preview`
- `anthropic/claude-sonnet-4-20250514`
- `anthropic/claude-haiku-4-5-20251001`

## Credentials

You need three credentials:

| Credential | Description |
|------------|-------------|
| **Browserbase API Key** | Your Browserbase API key |
| **Browserbase Project ID** | Your Browserbase project ID |
| **Model API Key** | API key for your chosen model provider |

> **Important:** The Model API Key must match the provider of your models. If using Google models, provide a Google API key. If using OpenAI, provide an OpenAI key.

### Getting Credentials

1. Sign up at [Browserbase](https://browserbase.com)
2. Navigate to your dashboard for API key and Project ID
3. Get an API key from your model provider:
   - [Google AI Studio](https://aistudio.google.com/apikey)
   - [OpenAI](https://platform.openai.com/api-keys)
   - [Anthropic](https://console.anthropic.com/)

## Example Usage

**Simple extraction:**
- URL: `https://news.ycombinator.com`
- Instruction: `Find the top 3 stories and return their titles and URLs`

**Form filling:**
- URL: `https://example.com/contact`
- Instruction: `Fill out the contact form with name "John Doe" and email "john@example.com", then submit`

**Navigation + action:**
- URL: `https://github.com`
- Instruction: `Search for "stagehand" and click on the first repository result`

## Output

The node returns an `AgentResult` object:

```json
{
  "success": true,
  "message": "Task completed successfully",
  "actions": [
    { "type": "act", "action": "clicked submit button" }
  ],
  "completed": true,
  "usage": {
    "input_tokens": 1250,
    "output_tokens": 340,
    "inference_time_ms": 2500
  },
  "sessionId": "abc-123"
}
```

## Compatibility

Compatible with n8n@1.60.0 or later

## Resources

- [n8n community nodes documentation](https://docs.n8n.io/integrations/#community-nodes)
- [Browserbase Documentation](https://docs.browserbase.com)
- [Stagehand Documentation](https://docs.stagehand.dev)
