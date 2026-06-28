# ESP32-S3 Distributed Heterogeneous Embedded Computing Cluster
## A Four-Layer Self-Evolving Edge Architecture with Hierarchical AI and Localized OTA Infrastructure

> 📄 **Document Type**: Technical Paper (v1.0)
> 📅 **Date**: June 28, 2026
> 🚧 **Status**: System architecture design finalized. SPI physical layer & dual-board communication under verification. Logic board system & OTA mechanisms under development. AI-related modules are reserved as architectural headroom.

---

## Table of Contents
- [Abstract](#abstract)
- [Keywords](#keywords)
- [1. Introduction](#1-introduction)
  - [1.1 Problem Statement](#11-problem-statement)
  - [1.2 Design Goals](#12-design-goals)
  - [1.3 Architectural Overview](#13-architectural-overview)
- [2. System Architecture](#2-system-architecture)
  - [2.1 Cloud Brain Layer: Local Server](#21-cloud-brain-layer-local-server)
  - [2.2 Three-Layer Physical Separation of Embedded Cluster](#22-three-layer-physical-separation-of-embedded-cluster)
  - [2.3 Function Board Expansion Mechanism](#23-function-board-expansion-mechanism)
  - [2.4 Design Principles](#24-design-principles)
- [3. Inter-Chip Communication Protocol Stack](#3-inter-chip-communication-protocol-stack)
  - [3.1 Physical Layer](#31-physical-layer)
  - [3.2 Protocol Frame Format](#32-protocol-frame-format)
  - [3.3 Device Discovery Mechanism](#33-device-discovery-mechanism)
  - [3.4 Asynchronous Communication Model](#34-asynchronous-communication-model)
- [4. Logic Board System Design](#4-logic-board-system-design)
  - [4.1 Core Responsibilities](#41-core-responsibilities)
  - [4.2 Excluded Responsibilities](#42-excluded-responsibilities)
  - [4.3 Device Routing Table Structure](#43-device-routing-table-structure)
- [5. Storage System Design](#5-storage-system-design)
  - [5.1 Local Firmware Repository (SD Card)](#51-local-firmware-repository-sd-card)
  - [5.2 Version Index Table](#52-version-index-table)
- [6. OTA Upgrade Mechanism](#6-ota-upgrade-mechanism)
  - [6.1 Dual-File Evolution System](#61-dual-file-evolution-system)
  - [6.2 Upgrade Workflow](#62-upgrade-workflow)
  - [6.3 Offline Re-flashing](#63-offline-re-flashing)
  - [6.4 Rollback Mechanism](#64-rollback-mechanism)
- [7. Permission Isolation](#7-permission-isolation)
- [8. Current Validation Status](#8-current-validation-status)
- [9. Key Technical Constraints](#9-key-technical-constraints)
- [10. Reserved Architectural Headroom](#10-reserved-architectural-headroom)
- [License](#license)

---

## Abstract
This paper presents the architecture design of a distributed embedded computing system built around the ESP32-S3 platform. The system addresses the performance bottleneck of single-chip solutions where computation, scheduling, rendering, and peripheral management compete for CPU time, memory bandwidth, and bus resources.

The proposed architecture consists of four layers: Cloud Brain Layer (local server), Command Layer (Top Board), Hub Layer (Logic Board), and Execution Layer (Function Boards). The Cloud Brain Layer runs an MCP Host responsible for deep learning training, strategy generation, and dual evolution file compilation. The Command Layer executes strategy AI, aggregates work logs, and communicates with the local server via Wi-Fi. The Logic Board serves as the system backbone, responsible for device discovery, routing table maintenance, unified peripheral management, and SD-card-backed local firmware repository management. Function Boards are modular and hot-swappable; any board providing a standard identity descriptor is automatically discovered and bound.

The system incorporates a dual-file OTA evolution mechanism: strategy files for the Top Board and firmware images for Logic/Function Boards are distributed via Wi-Fi, stored in the SD card repository, and deployed asynchronously. Offline re-flashing of replacement Function Boards is supported without server involvement. A hierarchical permission chain (MCP Server > Top Board > Logic Board > Function Board) ensures strict privilege isolation.

The architecture reserves headroom for a distributed AI log loop, where Function Boards generate structured work logs for future server-side intelligent analysis and adaptive scheduling.

### Keywords
ESP32-S3, distributed system, heterogeneous computing, embedded architecture, OTA, edge computing, firmware repository, hierarchical AI

---

## 1. Introduction

### 1.1 Problem Statement
In embedded graphics and edge computing systems, a single chip is often required to simultaneously handle computation, scheduling, rendering, and peripheral management. These four roles contend for CPU time, memory bandwidth, and bus resources, resulting in degraded overall performance. Furthermore, modifications to any module can affect the stability of the entire system. This limitation becomes increasingly apparent as application complexity grows.

### 1.2 Design Goals
The proposed architecture separates the four roles onto independent chips, interconnected via a high-speed serial bus, forming a distributed embedded computing cluster. Each node runs autonomously without mutual interference, and the overall system is designed to be modular, replaceable, and scalable on demand.

### 1.3 Architectural Overview
The system adopts a four-layer architecture design:

| Layer | Board / Node | Role | Analogy |
| :--- | :--- | :--- | :--- |
| Cloud Brain Layer | Local Server (x64 Desktop, 24/7 Online) | MCP Host: deep learning training, strategy generation, dual evolution file compilation | Cloud intelligence / prefrontal cortex |
| Command Layer | Top Board (ESP32-S3 + Wi-Fi) | Run strategy AI (execute File A from MCP), aggregate work logs and forward to MCP server, receive and verify evolution files, decide upgrade timing | Decision center / cerebral cortex |
| Hub Layer | Logic Board (ESP32-S3 SPI Master + SD Card) | Bus enumeration, device discovery, task routing, unified peripheral management, local firmware repository | System bus / chipset + autonomic nervous system |
| Execution Layer | Function Boards (ESP32-S3 SPI Slaves) | Receive specific instructions, execute dedicated tasks (rendering, audio, AI acceleration, etc.), generate work logs | Dedicated accelerators / peripheral controllers |

---

## 2. System Architecture

### 2.1 Cloud Brain Layer: Local Server
The Cloud Brain Layer is an x64 desktop machine running 24/7 on the local network. It serves as the MCP (Model Context Protocol) Host and is responsible for:
- Aggregating work logs from all Function Boards via the Top Board
- Performing deep learning training and pattern recognition
- Generating optimal strategies and scheduling policies
- Compiling dual evolution files: File A (Top Board strategy) and File B (Function Board execution weights)
- Distributing evolution files to the Top Board via Wi-Fi

This layer is the only node in the system with the authority to upgrade the Top Board's strategy logic. It operates offline from the embedded cluster, decoupling heavy computation from real-time embedded operations.

### 2.2 Three-Layer Physical Separation of Embedded Cluster
The physical architecture of the embedded cluster (Top Board, Logic Board, Function Boards) is illustrated below:

```text
┌─────────────────────────────────────────────────────────────────────────────────┐
│ Local Server (x64 Desktop, 24/7 Online)                                       │
│ MCP Host: Deep Learning Training + Strategy Generation                        │
│ + Dual Evolution File Compilation                                             │
└─────────────────────────────────────────────────────────────────────────────────┘
                                    │ Wi-Fi (MCP Protocol)
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│ Top Board (Command Layer) ESP32-S3 + Wi-Fi                                    │
│ Responsibilities:                                                             │
│ · Run strategy AI (execute "File A: Top Strategy" from MCP)                   │
│ · Aggregate all function board work logs → forward to MCP server              │
│ · Receive dual evolution files from MCP → verify → distribute to Logic Board │
│   for SD card storage                                                         │
│ · Decide "when to upgrade which board" (strategy decision authority)          │
└─────────────────────────────────────────────────────────────────────────────────┘
                                    │ SPI HD (Command + OTA Data Transfer)
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│ Logic Board (Hub Layer) ESP32-S3 (SPI Master) + High-Speed SD Card            │
│ (4GB+, as "Hard Drive")                                                       │
│ Responsibilities:                                                             │
│ · Bus enumeration and device discovery (multi-board auto-identification       │
│   and binding)                                                                │
│ · Driver loading and task routing (maintain device routing table)             │
│ · Scheduling AI (monitor per-channel bandwidth, dynamically adjust priority)  │
│ · Unified peripheral management (SD card/buttons/audio/power)                 │
│ · Version repository manager: maintain "firmware version library" on SD card  │
│ · Offline re-flash: when replacing a function board, directly fetch           │
│   corresponding firmware from SD card and re-flash                            │
│ · Self-upgrade: read own firmware from SD card → write to OTA partition →     │
│   reboot to apply                                                             │
│ · [FORBIDDEN] Cannot modify Top Board firmware (permission isolation)         │
└─────────────────────────────────────────────────────────────────────────────────┘
          │ SPI Bus (CS1)        │ SPI Bus (CS2)        │ SPI Bus (CS3)        │ SPI Bus (CS4)
          ▼                      ▼                      ▼                      ▼
┌──────────────┐        ┌──────────────┐        ┌──────────────┐        ┌──────────────┐
│  Function    │        │  Function    │        │  Function    │        │  Function    │
│  Board A     │        │  Board B     │        │  Board C     │        │  Board D     │
│  (Rendering) │        │  (Audio)     │        │  (AI Accel.) │        │  (Reserved)  │
│              │        │              │        │              │        │              │
│ ·Execute AI  │        │ ·Execute AI  │        │ ·Execute AI  │        │              │
│ ·Standalone  │        │ ·DSP         │        │ ·NPU         │        │              │
│  pipeline    │        │  processing  │        │  acceleration│        │              │
│ ·Generate    │        │ ·Generate    │        │ ·Generate    │        │              │
│  work logs   │        │  audio logs  │        │  compute logs│        │              │
└──────────────┘        └──────────────┘        └──────────────┘        └──────────────┘
```

### 2.3 Function Board Expansion Mechanism
The third layer is an open function board plane, allowing different execution boards to be attached according to product requirements:
- **Rendering Board**: graphics rendering and display output
- **Audio Board**: audio codec and DSP processing
- **AI Accelerator Board**: neural network inference acceleration
- **Sensor Aggregation Board**: multi-channel sensor data acquisition and preprocessing
- Other custom-function boards as needed

The system supports multiple function boards of the same type running in standby (e.g., primary/backup rendering boards for hot-swap), all scheduled by the Logic Board.

### 2.4 Design Principles
- The Top Board is unaware of Function Boards; it communicates only with the Logic Board
- Function Boards are unaware of each other; they only respond to bus commands
- The Logic Board is the sole hub, but it does not parse business content—it only routes and schedules
- Any Function Board that provides a standard identity descriptor can be automatically discovered and bound
- The local server operates outside the embedded cluster; it does not directly control any board and only communicates with the Top Board

---

## 3. Inter-Chip Communication Protocol Stack

### 3.1 Physical Layer
| Item | Specification |
| :--- | :--- |
| Interface | SPI Slave HD (half-duplex), 1-bit data line |
| Clock frequency | 20 MHz |
| Effective bandwidth | ~2.4 MB/s |
| Topology | One master (Logic Board) with multiple slaves (Function Boards), individual CS pins for addressing |
| Max DMA per transaction | 4092 bytes |
| DMA restriction | Can only write to internal SRAM; PSRAM direct access is not supported |

### 3.2 Protocol Frame Format
Frame structure definition:
```
[HEADER 2B] [TARGET_ID 1B] [FRAME_TYPE 1B] [CMD 1B] [LEN 2B] [PAYLOAD N] [CHECKSUM 1B]
```

| Field | Description |
| :--- | :--- |
| HEADER | Frame start marker (`0xAA55`) |
| TARGET_ID | Target board ID (`0x00` = broadcast, `0x01`~`0xFE` = specific board) |
| FRAME_TYPE | `0x01` = real-time command / `0x02` = OTA data block / `0x03` = SD card storage / `0x04` = version query |
| CMD | Command code (valid for real-time commands) |
| LEN | Payload length (big-endian) |
| PAYLOAD | Data payload |
| CHECKSUM | XOR checksum |

### 3.3 Device Discovery Mechanism
1. After power-on, the Logic Board sends broadcast query commands sequentially over each CS pin
2. A Function Board (slave) replies with an **identity descriptor** (fixed 64-byte format) containing:
   - Device type (rendering, audio, AI accelerator, etc.)
   - Hardware version
   - Capability bitmap
   - Unique ID (last 6 bytes of chip MAC address)
   - Current firmware version
3. The Logic Board parses the descriptor, registers the device in its routing table, and loads the corresponding driver

### 3.4 Asynchronous Communication Model
Because SPI is synchronous, the slave must return data over MISO whenever the clock runs. To achieve application-level asynchronous semantics, the system uses the following strategy:
- The Logic Board does not wait for replies after sending command frames
- The Logic Board periodically inserts "dummy read transactions" (sending null commands) to fetch pending log data from Function Boards
- Function Boards store outgoing data in their MISO transmit register, waiting for the next read cycle

---

## 4. Logic Board System Design

### 4.1 Core Responsibilities
- Bus enumeration and device discovery
- Device routing table maintenance and dynamic driver binding
- Command routing and distribution
- Unified peripheral management (SD card file system, key scanning, I2S audio, power monitoring)
- Exception handling and degradation strategies (timeout retry, primary/backup switching)

### 4.2 Excluded Responsibilities
- Does not emulate CPU/PPU (that is the Top Board's role)
- Does not process graphics, audio, or inference algorithms (that is the Function Boards' role)
- Does not block waiting for external acknowledgements

### 4.3 Device Routing Table Structure
The Logic Board maintains up to 4 device slots. Each slot records:

| Field | Description |
| :--- | :--- |
| TARGET_ID | Device ID (1~4) |
| Device type | Rendering, audio, AI accelerator, etc. |
| CS pin | Corresponding GPIO number |
| Status | Empty / Active / Offline / OTA in progress |
| Unique ID | Chip unique identifier |
| Current firmware version | For version validation |
| Driver function pointer | Business dispatch entry |
| Scheduling priority | Dynamically adjustable |

---

## 5. Storage System Design

### 5.1 Local Firmware Repository (SD Card)
The Logic Board mounts a high-speed SD card (minimum 2 GB) as the system's local firmware repository. The directory structure is as follows:

```text
SD Card Root (/)
├── /system/
│   ├── logic_firmware_v2.1.0.bin    # Logic board own firmware
│   ├── logic_firmware_v2.0.9.bin    # Historical versions (for rollback)
│   └── rollback_flag.txt            # Rollback flag (read when watchdog triggers)
│
├── /devices/
│   ├── /renderer/
│   │   ├── v3.2.0.bin               # Rendering board latest
│   │   ├── v3.1.5.bin               # Rendering board stable historical
│   │   └── device_caps.json         # Known capability description for this device type
│   ├── /audio/
│   │   ├── v2.0.3.bin
│   │   └── v1.9.8.bin
│   ├── /ai_accel/
│   │   └── v1.0.1.bin
│   └── /unknown/                    # Newly inserted unknown devices, staging area
│
├── /top_level/
│   └── top_strategy_v5.bin          # Top Board strategy file (Logic Board only stores, no write authority to board)
│
└── /ota_logs/
    ├── upgrade_history.log          # All upgrade operation records
    └── fallback_events.log          # Rollback event records
```

### 5.2 Version Index Table
The Logic Board maintains an in-memory version index table that records all available firmware versions for each device type on the SD card, along with their checksums. It supports:
- Querying the latest stable version by device type
- Exact version lookup by version number
- CRC32 integrity verification

```c
typedef struct {
    uint8_t device_type;        // Device type
    uint8_t major, minor, patch; // Version number
    uint32_t file_offset;       // Position in SD card FAT table
    uint32_t file_size;
    uint32_t crc32;             // Firmware checksum
    uint8_t is_stable;          // 1=stable, 0=experimental
} version_entry_t;
```

---

## 6. OTA Upgrade Mechanism

### 6.1 Dual-File Evolution System
The system supports two types of upgrade files:

| File | Target | Content | Storage & Distribution Path |
| :--- | :--- | :--- | :--- |
| File A (Top Strategy) | Top Board | ESP-DL model weights + policy rules | MCP → Top Board direct application (not via Logic Board) |
| File B (Firmware Image) | Logic Board / Function Boards | Complete firmware binary | MCP → Top Board → Logic Board → SD card repository |

### 6.2 Upgrade Workflow
1. The Top Board receives new firmware/strategy file from the server via Wi-Fi
2. After verifying integrity, the Top Board sends the file to the Logic Board
3. The Logic Board writes the file to the appropriate SD card directory and updates the version index
4. At a suitable time (according to its running strategy), the Top Board issues an upgrade command to the Logic Board
5. The Logic Board reads the firmware from the SD card and blind-forwards it (in blocks) via SPI to the target Function Board
6. The Function Board receives all blocks → writes to the OTA backup partition → soft-resets and activates the new firmware

### 6.3 Offline Re-flashing
When a faulty Function Board is replaced, the new board is plugged into the bus and provides its identity descriptor:
1. The Logic Board enumerates the new device and reads its firmware version
2. It automatically queries the SD card for the latest stable version for that device type
3. It automatically initiates the re-flashing process, without requiring server or Top Board intervention
4. After successful programming, the new board registers and comes online

### 6.4 Rollback Mechanism
All boards (including the Logic Board itself) use an A/B partition OTA strategy:
- New firmware is written to the backup partition
- If the board does not resume normal heartbeat within 10 seconds after loading, the hardware watchdog triggers
- The boot flag is automatically redirected to the primary partition, rolling the system back to a safe state

---

## 7. Permission Isolation
The system enforces a strict hierarchical permission chain:
> **MCP Server > Top Board > Logic Board > Function Board**

| Operation | Permission Owner |
| :--- | :--- |
| Upgrade Top Board | MCP server delivers directly; Top Board applies itself |
| Order Logic Board upgrade | Top Board issues command; Logic Board reads from SD and executes |
| Offline re-flash Function Board | Logic Board executes automatically (based on local SD repository) |
| Function Board self-upgrade | Function Board receives OTA blocks and writes to its own backup partition |
| Logic Board modifying Top Board | Strictly forbidden (no physical write path) |

Although the SD card stores strategy files for the Top Board, the Logic Board firmware does not contain any instruction sequence that writes to the Top Board's flash – thus preventing privilege escalation at the physical layer.

---

## 8. Current Validation Status

🔄 **Under Verification**
- SPI HD two-board communication

🔧 **Under Construction**
- Logic Board device discovery and routing table maintenance
- SD Card FATFS file system driver and repository management
- `FRAME_TYPE` classification and handling framework

⛔ **Designed but Not Yet Implemented**
- Logic Board dynamic scheduling priority adjustment
- Top Board Wi-Fi MCP communication protocol
- Full OTA dual-file distribution and atomic rollback chain

---

## 9. Key Technical Constraints

| Constraint | Limitation | Mitigation |
| :--- | :--- | :--- |
| DMA transfer limit | 4092 bytes per transaction | Application-level chunking (1 KB per block) |
| DMA memory restriction | Only internal SRAM writable | Reassemble large data in SRAM in chunks |
| SPI slave mode | Quad mode not supported | Use standard SPI HD (1-bit) |
| Bus load | ≤ 4 slaves recommended at 20 MHz | 4 CS pins, support up to 4 Function Boards |
| 40 MHz unavailable | Signal integrity issue with jumper wires | Lock at 20 MHz; re-evaluate on PCB |
| SD card write latency | Large file writes may block interrupt response | Configure SD card in SPI mode + DMA transfer, place write operations in low-priority background tasks |

---

## 10. Reserved Architectural Headroom
The system reserves extension interfaces in the following areas (currently not implemented):

1. **Distributed AI Log Loop**
   - Each Function Board may deploy lightweight inference models to generate structured work logs
   - The Logic Board aggregates and forwards them via the Top Board
   - This provides a data channel for future server-side intelligent analysis

2. **Local Server Interaction**
   - The Top Board already reserves a Wi-Fi communication interface
   - Bidirectional MCP communication with a local server can be established later

3. **Online Scheduling Policy Adjustment**
   - The Logic Board's scheduling priorities accept dynamic configuration from the Top Board
   - This enables more complex adaptive scheduling algorithms in the future

Detailed implementation of the above modules will be defined in subsequent iterations based on requirements.

---

## License
This project documentation is released under the **MIT License**.
You are free to use, modify, and distribute the architecture design with attribution.

---
*Document Version: v1.0 | Last Updated: June 28, 2026*