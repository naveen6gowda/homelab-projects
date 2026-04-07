# Local AI with Ollama on Proxmox LXC

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

This gives Ollama access to the integrated GPU for accelerated inference without a discrete GPU.

### Ollama Installation (inside LXC)

```bash
curl -fsSL https://ollama.com/install.sh | sh
systemctl enable --now ollama

# Pull a model
ollama pull qwen2.5:3b
ollama pull llama3.2:3b

# Test
ollama run qwen2.5:3b "Hello, what can you do?"
```

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

| Model | Size | Use Case |
|-------|------|----------|
| Qwen 3.6B | ~2.5GB | General assistant, coding help |
| Llama 3.2 3B | ~2.0GB | Fast responses, chat |

## Integration with Home Assistant

Ollama can be connected to Home Assistant via the **Ollama integration** or **OpenAI-compatible API**:

```
Base URL: http://<ollama-lxc-ip>:11434/v1
Model: qwen2.5:3b
```

This enables AI-powered automations, natural language device control, and local voice assistant responses — all processed on-device.
