# Local AI on Proxmox LXC — llama.cpp + Hermes (current), Ollama + OpenClaw (earlier)

This LXC has gone through two local-AI stacks:

| Phase | Inference engine | Agent stack |
|-------|------------------|-------------|
| **Earlier** | **Ollama** — easy model management, OpenAI-compatible API on `:11434`, integrates cleanly with Home Assistant and Open WebUI | **OpenClaw** — first local agent stack I ran on top of Ollama |
| **Current** | **llama.cpp** (Vulkan backend) — direct GGUF inference with full control over quantization, context length, and `llama-server` tuning | **Hermes Agent** v0.13.0 (~35 tools / 88 skills, GPT-5.5) — migrated from OpenClaw via `hermes claw migrate` |

The LXC and iGPU passthrough stayed the same across both migrations — only the engine and agent changed. The Ollama install instructions below are kept as historical reference and because it's still useful as a quick OpenAI-compatible endpoint for the LangChain / LangGraph examples in this folder.

## Why Self-Hosted AI?

| Aspect | Cloud API | Self-Hosted (this setup) |
|--------|-----------|--------------------------|
| Privacy | Data sent to provider | Stays on local hardware |
| Cost | Per-token billing | One-time hardware cost |
| Latency | Network dependent | Sub-100ms local |
| Availability | Internet required | Works offline |
| Control | Provider's models | Any model, any time |

## Setup

### LXC Container Configuration (Proxmox)

```
CT ID: 101
OS: Ubuntu
CPU: 4 cores
RAM: 4096 MB
Disk: 44GB (local-lvm)
Network: VLAN 178, static IP
Features: nesting=1, keyctl=1 (unprivileged)
```

### iGPU Passthrough

```ini
# /etc/pve/lxc/101.conf
lxc.cgroup2.devices.allow: c 226:128 rwm   # renderD128
lxc.cgroup2.devices.allow: c 226:0 rwm     # card0
lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file
lxc.mount.entry: /dev/dri/card0 dev/dri/card0 none bind,optional,create=file
```

This gives whichever inference engine is running (llama.cpp today, Ollama in the earlier setup) access to the integrated GPU for accelerated inference without a discrete GPU.

### Ollama Installation (inside LXC) — earlier setup

```bash
curl -fsSL https://ollama.com/install.sh | sh
systemctl enable --now ollama

# Pull a model
ollama pull qwen2.5:3b
ollama pull llama3.2:3b

# Test
ollama run qwen2.5:3b "Hello, what can you do?"
```

### llama.cpp Installation (inside LXC) — current setup

Built from source for the same LXC, with the Vulkan backend so it shares the iGPU with Ollama:

```bash
# Build with Vulkan (iGPU acceleration)
sudo apt install -y build-essential cmake git libvulkan-dev vulkan-tools
git clone https://github.com/ggerganov/llama.cpp ~/llama.cpp
cmake -S ~/llama.cpp -B ~/llama.cpp/build -DGGML_VULKAN=ON
cmake --build ~/llama.cpp/build --config Release -j

# Pull a GGUF model (any HF GGUF works)
mkdir -p ~/models && cd ~/models
huggingface-cli download bartowski/Qwen2.5-3B-Instruct-GGUF \
    Qwen2.5-3B-Instruct-Q4_K_M.gguf --local-dir .

# Run the OpenAI-compatible server
~/llama.cpp/build/bin/llama-server \
    -m ~/models/Qwen2.5-3B-Instruct-Q4_K_M.gguf \
    --host 0.0.0.0 --port 8080 \
    -ngl 99 -c 8192
```

llama.cpp's `llama-server` exposes the same `/v1/chat/completions` shape as Ollama, so any LangChain / LangGraph code in this folder can target it by changing the `base_url`. Why I moved here from Ollama:

- Run a quantization Ollama doesn't ship (e.g. `IQ3_XXS`).
- Pin a specific GGUF for reproducible benchmarks.
- Squeeze the iGPU harder with `-ngl 99 --batch-size N --ubatch-size N`.
- It's the inference layer the **Hermes Agent** sits on top of.

### Hermes Agent on top of llama.cpp — current agent stack

[Hermes](https://github.com/ollama/...) v0.13.0 replaced **OpenClaw** as my local agent stack on **2026-05-10**. Hermes runs out of `/home/ollama/.hermes/` and ships ~35 tools and ~88 skills against the local `llama-server` endpoint. The model used for agentic reasoning is **GPT-5.5**.

```bash
# Migrating from the earlier OpenClaw setup
hermes claw migrate          # imports config from ~/.openclaw/
hermes status                # check engine + tool availability
hermes run "<instruction>"   # run an agentic instruction
```

OpenClaw is no longer actively used — keeping it referenced here because the migration path and the design decisions that pushed me from Ollama→llama.cpp and OpenClaw→Hermes are part of the work.

### VS Code Remote Development

```
# ~/.ssh/config (on laptop)
Host ollama
    HostName <ollama-lxc-ip>
    User ollama
    IdentityFile ~/.ssh/id_ed25519
    Port 22
```

Connect in VS Code → Remote SSH → `ollama` → full IDE inside the LXC container.

## Models Used

| Model | Size | Engine | Use Case |
|-------|------|--------|----------|
| Qwen 2.5 3B (Q4_K_M) | ~2.0GB | Ollama / llama.cpp | General assistant, coding help |
| Llama 3.2 3B | ~2.0GB | Ollama | Fast responses, chat |
| Qwen 2.5 3B GGUF (custom quant) | ~1.6GB | llama.cpp | Benchmarking, low-RAM tests |

## Integration with Home Assistant

Either engine can be connected to Home Assistant via the OpenAI-compatible API. I switch between them via base URL:

```
# Ollama (port 11434, model lookup by name)
Base URL: http://<lxc-ip>:11434/v1
Model: qwen2.5:3b

# llama.cpp llama-server (port 8080, model fixed at launch)
Base URL: http://<lxc-ip>:8080/v1
Model: any (llama-server serves whichever GGUF was loaded with -m)
```

This enables AI-powered automations, natural language device control, and local voice assistant responses — all processed on-device.
