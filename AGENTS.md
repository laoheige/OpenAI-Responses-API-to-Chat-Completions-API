# AGENTS.md

## What this is

Single-file Node.js proxy (`proxy.js`) that converts OpenAI Responses API ↔ Chat Completions API. Used to connect Codex CLI to any Chat Completions backend.

## Run

```bash
node proxy.js          # reads config/config.json
```

No tests, no build step, no lint. Just `node proxy.js`.

## Key gotchas

- **SSE format is strict**: Must match `codex-rs/codex-api/src/sse/responses.rs` parsing. Events need exact fields (`response` for created/completed, `item` for added/done, `delta` for text deltas).
- **`sequence_number`**: Must be per-stream (not global), starting from 1. Each concurrent request has its own counter.
- **tool_call name dedup**: GLM re-sends full name in every chunk. Don't `+=` on name, just take first occurrence.
- **`fixToolCallArguments()`**: Always call before sending tool_call to client. Empty args → `"{}"`, non-object JSON → wrap in `{ value: ... }`.
- **`model_map`**: Codex sends `gpt-5.4-mini` (only model it recognizes). Map it to your actual backend model in config.

## Config

`config/config.json` — all settings with defaults:
- `backend_url`: Your Chat Completions endpoint
- `backend_api_key`: Added as `Authorization: Bearer` header
- `model_map`: Maps Codex model names to backend model names
- `models`: Returned by `/v1/models` endpoint

## Codex setup (~/.codex/config.toml)

```toml
openai_base_url = "http://127.0.0.1:57321/v1"
model_provider = "openai"
model = "gpt-5.4-mini"
```

Requires `OPENAI_API_KEY` env var (any value works).

## Routes

| Path | Method | Behavior |
|------|--------|----------|
| `/v1/responses` | POST | Convert Responses API → Chat Completions → SSE |
| `/v1/chat/completions` | POST | Direct passthrough |
| `/v1/models` | GET | Return configured models |
| `/health` | GET | Health check |

## Logs

`logs/YYYY-MM-DD.log` — daily files, also printed to console.
