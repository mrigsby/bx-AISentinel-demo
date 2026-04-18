# bx-AISentinel Demo

A ColdBox + CBWire chat app on BoxLang that demonstrates [bx-AISentinel](https://github.com/mrigsby/bx-AISentinel) protecting a live OpenRouter call.

## Why this demo exists

The default model this demo ships with — [`openrouter/elephant-alpha`](https://openrouter.ai/openrouter/elephant-alpha) — is **free** but explicitly says:

> *Prompts and completions may be logged by the provider and used to improve the model.*

That is exactly the real-world risk bx-AISentinel mitigates. The demo lets you paste secrets into a chat, watch the middleware tokenize them before the LLM sees anything, and watch the plaintext come back on the response — all in under 10 ms of overhead per turn.

Toggle the middleware off and send the same prompt; you'll see the unredacted payload reach the provider. Toggle it back on and every future request stays clean.

## Architecture at a glance

```text
Browser (CBWire)
     │
     ▼
ChatWire.send( userInput )
     │
     ▼
agent.run( userInput )    ← bx-ai aiAgent() with aiMemory("window", {maxMessages:10})
     │
     ▼
AiSentinelMiddleware::beforeLLMCall   ← redact → ⟦SECRET:LABEL:hmac8⟧
     │
     ▼
OpenRouter → openrouter/elephant-alpha
     │
     ▼
AiSentinelMiddleware::afterLLMCall    ← restore plaintext from manifest
     │
     ▼
Conversation transcript + timing badge
```

## Setup

### Prerequisites

- CommandBox 6+ (`brew install commandbox` or [commandbox.ortusbooks.com](https://commandbox.ortusbooks.com))
- An [OpenRouter](https://openrouter.ai) account + API key (elephant-alpha is free, but an account is required for the key)

### One-time install

```sh
# Clone the demo repo and drop into it
git clone https://github.com/mrigsby/bx-AISentinel-demo.git
cd bx-AISentinel-demo

# Install ColdBox, CBWire, TestBox, and pull bx-AISentinel straight from its repo
box install

# Copy the env template and paste your OpenRouter key
cp .env.example .env
#  → edit .env and set OPENROUTER_API_KEY
```

`box install` reads [box.json](box.json) and pulls [bx-AISentinel](https://github.com/mrigsby/bx-AISentinel) from its git repo into `modules_app/bx-aisentinel/`. No manual symlink or checkout needed.

### Run

```sh
box server start
```

The BoxLang server opens [http://localhost:8888](http://localhost:8888) in your browser.

## Using the demo

1. **Click a seeded prompt** (top of chat panel). They contain fake-but-realistic secrets: AWS keys, GitHub tokens, SSNs, customer emails, Postgres URIs, datasource blocks.
2. **Hit Send**. The assistant replies; a timing badge appears under the reply showing Sentinel overhead (green / amber / red) and the LLM round-trip time for comparison.
3. **Open the Inspector** (right sidebar). You'll see your plaintext prompt and the restored reply. Both stay local; the LLM only saw tokenized versions.
4. **Toggle Sentinel OFF** (switch at the top of the chat card). Send a seeded prompt again — the raw payload now reaches the provider. Toggle back ON.
5. **Toggle "Explain tokens to LLM"** to compare responses with and without the token-protocol system message. When on, a short primer is injected so the model treats `⟦SECRET:…⟧` placeholders verbatim instead of paraphrasing them.
6. **Check the dashboard** (`/dashboard`). Session-scoped totals: tokens minted, category breakdown, overhead ratio, recent runs.

## Running the tests

The demo ships with a small TestBox 7 suite under [`tests/`](tests/) covering the three local models (`SessionMetrics`, `SeededPrompts`, `AgentFactory`) and the `Main` handler's routing. With the server running:

- **Sidebar**: click **Developer → Run tests** to open the runner in a new tab.
- **Direct URL**: [http://localhost:8888/tests/runner.bxm](http://localhost:8888/tests/runner.bxm)
- **CLI**: `box testbox run` from the project root.

The middleware itself has its own standalone harness in the module repo — see [bx-AISentinel/tests](https://github.com/mrigsby/bx-AISentinel/tree/main/tests) for the full detector / core round-trip coverage.

## What to read next

- [bx-AISentinel README](https://github.com/mrigsby/bx-AISentinel#readme) — module overview and API
- [bx-AISentinel token protocol](https://github.com/mrigsby/bx-AISentinel/blob/main/includes/token-protocol.md) — the contract the middleware teaches the LLM when "Explain tokens to LLM" is on
- [OpenRouter models](https://openrouter.ai/models) — swap in a different `OPENROUTER_MODEL` in `.env`

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| `404` on `/chat` | `box install` didn't finish — ensure both `modules/cbwire/` and `modules_app/bx-aisentinel/` are present. |
| `Could not resolve bxAiSentinel.models.*` | The git install of the module didn't land. Re-run `box install` and confirm `modules_app/bx-aisentinel/ModuleConfig.bx` exists. Common causes: offline machine, expired GitHub token for a private repo, network block. |
| `401 Unauthorized` from OpenRouter | Your `.env` has a stale / missing `OPENROUTER_API_KEY`. Restart the server after editing `.env`. |
| Blank timing badge | The LLM call errored before `afterLLMCall` fired. Check the ColdBox error log and the Sentinel log (`logs/bx-aisentinel*.log`). |
| Model name unknown | Pick one from [openrouter.ai/models](https://openrouter.ai/models) and update `OPENROUTER_MODEL` in `.env`. |
