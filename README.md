<div align="center">

<!-- Logo -->
<img src="docs/assets/logo.svg" alt="Agent AKI" width="140">

# Agent AKI

### The AI-Powered Arduino IDE

**One prompt. Complete hardware project. Camera-verified wiring.**

[![CI](https://github.com/jainapurva/agent-aki/actions/workflows/ci.yml/badge.svg)](https://github.com/jainapurva/agent-aki/actions/workflows/ci.yml)
[![License: AGPL v3](https://img.shields.io/badge/License-AGPL%20v3-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)
[![Arduino IDE](https://img.shields.io/badge/Built%20on-Arduino%20IDE%202.x-00979D?logo=arduino)](https://github.com/arduino/arduino-ide)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.4-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

<br>

*Think [Cursor](https://cursor.sh) for hardware. Fork of Arduino IDE 2.x with an AI agent built in.*

<br>

</div>

---

<!-- Demo Video -->
<div align="center">

### See it in action

https://github.com/user-attachments/assets/demo-placeholder

*Full demo: prompt → design → wiring → code → compile → camera verify → working hardware*

</div>

---

## The Problem

Every Arduino project starts the same way:

```
1. Google which components you need              ← 30 min
2. Find a wiring diagram (hope it's correct)     ← 20 min
3. Copy-paste code from StackOverflow            ← 15 min
4. Debug why the LCD shows garbage               ← 2 hours
5. Realize you wired SDA to the wrong pin        ← pain
```

## The Solution

```
You: "Build me a temperature sensor with an LCD display"

Agent AKI: Done. Here's what I did:
  ✓ Picked ESP32 (you need WiFi for logging)
  ✓ Selected DHT11 + SSD1306 OLED
  ✓ Validated: no pin conflicts, power OK
  ✓ Generated 12-step wiring guide
  ✓ Wrote complete Arduino sketch (47 lines)
  ✓ Compiled successfully
  ✓ Verified your breadboard via camera — all connections correct
  ✓ Uploaded and running. Serial shows: "Temp: 23.4°C, Humidity: 61%"
```

**Your only job is physical:** gather components, wire the breadboard, plug in USB.

---

## How It Works

<div align="center">

```
 ┌─────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
 │  DESIGN  │ ──▶ │ VALIDATE  │ ──▶ │   WIRE   │ ──▶ │   CODE   │
 │          │     │           │     │          │     │          │
 │ Pick board│     │ Pin check │     │ Step-by- │     │ Write +  │
 │ + parts  │     │ Power OK  │     │ step     │     │ compile  │
 └─────────┘     └──────────┘     └──────────┘     └────┬─────┘
                                                         │
 ┌─────────┐     ┌──────────┐     ┌──────────┐          │
 │  DONE!   │ ◀── │   TEST   │ ◀── │  UPLOAD  │ ◀────────┘
 │          │     │          │     │          │
 │ Working  │     │ Serial   │     │ Flash to │
 │ hardware │     │ verify   │     │ board    │
 └─────────┘     └──────────┘     └──────────┘

         ┌──────────┐
         │  CAMERA  │  ← Optional: verify wiring visually
         │ VERIFY   │
         └──────────┘
```

</div>

The agent handles **8 steps autonomously**. If compilation fails, it reads the error and fixes the code itself. You never touch the code.

---

## Features

<table>
<tr>
<td width="50%">

### 🧠 Hardware Intelligence
- **Auto board selection** — picks ESP32, XIAO ESP32S3, or Arduino Uno based on your project needs
- **Component recommendation** — knows 15+ components with specs, libraries, and pin requirements
- **Pin validation** — catches conflicts, strapping pin issues, power budget overruns, voltage mismatches
- **Wiring generator** — step-by-step breadboard instructions with wire colors

</td>
<td width="50%">

### 🤖 Autonomous Agent
- **12 tools** — design, validate, wire, code, compile, upload, serial, camera, and more
- **20-iteration loop** — keeps going until your hardware works
- **Self-healing** — if code doesn't compile, it fixes the error and retries
- **Never asks you to code** — the agent writes ALL the code

</td>
</tr>
<tr>
<td width="50%">

### 📷 Camera Verification
- Captures photos from your webcam
- AI vision analyzes your breadboard
- Reports: verified connections, missing wires, issues
- Catches mistakes **before** you power on

</td>
<td width="50%">

### 🛠 Full Arduino IDE
- Complete Arduino IDE 2.x — editor, board manager, library manager
- Serial monitor + plotter
- Compile, upload, debug
- Supports 1000+ Arduino-compatible boards

</td>
</tr>
</table>

---

## Quick Start

### Prerequisites

| Tool | Required | Install |
|------|----------|---------|
| Node.js 18-20 | Yes | `nvm install 20` |
| Yarn | Yes | `npm install -g yarn` |
| Python 3.x | Yes | For native modules |
| Claude Code CLI | Yes | [Install guide](https://docs.anthropic.com/en/docs/claude-code) |
| imagesnap | Optional | `brew install imagesnap` (camera on macOS) |

### Install & Run

```bash
# Clone
git clone https://github.com/jainapurva/agent-aki.git
cd agent-aki

# Install dependencies
yarn install

# Build
yarn build

# Launch
yarn start
```

### First Project

1. Launch Agent AKI — the AI panel opens on the right (`Ctrl+Shift+A`)
2. Select your board from the toolbar
3. Type your first prompt:

```
Build me a temperature and humidity monitor with a display
```

4. Follow the agent's wiring instructions
5. Watch it write, compile, upload, and verify — automatically

---

## Supported Hardware

### Boards

| Board | WiFi | Camera | Complexity | Price |
|-------|:----:|:------:|:----------:|:-----:|
| **Arduino Uno** | — | — | Beginner | ~$12 |
| **ESP32-WROOM-32** | ✅ | — | Intermediate | ~$8 |
| **XIAO ESP32S3 Sense** | ✅ | ✅ | Intermediate | ~$14 |

### Components (15 built-in)

<details>
<summary>Click to expand full component list</summary>

| Component | Type | Use Case |
|-----------|------|----------|
| SSD1306 OLED 128x64 | I2C Display | Small screen |
| LCD 16x2 I2C | I2C Display | Text display |
| DHT11 | Temperature | Basic temp/humidity |
| DHT22 | Temperature | Accurate temp/humidity |
| HC-SR04 | Ultrasonic | Distance/motion |
| SG90 Servo | Motor | Actuation/locks |
| MAX98357A | I2S Amp | Audio output |
| INMP441 | I2S Mic | Audio input |
| TTP223 | Touch | Capacitive touch |
| WS2812B NeoPixel | LED Strip | RGB lighting |
| Passive Buzzer | Sound | Alarms/tones |
| LED + Resistor | Light | Indicators |
| Push Button | Input | User input |
| Potentiometer 10K | Analog | Dials/knobs |
| Relay Module | Switch | High-power control |

</details>

> **Adding more is easy.** Drop a JSON entry into `components.json` — see [CONTRIBUTING.md](CONTRIBUTING.md).

---

## How It Compares

| Feature | ArduinoVision | Embedr | Cirkit Designer | **Agent AKI** |
|---------|:------------:|:------:|:---------------:|:-------------:|
| Full IDE | — | ✅ | — | ✅ |
| Board auto-selection | — | — | — | ✅ |
| Circuit design | — | — | ✅ (closed) | ✅ **(open)** |
| Pin validation | — | — | ✅ (closed) | ✅ **(open)** |
| Wiring instructions | — | — | — | ✅ |
| Camera verification | AVR only | — | — | **Multi-board** |
| ESP32 support | — | ✅ | Partial | ✅ |
| Open source | ✅ | — | — | ✅ **(AGPLv3)** |

---

## Architecture

```
┌───────────────────────────────────────────────────────────────┐
│                      Electron Shell                            │
├─────────────────────────────┬─────────────────────────────────┤
│     Frontend (React)        │        AI Agent Panel           │
│  ┌─────────────────────┐    │  ┌───────────────────────────┐  │
│  │ Code Editor          │    │  │ Chat UI (ReactWidget)     │  │
│  │ Board Selector       │    │  │ Streaming + Tool Calls    │  │
│  │ Library Manager      │    │  │ Welcome + Example Prompts │  │
│  │ Serial Monitor       │    │  └───────────────────────────┘  │
│  └─────────────────────┘    │                                  │
├─────────────────────────────┼─────────────────────────────────┤
│     Backend (Node.js)       │       Agent Service              │
│  ┌─────────────────────┐    │  ┌───────────────────────────┐  │
│  │ gRPC → arduino-cli   │    │  │ Claude CLI subprocess     │  │
│  │ Board discovery      │    │  │ 12-tool registry          │  │
│  │ Compile / Upload     │    │  │ Agentic loop (20 iter)    │  │
│  │ Serial I/O           │    │  │ Hardware knowledge base   │  │
│  └─────────────────────┘    │  └───────────────────────────┘  │
├─────────────────────────────┴─────────────────────────────────┤
│                  arduino-cli (bundled, gRPC)                    │
└───────────────────────────────────────────────────────────────┘
```

### Agent Tools

<details>
<summary>12 tools the agent can use autonomously</summary>

| Tool | Purpose |
|------|---------|
| `suggest_design` | Pick board + components + pins from a prompt |
| `validate_design` | Check for pin conflicts, power issues |
| `generate_wiring` | Step-by-step breadboard instructions |
| `compile` | Compile sketch via arduino-cli |
| `upload` | Flash sketch to board |
| `read_serial` | Read serial monitor output |
| `write_file` | Create/edit sketch files |
| `read_file` | Read sketch files |
| `list_files` | List sketch directory |
| `suggest_library` | Recommend Arduino libraries |
| `capture_photo` | Capture webcam image |
| `verify_wiring` | Vision-based wiring verification |

</details>

---

## Roadmap

- [x] AI agent chat panel with streaming
- [x] 7 core tools (compile, upload, serial, files, library)
- [x] Hardware knowledge base (3 boards, 15 components)
- [x] Pin validation engine
- [x] Design suggestion tool with board auto-selection
- [x] Wiring instruction generator
- [x] Camera capture + vision verification
- [x] Autonomous system prompt (4-phase workflow)
- [ ] More boards (Mega, Nano, Pi Pico, STM32)
- [ ] More components (BME280, MPU6050, GPS, RFID, steppers)
- [ ] Syntax highlighting in agent chat
- [ ] Diff view for AI-proposed code changes
- [ ] Datasheet intelligence (PDF upload → register maps)
- [ ] Circuit diagram generation (SVG schematic)
- [ ] Phone camera support (HTTP endpoint)
- [ ] Multi-board orchestration

---

## Contributing

We'd love your help! See [**CONTRIBUTING.md**](CONTRIBUTING.md) for how to get started.

**Easiest ways to contribute:**
- Add a board to [`boards.json`](arduino-ide-extension/src/node/agent/knowledge/data/boards.json)
- Add a component to [`components.json`](arduino-ide-extension/src/node/agent/knowledge/data/components.json)
- Pick up a [`good first issue`](https://github.com/jainapurva/agent-aki/labels/good%20first%20issue)

See our [Agent Architecture](AGENTS.md) for technical deep-dive.

---

## License

**GNU Affero General Public License v3.0** — see [LICENSE](LICENSE).

Fork of [Arduino IDE 2.x](https://github.com/arduino/arduino-ide) (also AGPLv3).

Copyright (C) 2026 [Apurva Jain](https://github.com/jainapurva) and contributors.

---

<div align="center">

**Built with frustration, caffeine, and too many wrong wiring diagrams.**

[GitHub](https://github.com/jainapurva/agent-aki) · [Issues](https://github.com/jainapurva/agent-aki/issues) · [Contributing](CONTRIBUTING.md) · [Agent Architecture](AGENTS.md)

</div>
