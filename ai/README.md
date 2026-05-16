# AI / LLM Application Development

> **LangChain · LangGraph · Anthropic SDK · Pydantic · AI Agents · Tool Use · Structured Output · Local LLM Inference (Ollama)**

This directory contains my hands-on Python work on **LLM application development** — from the simplest "prompt → LLM → output" chain to a **graph-based, tool-using SRE agent** that operates my homelab.

The goal of this work is concrete: build the skills needed to deliver **AI-agent and LLM-application engineering** in production — chains, structured output, tool use, retrieval, observability, and the graph-based control flow that real agents need.

---

## Why this work matters (for hiring)

I am an **embedded systems engineer transitioning into LLM / AI-agent engineering**. Rather than only consume tutorials, I built and ran:

1. **A working multi-turn tool-using agent** against my own homelab (Proxmox, Home Assistant, Telegram) — not a toy chatbot.
2. **Both halves of LangGraph** — the prebuilt `create_react_agent` shortcut **and** a hand-built `StateGraph` with `ToolNode` + `tools_condition` — so I understand what the abstraction is doing under the hood.
3. **Both halves of tool use** — LangChain's `@tool` decorator **and** the raw Anthropic Messages API tool loop with `tool_use` / `tool_result` blocks.
4. **Typed, validated LLM output** with Pydantic — including range/enum constraints — instead of brittle string parsing.

Each file below is a self-contained, runnable example.

---

## Stack

| Layer | Choice |
|-------|--------|
| LLM provider | **Anthropic** (`claude-sonnet-4-6`) via `anthropic` SDK and `langchain-anthropic` |
| App framework | **LangChain** (chains, structured output, LCEL) + **LangGraph** (stateful agents) |
| Validation | **Pydantic v2** (typed structured output, schema-constrained generation) |
| Config | `python-dotenv` (API key via `.env`, never committed) |
| Data / analytics | `requests` · `pandas` · `matplotlib` |
| Local inference target | **Ollama** on Proxmox LXC with iGPU passthrough — see [`ollama-lxc-setup.md`](./ollama-lxc-setup.md) |
| Packaging (Agent_AI) | `pyproject.toml` (PEP 621), `uv`-compatible |

---

## Contents

### Building blocks — single concepts, runnable in isolation

| File | What it teaches | Concepts |
|------|-----------------|----------|
| [`basic.py`](./basic.py) | The minimum viable LLM call | Provider SDK, system + user messages |
| [`LCEL.py`](./LCEL.py) | LangChain Expression Language | `ChatPromptTemplate \| llm \| StrOutputParser` — composable chains |
| [`structured.py`](./structured.py) | Structured output (OpenAI SDK style) | Pydantic schema → typed `parsed` response |
| [`structure_io.py`](./structure_io.py) | Structured output via LangChain + Anthropic | `llm.with_structured_output(MovieReview)`, ranged ints, enums |
| [`pydantic_learn.py`](./pydantic_learn.py) | Pydantic fundamentals | `BaseModel`, validators, `Field` constraints |
| [`tools.py`](./tools.py) | Anthropic SDK **raw** tool-use loop | `tool_use` / `tool_result` blocks, multi-turn loop, real Open-Meteo API call |
| [`tool_uses.py`](./tool_uses.py) | Compact tool-use reference | The same pattern, distilled |

### LangGraph — stateful, graph-based agents

| File | What it teaches |
|------|-----------------|
| [`Langraph_prebuilt.py`](./Langraph_prebuilt.py) | `langgraph.prebuilt.create_react_agent` — the **fast path** to a ReAct agent with two `@tool`s (`get_weather`, `homelab_status`). Demonstrates `agent.invoke({"messages": [...]})` and `pretty_print()`. |
| [`custom_langraph.py`](./custom_langraph.py) | The **same agent, hand-built**: a typed `AgentState` with `add_messages` reducer, a `StateGraph` with `agent` and `tools` nodes, conditional routing via `tools_condition`, and the agent ↔ tools loop wired explicitly. This is what `create_react_agent` is doing internally — knowing both lets me debug real agent graphs, not just call a helper. |

### Data + analytics — practical Python for AI work

| File | What it does |
|------|--------------|
| [`get_data.py`](./get_data.py) | Pulls a week of weather data from the **Open-Meteo API**, loads it into a pandas DataFrame, and writes a clean CSV (`data/Erdweg_weather.csv`). Used as the data source for downstream charting / analysis. |
| [`hello.py`](./hello.py) · [`IPL.py`](./IPL.py) | Small Python warm-up scripts |

### `Agent_AI/` — the production-style agent

A standalone, **`uv`-packaged** project with its own `pyproject.toml`. This is the most representative piece of work in this folder.

**[`Agent_AI/agent_v1_raw.py`](./Agent_AI/agent_v1_raw.py) — HomelabSentinel**

A multi-turn, tool-using agent that acts as an **SRE for my homelab**. It is given a natural-language question ("Check on the openclaw container and mqtt broker. Fix anything broken."), and then:

1. **Plans** — decides which tool to call.
2. **Acts** — calls one of four real tools (currently stubbed for safety):
   - `check_proxmox_status(node, vmid)` — get LXC/VM status, memory %, uptime
   - `get_ha_entity(entity_id)` — read a Home Assistant entity state
   - `restart_lxc(node, vmid)` — destructive action, gated by reasoning
   - `send_telegram_alert(message)` — notify the operator
3. **Observes** — feeds the `tool_result` back into the model.
4. **Iterates** — up to 10 turns until `stop_reason == "end_turn"`.

**Why this matters:**
- It uses the **raw Anthropic Messages API** — no framework hiding the control loop. Every `tool_use` block, `tool_result` injection, and `stop_reason` check is explicit.
- The **system prompt encodes operational policy**: "always check status before restarting", "if mem_pct > 85, alert before restart", "if stopped, restart **and** alert". This is how you make an agent behave like an on-call engineer rather than a random LLM.
- **Destructive tools are isolated** (`restart_lxc`) and require the agent to reason its way to them — they're not the default action.
- The tool layer is **stubbed but realistic** — same function signatures and return shapes as the real Proxmox / HA / Telegram APIs, so swapping to live calls is a drop-in change.

**Files:**
- [`agent_v1_raw.py`](./Agent_AI/agent_v1_raw.py) — the agent loop and system prompt
- [`tools.py`](./Agent_AI/tools.py) — tool implementations (stubbed; matches real API shapes)
- [`pyproject.toml`](./Agent_AI/pyproject.toml) — `uv`-managed dependencies (anthropic, fastapi, httpx, langchain-anthropic, langgraph, pydantic, uvicorn) — ready to expose as an HTTP service
- [`main.py`](./Agent_AI/main.py) — entry point

---

## Running the examples

```bash
# 1. Install dependencies (root-level scripts)
pip install -r requirements.txt

# 2. Create a .env file with your Anthropic API key
echo "CLAUDE_API_KEY=sk-ant-..." > .env

# 3. Run any example
python LCEL.py
python structure_io.py
python tools.py
python Langraph_prebuilt.py
python custom_langraph.py

# 4. Run the homelab agent (uv-packaged)
cd Agent_AI
uv sync
uv run agent_v1_raw.py
```

For local inference (no API key, fully offline), point any of these scripts at the **Ollama LXC** described in [`ollama-lxc-setup.md`](./ollama-lxc-setup.md) — `claude-sonnet-4-6` can be swapped for `qwen2.5:3b` or `llama3.2:3b` via `ChatOllama`.

---

## Skills evidenced

| Skill | Where to look |
|-------|---------------|
| **LangChain LCEL chains** | [`LCEL.py`](./LCEL.py) |
| **Structured output with Pydantic** | [`structure_io.py`](./structure_io.py), [`structured.py`](./structured.py) |
| **Tool-use loop (raw Anthropic SDK)** | [`tools.py`](./tools.py), [`Agent_AI/agent_v1_raw.py`](./Agent_AI/agent_v1_raw.py) |
| **LangGraph — prebuilt ReAct agent** | [`Langraph_prebuilt.py`](./Langraph_prebuilt.py) |
| **LangGraph — hand-built `StateGraph`** | [`custom_langraph.py`](./custom_langraph.py) |
| **Multi-turn agent with operational policy** | [`Agent_AI/agent_v1_raw.py`](./Agent_AI/agent_v1_raw.py) |
| **Real API integration in agent tools** | [`tools.py`](./tools.py) (Open-Meteo), [`Agent_AI/tools.py`](./Agent_AI/tools.py) (Proxmox / HA / Telegram shape) |
| **Data wrangling around LLM workflows** | [`get_data.py`](./get_data.py) |
| **Local LLM inference / infrastructure** | [`ollama-lxc-setup.md`](./ollama-lxc-setup.md) |
| **Python packaging (PEP 621 / `uv`)** | [`Agent_AI/pyproject.toml`](./Agent_AI/pyproject.toml) |
| **Secret hygiene** | `.env` + `.gitignore`, no API keys in code |

---

## Where this is going

Next steps I'm actively working on:

- **Wiring `Agent_AI`'s stubbed tools to real Proxmox + HA APIs** behind a `dry_run` flag.
- **Wrapping it in FastAPI** (already in `pyproject.toml`) so the agent runs as a long-lived service, triggered from Home Assistant / Telegram.
- **Adding observability** — token usage, tool-call latency, decision traces — because debugging agents without traces is impossible.
- **Memory** — short-term conversational + long-term entity memory (Postgres + pgvector — already running in the homelab Docker stack).
- **Evaluation** — scripted scenarios (container stopped, memory at 90%, HA offline) to regression-test the agent's decisions as the prompt and toolset evolve.

---

*All code here was written, run, and debugged by me on my own infrastructure. Where I used AI assistance (Claude) it was as a pair-programmer — the architecture, the decisions, and the homelab integration are mine.*
