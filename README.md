https://doi.org/10.5281/zenodo.20987896

# ESP32-S3 Distributed Heterogeneous Computing Cluster

**A Self-Evolving Embedded Architecture with Hierarchical AI and Autonomous OTA Infrastructure**

This repository contains the complete system architecture specification for a distributed embedded computing cluster built around the ESP32-S3 platform.

---

## Overview

Single-chip solutions face performance bottlenecks when computation, scheduling, rendering, and peripheral management compete for CPU time, memory bandwidth, and bus resources. This architecture physically separates these roles across three independent layers, interconnected via a high-speed SPI bus, forming a distributed embedded computing cluster.

Each node runs autonomously without mutual interference. The whole system is modular, replaceable, and scalable on demand.

---

## Architecture Layers

| Layer | Board | Role |
| :--- | :--- | :--- |
| **Command Layer** | Top Board | AI inference, strategy execution, Wi-Fi communication with local server |
| **Hub Layer** | Logic Board | Device discovery, task routing, peripheral management, SD card firmware repository |
| **Execution Layer** | Function Boards | Dedicated tasks: graphics rendering, audio processing, AI acceleration, etc. |

---

## Key Features

- **Three-Layer Physical Separation** — Independent boards for command, logic, and execution
- **SPI-Based Communication** — 20 MHz HD protocol with custom binary frame format
- **Automatic Device Discovery** — Any function board with a standard identity descriptor is automatically discovered and bound
- **Local Firmware Repository** — SD-card-backed storage for firmware versions, enabling offline re-flashing
- **Dual-File OTA Evolution** — Separate strategy files (Top Board) and firmware images (Logic/Function Boards)
- **A/B Partition Rollback** — Hardware watchdog triggers automatic fallback on upgrade failure
- **Hierarchical Permission Isolation** — MCP Server > Top Board > Logic Board > Function Board

---

## Communication Protocol Stack

**Physical Layer:** SPI Slave HD, 20 MHz, ~2.4 MB/s effective bandwidth

**Frame Format:**

```
[HEADER 2B] [TARGET_ID 1B] [FRAME_TYPE 1B] [CMD 1B] [LEN 2B] [PAYLOAD N] [CHECKSUM 1B]
```

| Field | Description |
| :--- | :--- |
| HEADER | Frame start marker (0xAA55) |
| TARGET_ID | Target board ID (0x00 = broadcast) |
| FRAME_TYPE | 0x01 = command / 0x02 = OTA block / 0x03 = SD storage / 0x04 = version query |
| CMD | Command code |
| LEN | Payload length |
| PAYLOAD | Data payload |
| CHECKSUM | XOR checksum |

---

## SD Card Firmware Repository

The logic board mounts a high-speed SD card (minimum 2 GB) as a local firmware repository:

```
SD Card Root
├── /system/                     # Logic board firmware
├── /devices/                    # Function board firmware library
│   ├── /renderer/               # Rendering board versions
│   ├── /audio/                  # Audio board versions
│   └── /ai_accel/               # AI accelerator board versions
├── /top_level/                  # Top board strategy files (stored only)
└── /ota_logs/                   # Upgrade operation logs
```

When a faulty function board is replaced, the logic board automatically detects the new device, queries the SD card for the latest stable firmware, and re-flashes it offline — no server or top board intervention required.

---

## OTA Upgrade Flow

1. Top board receives new firmware/strategy file from server via Wi-Fi
2. Top board forwards file to logic board
3. Logic board stores file in SD card repository, updates version index
4. Top board issues upgrade command at the appropriate time
5. Logic board reads firmware from SD card and blind-forwards via SPI to target function board
6. Function board writes to OTA backup partition and soft-resets
7. If heartbeat fails within 10 seconds, hardware watchdog triggers automatic rollback

---

## Current Status

| Component | Status |
| :--- | :--- |
| SPI Physical Layer (20 MHz) | ✅ Verified |
| Rendering Board Display Pipeline | ✅ Verified |
| CS_KEEP_ACTIVE Frame Boundary Control | ✅ Verified |
| Logic Board Device Discovery | 🔧 Under Construction |
| Routing Table & Driver Binding | 🔧 Under Construction |
| SD Card FATFS Repository | 🔧 Under Construction |
| OTA Dual-File Distribution | 📐 Designed |
| AI Inference Modules | 📐 Designed (Reserved) |
| MCP Server Integration | 📐 Designed (Reserved) |

---

## Repository Contents

- `architecture_specification.md` — Complete system architecture design document
- `README.md` — This file

---

## Citation

If you use this architecture design in your research or projects, please cite:

**APA:**
> NextGod2016. (2026). *ESP32-S3 Distributed Heterogeneous Computing Cluster - Architecture Design* (v1.0.1). Zenodo. https://doi.org/10.5281/zenodo.20974068

**BibTeX:**
```bibtex
@software{NextGod2016_esp32-cluster-architecture_2026,
  author = {NextGod2016},
  title = {ESP32-S3 Distributed Heterogeneous Computing Cluster - Architecture Design},
  month = jun,
  year = {2026},
  publisher = {Zenodo},
  version = {v1.0.1},
  doi = {10.5281/zenodo.20974068},
  url = {https://doi.org/10.5281/zenodo.20974068}
}
```

---

## License

This work is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

---

**DOI: [10.5281/zenodo.20974068](https://doi.org/10.5281/zenodo.20974068)**

---

*Part of the ESP32-S3 Distributed Heterogeneous Computing Cluster project.*
