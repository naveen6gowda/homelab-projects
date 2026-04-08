# Homelab Infrastructure

## Overview

A production home network built on enterprise-grade open-source tools, running 24/7 on a mini-PC.

## Hardware

| Component | Details |
|-----------|---------|
| Host machine | x86 mini-PC, 4-core CPU, 16GB RAM, SSD |
| Hypervisor | Proxmox VE (bare metal) |
| Router/Firewall | OPNsense (separate device) |
| IoT devices | 9× ESP32 nodes (various models) |

## Proxmox Virtual Machines & Containers

### Home Assistant OS (HAOS) VM

- **Network:** VLAN-segmented HA subnet (static IP)
- **Purpose:** Central home automation hub
- **Add-ons running:**
  - ESPHome — manages all ESP32 firmware OTA
  - MQTT Broker (Mosquitto)
  - Advanced SSH & Web Terminal
- **Integrations:** 50+ entities from ESP32 sensors, relay switches, weather

### Ollama LXC Container (Local AI Inference)

- **OS:** Ubuntu
- **Network:** Management VLAN (static IP)
- **Purpose:** Self-hosted LLM inference — no cloud, no API costs
- **GPU passthrough:** `/dev/dri/renderD128` (iGPU accelerated inference)
- **Models:** Qwen 3.6B, Llama 3, and others
- **Access:** VS Code Remote SSH for development

## Network Architecture

```
Internet
    │
OPNsense Firewall / Router
    │
    ├── VLAN: Management
    │   ├── Proxmox host
    │   └── Ollama LXC (AI inference)
    │
    ├── VLAN: Home Assistant
    │   └── HA VM (HAOS)
    │
    └── VLAN: IoT Devices (isolated)
        ├── Mailbox sensor
        ├── Plant moisture monitor
        ├── Hall clock
        └── ... (all 9 ESP32 nodes)
```

## Remote Access

- **Tailscale VPN** — secure remote access to all services without port forwarding
- **No exposed ports** to the internet

## Key Design Decisions

- **VLAN segmentation** — IoT devices cannot reach management network; HA is the only bridge
- **Local-first** — Ollama, MQTT broker, and ESPHome all run locally; no cloud dependencies
- **MQTT over native API** for battery devices — fire-and-forget avoids HA API reconnect delays during short wake windows

---

*This infrastructure was designed and deployed with AI assistance (Claude by Anthropic).*
