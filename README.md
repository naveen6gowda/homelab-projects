# Naveen Gowda — AI, Embedded Systems & Home Automation Portfolio

> **Embedded Systems Engineer exploring AI / LLMs** | LangChain · LangGraph · AI Agents · Local LLM Infrastructure · PCB Design · ESP32 · Home Assistant · Docker

This repository showcases hands-on work across **AI / LLM application development (LangChain, LangGraph, AI agents)**, **local AI infrastructure**, and **embedded / home-automation projects** built with real hardware and deployed in a production home environment. Every project here runs 24/7 and was designed, debugged, and refined through real-world use.

**Related repositories:**
- [`ai/`](./ai/) — LangChain / LangGraph chains, tool-use agents, and a homelab SRE agent (in this repo)
- [KiCad-projects](https://github.com/naveen6gowda/KiCad-projects) — PCB designs (CM5 carrier, relay controller)

---

## About Me

I am an embedded systems engineer with strong hands-on interest in **AI / LLMs** — local inference, LangChain / LangGraph application development, and AI-agent workflows — alongside a background in IoT and embedded hardware. My work spans:

- **AI / LLM application development** — LangChain & LangGraph (Python) for chains, structured output, tool use, and graph-based agent workflows ([`ai/`](./ai/))
- **AI agents for real systems** — a homelab SRE agent that reads Proxmox + Home Assistant state through tools and acts on it
- **Local AI infrastructure** — self-hosted LLM inference on custom Proxmox homelab with iGPU passthrough (Ollama, Open WebUI)
- **PCB design** for custom embedded boards (KiCad) — from schematic to 3D-rendered production layout
- **Firmware development** for ESP32 microcontrollers using ESPHome / C++ (ESP-IDF & Arduino framework)
- **Home automation** integration with Home Assistant via MQTT and native API
- **Linux systems** — Proxmox hypervisor, LXC containers, Docker, VLANs, OPNsense routing

---

## Project Overview

| # | Project | Hardware / Stack | Key Technologies |
|---|---------|------------------|-----------------|
| 1 | [LangChain & LangGraph AI Agents](#1-langchain--langgraph-ai-agents) | Python | LangChain, LangGraph, Anthropic SDK, Pydantic, Tool use |
| 2 | [AI Homelab Infrastructure](#2-ai-homelab-infrastructure) | Proxmox + LXC | Ollama, Local LLM, iGPU passthrough |
| 3 | [Docker Self-Hosted Services](#3-docker-self-hosted-services) | Debian VM | 26 containers, Immich (ML), Open WebUI, n8n |
| 4 | [PCB Design — CM5 Minima REV3](#4-pcb-design--cm5-minima-rev3) | KiCad | CM5 carrier board, Hailo-8 AI accelerator, M.2, RJ45 |
| 5 | [PCB Design — Relay Controller](#5-pcb-design--relay-controller) | KiCad | ESP32-C6-MINI-1, 2-ch relay, optocoupler isolation |
| 6 | [Mailbox Alert](#6-mailbox-alert) | ESP32-C6 SuperMini | Deep sleep, MQTT, Reed switch, AHT21 |
| 7 | [Plant Moisture Monitor](#7-plant-moisture-monitor) | ESP32-C3 | ADC, OLED, Deep sleep, Battery optimized |
| 8 | [Hall Clock & Presence Display](#8-hall-clock--presence-display) | ESP32-C3 | LD2410C Radar, SSD1306 OLED, Weather |
| 9 | [Touch Voice Assistant Dashboard](#9-touch-voice-assistant-dashboard) | ESP32-S3 (16MB+PSRAM) | I2S audio, Wake word, Touchscreen |
| 10 | [Room Environment Monitors](#10-room-environment-monitors) | ESP32-C3 | BME280, AHT20, ENS160 CO2/AQI |
| 11 | [Kitchen Smart Display](#11-kitchen-smart-display) | ESP32-S3 | Relay control, Multi-page UI, Alerts |
| 12 | [E-Paper Display](#12-e-paper-display) | ESP32-C3 | E-ink, Low power, Home Assistant data |

---

## 1. LangChain & LangGraph AI Agents

**Directory:** [`ai/`](./ai/)

Hands-on Python work building LLM applications end-to-end — from simple chains to graph-based, tool-using agents. Code, examples, and a homelab SRE agent live in [`ai/`](./ai/).

**Highlights:**
- **LangChain Expression Language (LCEL)** — prompt → model → output-parser chains
- **Structured output with Pydantic** — typed, validated LLM responses (ratings, enums, nested models)
- **Tool use with the Anthropic SDK** — raw multi-turn tool-calling loop against `claude-sonnet-4-6`
- **LangGraph (prebuilt + custom)** — `create_react_agent` and a hand-built `StateGraph` with `ToolNode`/`tools_condition`
- **HomelabSentinel agent** — an autonomous SRE agent that inspects Proxmox LXC/VM state and Home Assistant entities through tools, decides if a restart is needed, and sends Telegram alerts
- **Real-data analytics** — Open-Meteo weather pull → pandas DataFrame → matplotlib chart

**Stack:** Python · LangChain · LangGraph · Anthropic SDK · Pydantic · `python-dotenv` · pandas · matplotlib · Ollama (local inference target)

---

## 2. AI Homelab Infrastructure

**Directory:** [`homelab/`](./homelab/) · [`ai/`](./ai/)

A self-hosted AI inference stack running locally — no cloud, no subscription, full control. This is the platform that hosts the local LLMs used as inference targets for the agent work above.

**Infrastructure:**
- **Proxmox VE** hypervisor on a mini-PC (x86, 4-core, 16GB RAM)
- **Ollama LXC container** (Ubuntu) with iGPU passthrough (`/dev/dri/renderD128`) for accelerated inference
- **Home Assistant OS** VM with ESPHome add-on managing all ESP32 devices
- **OPNsense** firewall/router with VLAN segmentation (IoT, HA, Management)
- **Tailscale** — secure remote access without exposing any ports

**AI Stack:**
```
Proxmox Host
├── Ollama LXC — Local LLM inference (iGPU accelerated)
│   └── Models: Qwen 3.6B, Llama 3.2, and others on demand
├── Home Assistant OS VM
│   └── ESPHome add-on → manages all 9 ESP32 devices OTA
└── OPNsense: VLAN routing + firewall
    ├── VLAN IoT    — all ESP32 nodes (isolated)
    ├── VLAN HA     — Home Assistant
    └── VLAN Mgmt   — Proxmox, Ollama, management
```

**Why self-hosted:** Privacy (no data leaves home), zero latency, no API costs, and real infrastructure experience for local LLM serving and agent workflows.

---

## 3. Docker Self-Hosted Services

**Directory:** [`docker/`](./docker/)

A 26-container self-hosted stack running on a Debian VM — replacing cloud services with privacy-respecting, locally-controlled alternatives.

| Category | Services |
|----------|----------|
| **Media** | Jellyfin (movies/TV), Immich (photos + ML face recognition) |
| **AI / LLM** | Open WebUI → Ollama (local GPU) + OpenRouter (cloud fallback), Mirofish |
| **Automation** | n8n (visual workflow automation, Home Assistant integration) |
| **Security** | Vaultwarden (self-hosted Bitwarden), AdGuard Home (DNS ad blocker) |
| **Finance** | Firefly III (personal finance & budgeting) |
| **Bookmarks** | Linkwarden (archiving + Meilisearch full-text search) |
| **Management** | Portainer, Watchtower (auto-update), Dozzle (log viewer), Prunemate |
| **Sync & Backup** | Syncthing (P2P file sync), Duplicati (encrypted backups) |
| **Databases** | PostgreSQL with pgvector, MariaDB, Redis |

**Key highlights:**
- **Zero cloud dependencies** — all data stays on local hardware
- **Immich ML** runs on-device CLIP semantic search and face recognition
- **Open WebUI** bridges local Ollama (GPU-accelerated) with OpenRouter as cloud fallback
- **n8n** automates workflows between Home Assistant, Immich, and external services
- **Watchtower** auto-updates all containers weekly with Discord notifications

Full compose file (secrets removed) + architecture diagram: [`docker/`](./docker/)

---

## 4. PCB Design — CM5 Minima REV3

**Repository:** [github.com/naveen6gowda/KiCad-projects](https://github.com/naveen6gowda/KiCad-projects)

A compact carrier board for the **Raspberry Pi Compute Module 5 (CM5)** — based on an existing open-source CM5 Minima design from GitHub, adapted and extended with a **Zigbee/Thread module (ESP32-C6-MINI-1)** addition for wireless connectivity. The base design was not created from scratch; the primary contribution was integrating and modifying the board to accommodate the ESP32-C6 Zigbee/Thread module.

> **Note:** This is a derivative of an open-source CM5 carrier board design. The Zigbee/Thread (ESP32-C6-MINI-1) module integration was added on top of the existing layout.

**What's in the KiCad repo:**
- Complete schematic (`.kicad_sch`) and PCB layout (`.kicad_pcb`) — routed, DRC clean
- Custom footprint library (`CM5IO.pretty`) — CM5 module, M.2 sockets, Hirose connectors
- Custom 3D models (`3d_lib/`) — Hirose FH12, JST BM series, SHTC3, RJ45, Hailo M.2
- Blender renders (`.pcb3d`) and CNC enclosure design (`.step`)
- PDF schematic for easy review

**Design Highlights:**
- **Hailo-8 M.2 M-key slot** — AI accelerator (8 TOPS) for on-device inference
- **M.2 B-key 2230 slot** — NVMe SSD storage
- **RJ45 Gigabit Ethernet** with magnetics
- **USB 3.0** (stacked connector), DSI, CSI camera interfaces
- **SHTC3** onboard temperature/humidity sensor
- **4× M2.5 mounting holes** for CNC enclosure integration

---

## 5. PCB Design — Relay Controller

**Repository:** [github.com/naveen6gowda/KiCad-projects](https://github.com/naveen6gowda/KiCad-projects)

A custom **ESP32-C6-MINI-1-based 2-channel relay controller** PCB — the hardware behind the relay switch automations running in the ESPHome projects below.

**Design Features:**
- 2-channel relay (mains-rated, 10A per channel)
- **Optocoupler isolation** between ESP32-C6 logic and relay coils — protects the MCU from mains transients
- Onboard **HLK-PM01** mains-to-5V — single mains input, no external PSU
- **ESP32-C6-MINI-1 module** used directly — module has onboard regulation, no external LDO required
- Status LED per relay channel
- Screw terminals for field wiring

---

## ESPHome Projects

All ESPHome devices integrate with **Home Assistant** on an isolated IoT VLAN. YAML configs are in [`/esphome`](./esphome/) (credentials removed — use `secrets.yaml` for your deployment).

---

## 6. Mailbox Alert

**File:** [`esphome/mailbox-alert.yaml`](./esphome/mailbox-alert.yaml)
**Workflow:** Spec-driven development using [GitHub Spec Kit](https://github.com/github/spec-kit) (`specify` CLI) — features were specified as structured markdown specs before implementation, enabling predictable, reviewable AI-assisted development.

A battery-powered IoT mailbox sensor that wakes from deep sleep when mail is delivered and sends an instant notification to Home Assistant.

**Hardware:**
- ESP32-C6 SuperMini (Tenstar)
- MC-38 reed switch (door/lid sensor)
- AHT21 temperature + humidity sensor
- 470kΩ/470kΩ voltage divider for battery monitoring

**Key Technical Features:**
- **Deep sleep architecture** — `ext1` wakeup on GPIO4 (reed switch) + 60-minute timer fallback
- **MQTT fire-and-forget** transport — no HA API dependency, works even if HA is briefly offline
- **Crash guard** at priority 800 — device sleeps before WiFi starts if a crash loop is detected
- **Battery voltage monitoring** — ADC with resistor divider, reported to HA
- **WiFi brownout prevention** — 15dBm TX power limit + light power save mode
- Temperature & humidity logged on every 60-minute timer wake

**Architecture:**
```
Lid opens → GPIO4 HIGH → ext1 wakeup → WiFi → MQTT publish → deep sleep
Timer (60min) → wakeup → temp/humidity publish → deep sleep
```

---

## 7. Plant Moisture Monitor

**File:** [`esphome/plant-moisture.yaml`](./esphome/plant-moisture.yaml)

A battery-optimized soil moisture sensor with OLED display. Completely rewritten from scratch to fix bugs causing zero data to reach Home Assistant and excessive battery drain.

**Hardware:**
- ESP32-C3 DevKitM-1
- Capacitive soil moisture sensor (ADC on GPIO1)
- SSD1306 128×64 OLED display (I2C)
- Wake button (GPIO5)

**Key Technical Features:**
- **Dual wakeup modes** — 3-hour timer (silent, no display) + button press (display ON)
- **Median filter** on ADC — 3 readings averaged to eliminate noise
- **OLED power management** — hardware sleep (0xAE/0xAF commands) to prevent phantom current draw
- **MQTT retained** publish + ESPHome native API — HA auto-discovers all entities
- **OTA without physical access** — toggle "Prevent Deep Sleep" switch in HA to keep device awake for flashing
- **Bug fixed:** ESP32-C3 uses `ESP_SLEEP_WAKEUP_GPIO` not `EXT0` — was breaking display-on logic

**Display Layout:**
```
┌─────────────────┐
│   Soil Monitor  │
├─────────────────┤
│      67%        │
│ ████████░░░░░   │
│      MOIST      │
│ WiFi:OK   3.7V  │
└─────────────────┘
```

---

## 8. Hall Clock & Presence Display

**File:** [`esphome/hall-clock.yaml`](./esphome/hall-clock.yaml)

A hallway clock with live time, weather, temperature, humidity, and radar-based presence detection — all on a compact OLED display.

**Hardware:**
- ESP32-C3 DevKitM-1
- SSD1306 128×64 OLED (I2C)
- LD2410C 24GHz mmWave radar sensor (UART)
- BME280 temperature/pressure/humidity sensor

**Key Technical Features:**
- **LD2410C radar** — moving target and stationary presence detection with distance reporting
- **Material Design Icons** font for weather symbols (sunny, cloudy, rain, snow, etc.)
- **Weather condition** pulled from HA (`weather.home` entity) — icon updates automatically
- **Dual time source** — Home Assistant primary, SNTP fallback for reliability
- **Output power tuned** to fix WiFi connectivity in thick-wall environment

---

## 9. Touch Voice Assistant Dashboard

**File:** [`esphome/touch-voice-assistant.yaml`](./esphome/touch-voice-assistant.yaml)

A wall-mounted smart home dashboard combining touchscreen control, ambient sensor display, and a fully local voice assistant — all running on a single ESP32-S3.

**Hardware:**
- ESP32-S3 DevKitC-1 (16MB Flash, Octal PSRAM @ 80MHz)
- 3.5" TFT touchscreen (ST7789V, SPI)
- I2S MEMS microphone (INMP441)
- I2S DAC speaker output
- AHT20 temperature/humidity sensor
- ENS160 CO2/TVOC/AQI sensor

**Key Technical Features:**
- **Wake word detection** — "Okay Nabu" via `micro_wake_word` (on-device ML inference)
- **Voice assistant** — streams audio to Home Assistant, plays back TTS response via speaker
- **Noise suppression level 4** + 6× volume amplification for reliable mic pickup
- **Multi-page touchscreen UI** — home dashboard, device controls, sensor data
- **Relay control** — toggle lights and switches directly from the touchscreen
- **Washer done alert** + **mailbox alert** icons on dashboard
- **Mute switch** — disables wake word listening without rebooting

**Voice Flow:**
```
"Okay Nabu" → micro_wake_word → voice_assistant.start → STT → LLM → TTS → speaker
```

---

## 10. Room Environment Monitors

**Files:**
- [`esphome/bathroom-monitor.yaml`](./esphome/bathroom-monitor.yaml)
- [`esphome/livingroom-monitor.yaml`](./esphome/livingroom-monitor.yaml)
- [`esphome/office-monitor.yaml`](./esphome/office-monitor.yaml)

Distributed sensor network covering every room — temperature, humidity, CO₂, and air quality — all reporting to Home Assistant dashboards.

**Hardware (per device):**
- ESP32-C3 SuperMini
- BME280 or AHT20 (temperature + humidity)
- ENS160 (CO₂ / TVOC / AQI) — living room and office
- SSD1306 OLED local display

**Key Technical Features:**
- **Sliding window moving average** on all sensors (window=3) — smooth readings, no spikes
- **Watchdog** — auto-restart if device freezes (prevents silent failures)
- **Presence-aware display** — screen shows live data; dims when room is unoccupied
- **Battery voltage monitoring** on bathroom monitor (runs on USB power bank backup)
- **CO₂ compensation** — ENS160 uses live temperature/humidity to correct AQI readings
- All devices on static IPs, VLAN-segmented IoT network

---

## 11. Kitchen Smart Display

**File:** [`esphome/kitchen-display.yaml`](./esphome/kitchen-display.yaml)

A kitchen control panel with live sensor data, appliance status, and one-touch control of all smart home devices in the home.

**Hardware:**
- ESP32-S3 (custom partition table for 16MB flash)
- 3.5" TFT display
- GT911 capacitive touchscreen controller

**Key Technical Features:**
- **Multi-page UI** — Page 1: appliance control, Page 2: sensor overview
- **Relay toggle** for bedroom light, passage light, TV socket, backlight strip
- **Washer completion alert** — icon appears when washing machine finishes, dismissable by touch
- **Mailbox alert** — icon appears when mail arrives (from mailbox sensor via HA)
- **Day counter** — tracks days since last event (persistent across reboots via RTC)
- **Safe mode OTA** — survives firmware crash, always recoverable wirelessly
- **Dual time source** — HA primary + SNTP fallback

---

## 12. E-Paper Display

**File:** [`esphome/epaper.yaml`](./esphome/epaper.yaml)

A low-power e-ink display node showing Home Assistant data. E-paper retains the last image with zero power, making it ideal for at-a-glance information panels.

**Hardware:**
- ESP32-C3 DevKitM-1
- E-paper (e-ink) display module

**Key Technical Features:**
- Boots and updates display, then idles — zero power between updates
- Pulls live data from Home Assistant via native API
- ESP-IDF framework for lower power consumption

---

## Skills Demonstrated

| Skill | Evidence |
|-------|----------|
| **LLM application development** | LangChain & LangGraph chains, structured output, LCEL, ReAct agents — see [`ai/`](./ai/) |
| **AI-agent architectures** | HomelabSentinel: multi-turn tool-using agent over Proxmox + Home Assistant — see [`ai/Agent_AI/`](./ai/Agent_AI/) |
| **Structured LLM output** | Pydantic-typed responses, validation, enum constraints — see [`ai/structure_io.py`](./ai/structure_io.py) |
| **Local AI infrastructure** | Ollama, iGPU passthrough, Open WebUI, model management on Proxmox homelab |
| PCB design (KiCad) | CM5 Minima REV3 (based on open-source design, extended with ESP32-C6 Zigbee/Thread module), relay controller — schematic to production |
| ESP32 firmware (ESPHome/C++) | 9 production devices — deep sleep, ADC, I2C, SPI, UART, I2S |
| MQTT protocol | Fire-and-forget, retained topics, HA auto-discovery |
| Battery optimization | Crash guards, deep sleep sequencing, display power management |
| Docker (self-hosted) | 26-container compose stack — Immich, n8n, Jellyfin, Vaultwarden, Watchtower |
| Home Assistant integration | 9 ESPHome nodes, MQTT, native API, entities, automations |
| Linux systems | Proxmox, LXC, VLANs, SSHFS, systemd, Debian |
| Networking | OPNsense, VLANs, Tailscale VPN, DNS (AdGuard Home) |
| Debugging / root-cause analysis | Detailed bug fix notes in YAML comments |

---

## Repository Structure

```
homelab-projects/
├── README.md                        ← You are here
├── esphome/                         ← ESPHome YAML configs (secrets removed)
│   ├── mailbox-alert.yaml
│   ├── plant-moisture.yaml
│   ├── hall-clock.yaml
│   ├── touch-voice-assistant.yaml
│   ├── bathroom-monitor.yaml
│   ├── livingroom-monitor.yaml
│   ├── kitchen-display.yaml
│   ├── office-monitor.yaml
│   └── epaper.yaml
├── homelab/                         ← Homelab infrastructure docs
│   └── infrastructure.md
├── ai/                              ← LangChain / LangGraph code + AI agents + Ollama setup
│   ├── README.md
│   ├── ollama-lxc-setup.md
│   ├── LCEL.py                      ← LangChain Expression Language chain
│   ├── structure_io.py              ← Structured output with Pydantic
│   ├── tools.py                     ← Anthropic SDK tool-use example
│   ├── Langraph_prebuilt.py         ← LangGraph create_react_agent
│   ├── custom_langraph.py           ← Hand-built LangGraph StateGraph
│   ├── get_data.py                  ← Open-Meteo weather → pandas
│   └── Agent_AI/                    ← HomelabSentinel SRE agent (Proxmox + HA + Telegram tools)
├── docker/                          ← Docker Compose stack (26 containers, secrets removed)
│   ├── README.md
│   └── docker-compose.yml
└── pcb/                             ← PCB project descriptions + link to KiCad repo
    └── README.md
```

---

## Contact

- **Email:** naveen6gowda@gmail.com
- **GitHub:** [github.com/naveen6gowda](https://github.com/naveen6gowda)
- **Location:** Germany

---

*All projects in this repository are real, deployed, and actively maintained. ESPHome YAML files have WiFi credentials and API keys replaced with placeholders — use a `secrets.yaml` file in production.*

*This portfolio — including the homelab infrastructure setup, PCB modifications, and ESPHome firmware — was developed with the assistance of AI (Claude by Anthropic). The designs, decisions, and deployments are mine; AI helped accelerate the process.*
