# PCB Design Projects

All PCBs designed in **KiCad**. Source files (.kicad_pcb, .kicad_sch, .kicad_pro) will be added to their respective subfolders.

---

## CM5 Minima

A compact carrier board for the **Raspberry Pi Compute Module 5 (CM5)**.

**Design Goals:**
- Smaller footprint than the official CM5 IO Board
- Essential peripherals only: USB, HDMI, GPIO header, power input
- Industrial power input range (suitable for DIN rail PSU)
- Compatible with CM5 Lite (no eMMC) and CM5 variants

**Target Use Case:**
Integration into custom enclosures for embedded Linux applications — media players, home automation controllers, edge AI devices.

---

## Relay Controller Board

A custom **4-channel relay controller** PCB built around an ESP32, designed to replace consumer smart plugs with a reliable, repairable, open-source alternative.

**Design Features:**
- 4× relay channels (mains-rated, 10A)
- Optocoupler isolation between ESP32 logic and relay coils
- Onboard HLK-PM01 (mains to 5V) + AMS1117 (5V to 3.3V) regulation
- Status LED per relay channel
- Screw terminals for wire connections
- ESP32 programmable via USB-C

**Firmware:** ESPHome (see [`../esphome/`](../esphome/)) — integrates directly with Home Assistant for relay control via MQTT.

---

*KiCad project files (.kicad_pcb, schematic, BOM, Gerbers) coming soon.*
