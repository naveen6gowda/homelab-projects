# PCB Design Projects

All PCBs designed in **KiCad**. Source files (.kicad_pcb, .kicad_sch, .kicad_pro) will be added to their respective subfolders.

---

## CM5 Minima

A compact carrier board for the **Raspberry Pi Compute Module 5 (CM5)**.

> **Note:** This design is based on an existing open-source CM5 Minima carrier board from GitHub — not designed from scratch. The key modification was adding a **Zigbee/Thread module (ESP32-C6-MINI-1)** for wireless connectivity.

**Design Goals:**
- Smaller footprint than the official CM5 IO Board
- Essential peripherals only: USB, HDMI, GPIO header, power input
- Industrial power input range (suitable for DIN rail PSU)
- Compatible with CM5 Lite (no eMMC) and CM5 variants
- **Added:** ESP32-C6-MINI-1 Zigbee/Thread module for local wireless protocol support

**Target Use Case:**
Integration into custom enclosures for embedded Linux applications — media players, home automation controllers, edge AI devices.

---

## Relay Controller Board

A custom **2-channel relay controller** PCB built around an **ESP32-C6-MINI-1 module**, designed to replace consumer smart plugs with a reliable, repairable, open-source alternative.

**Design Features:**
- 2× relay channels (mains-rated, 10A)
- Optocoupler isolation between ESP32-C6 logic and relay coils
- Onboard HLK-PM01 (mains to 5V) — ESP32-C6-MINI-1 module used directly (no external LDO required)
- Status LED per relay channel
- Screw terminals for wire connections
- ESP32-C6-MINI-1 programmable via USB-C

**Firmware:** ESPHome (see [`../esphome/`](../esphome/)) — integrates directly with Home Assistant for relay control via MQTT.

---

*KiCad project files (.kicad_pcb, schematic, BOM, Gerbers) coming soon.*
