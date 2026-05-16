# Agent_AI — HomelabSentinel

A multi-turn, tool-using **SRE agent for my homelab**, built directly on the Anthropic Messages API (no agent framework in the way).

Given a natural-language instruction, it plans tool calls, inspects Proxmox LXC/VM state and Home Assistant entities, decides if action is needed, and sends Telegram alerts before any destructive action.

This is a focused, raw-API SRE agent that runs **alongside the Hermes Agent** (the general-purpose local agent stack on the inference LXC, GPT-5.5 / ~35 tools / ~88 skills, which replaced the earlier **OpenClaw** stack on **2026-05-10**). HomelabSentinel exists as the deliberately minimal, framework-free counterpart so the full tool-use loop stays visible and easy to extend.

## Files

| File | Purpose |
|------|---------|
| `agent_v1_raw.py` | The agent loop — tool registry, system prompt (operational policy), `client.messages.create` → tool_use → tool_result iteration up to 10 turns |
| `tools.py` | Four tools: `check_proxmox_status`, `get_ha_entity`, `restart_lxc`, `send_telegram_alert` (stubs matching the real API shapes) |
| `main.py` | Entry point |
| `pyproject.toml` | `uv`-managed deps: anthropic, langchain-anthropic, langgraph, fastapi, httpx, pydantic, uvicorn, python-dotenv |

## Operational policy (encoded in the system prompt)

- Always check status before restarting anything.
- If `mem_pct > 85`, recommend restart **but send a Telegram alert first**.
- If a container is stopped, restart it **and** alert.
- Explain reasoning briefly at the end.

## Run

```bash
echo "ANTHROPIC_API_KEY=sk-ant-..." > .env
uv sync
uv run agent_v1_raw.py
```

Default test query (in `agent_v1_raw.py`):

> "Check on the openclaw container (vmid 100) and mqtt broker (vmid 102). Fix anything broken."

## Why this design

- **Raw Messages API, not a framework wrapper** — every `tool_use` block, `tool_result` injection, and `stop_reason` check is explicit. This is the layer real agent frameworks build on.
- **Tools are stubbed but realistic** — same signatures and return shapes as the live Proxmox / HA / Telegram APIs, so swapping to live calls is a single file change.
- **Destructive actions are gated by reasoning** — `restart_lxc` is in the tool registry, but the system prompt makes the model reason its way to it (check status first, alert first, then act).
- **`uv` + `pyproject.toml`** — modern Python packaging; ready to be wrapped as a FastAPI service (deps already declared).
