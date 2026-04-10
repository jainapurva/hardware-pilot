# Agent AKI

**An AI-powered Arduino IDE that takes you from a prompt to verified, working hardware.**

> "Build me a motion-activated alarm" — that's all you say. Agent AKI figures out the board, components, circuit, code, and even verifies your wiring with a camera.

[![CI](https://github.com/jainapurva/agent-aki/actions/workflows/ci.yml/badge.svg)](https://github.com/jainapurva/agent-aki/actions/workflows/ci.yml)
[![License: AGPL v3](https://img.shields.io/badge/License-AGPL%20v3-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)

<!-- ![Demo](docs/demo.gif) -->

---

## What is Agent AKI?

Agent AKI is a fork of [Arduino IDE 2.x](https://github.com/arduino/arduino-ide) with a built-in AI agent that handles the entire hardware development workflow. Think **Cursor, but for Arduino**.

You give it one prompt. It does everything else:

| Step | What happens | Who does it |
|------|-------------|-------------|
| 1. Design | Picks the best board, selects components, assigns pins | **Agent** |
| 2. Validate | Checks for pin conflicts, power issues, voltage mismatches | **Agent** |
| 3. Wiring | Generates step-by-step breadboard wiring instructions | **Agent** |
| 4. Code | Writes complete Arduino sketch | **Agent** |
| 5. Compile | Builds via arduino-cli, fixes errors automatically | **Agent** |
| 6. Guide Assembly | Walks you through wiring one component at a time | Agent guides, **you wire** |
| 7. Verify | Uses your camera to check breadboard matches the design | **Agent** |
| 8. Test | Uploads code, reads serial output, confirms it works | **Agent** |

**Your only job is physical**: gather components, wire the breadboard, plug in USB.

---

## Features

### AI Agent (Chat Panel)
- Built-in chat panel (right sidebar, `Ctrl+Shift+A`)
- Fully autonomous: designs, codes, compiles, uploads, and tests
- 12-tool agentic loop with up to 20 iterations per task
- Streaming responses with tool call visualization

### Hardware Intelligence
- **Board auto-selection** — picks the best board for your project (ESP32, XIAO ESP32S3, Arduino Uno)
- **Component catalog** — knows 15+ common components with specs, pin requirements, and libraries
- **Pin validation engine** — catches conflicts, strapping pin issues, power budget overruns, voltage mismatches
- **Wiring instruction generator** — step-by-step breadboard assembly with wire colors

### Camera Verification
- Captures photos from your webcam
- Uses Claude Vision to verify breadboard connections match the expected design
- Reports verified connections, missing wires, and issues with confidence score

### Full Arduino IDE
- All Arduino IDE 2.x features: code editor, board manager, library manager, serial monitor, serial plotter
- Compile, upload, and debug — all built in
- Supports 1000+ Arduino-compatible boards

---

## Quick Start

### Prerequisites

- **Node.js** 18+ ([download](https://nodejs.org/))
- **Yarn** 1.x (`npm install -g yarn`)
- **Python** 3.x (for native module compilation)
- **Git**
- **Claude Code** CLI installed (`~/.local/bin/claude`) — the agent backend
- **imagesnap** (macOS, optional for camera): `brew install imagesnap`

### Build & Run

```bash
git clone https://github.com/jainapurva/agent-aki.git
cd agent-aki
yarn install
yarn build
yarn start
```

### First Use

1. Launch the app — the AI Agent panel opens on the right
2. Select your board from the toolbar dropdown
3. Type a prompt: **"Build me a temperature sensor with an LCD display"**
4. Follow the agent's instructions — it handles the rest

---

## Supported Boards

| Board | Features | Complexity |
|-------|----------|------------|
| **Arduino Uno** | Basic GPIO, PWM, analog | Beginner |
| **ESP32-WROOM-32** | WiFi, Bluetooth, dual-core, 34 GPIO | Intermediate |
| **XIAO ESP32S3 Sense** | WiFi, Bluetooth, camera, mic, SD card | Intermediate |

More boards can be added by contributing to `boards.json` — see [CONTRIBUTING.md](CONTRIBUTING.md).

---

## Component Catalog

Agent AKI knows these components out of the box:

| Component | Bus | Use Case |
|-----------|-----|----------|
| SSD1306 OLED 128x64 | I2C | Small display |
| LCD 16x2 I2C | I2C | Text display |
| DHT11 / DHT22 | Single-wire | Temperature & humidity |
| HC-SR04 Ultrasonic | Digital | Distance / motion |
| SG90 Servo | PWM | Actuation |
| MAX98357A Amp | I2S | Audio output |
| INMP441 Mic | I2S | Audio input |
| TTP223 Touch | Digital | Touch input |
| WS2812B NeoPixel | Single-wire | RGB LED strip |
| Passive Buzzer | PWM | Sound / alarm |
| LED | Digital | Light indicator |
| Push Button | Digital | User input |
| Potentiometer | Analog | Dial / knob |
| Relay Module | Digital | High-power switching |

Add more by contributing to `components.json` — see [CONTRIBUTING.md](CONTRIBUTING.md).

---

## How It Compares

| Feature | ArduinoVision | Embedr | Cirkit Designer | **Agent AKI** |
|---------|--------------|--------|-----------------|---------------|
| Full IDE | No | Yes | No | **Yes** |
| Board auto-selection | No | No | No | **Yes** |
| Design suggestion | No | No | Yes (closed) | **Yes (open)** |
| Pin validation | No | No | Yes (closed) | **Yes (open)** |
| Wiring instructions | No | No | No | **Yes** |
| Camera verification | AVR only | No | No | **Multi-board** |
| ESP32/S3 support | No | Yes | Partial | **Yes** |
| Open source | Yes | No | No | **Yes (AGPLv3)** |
| One-click install | No | Yes | Yes | **Yes** |

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Electron Shell                         │
├──────────────────────┬──────────────────────────────────┤
│   Frontend (React)   │        AI Agent Panel            │
│   - Code Editor      │   - Chat UI (ReactWidget)       │
│   - Board Selector   │   - Streaming responses         │
│   - Library Manager  │   - Tool call visualization     │
│   - Serial Monitor   │   - Welcome screen + examples   │
├──────────────────────┴──────────────────────────────────┤
│                  Theia IDE Framework                      │
├──────────────────────┬──────────────────────────────────┤
│   Backend (Node.js)  │      Agent Service               │
│   - gRPC → CLI       │   - Claude CLI subprocess       │
│   - Board discovery  │   - 12-tool registry            │
│   - Compile/Upload   │   - Agentic loop (20 iter max)  │
│   - Serial monitor   │   - Hardware knowledge base     │
├──────────────────────┴──────────────────────────────────┤
│               arduino-cli (bundled, gRPC)                │
└─────────────────────────────────────────────────────────┘
```

### Agent Tools

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

---

## Project Structure

```
agent-aki/
├── arduino-ide-extension/src/
│   ├── browser/agent/              # Frontend: chat panel UI
│   │   ├── agent-panel-widget.tsx  # React chat interface
│   │   └── agent-view-contribution.ts
│   ├── node/agent/                 # Backend: agent logic
│   │   ├── agent-service-impl.ts   # Agentic loop orchestrator
│   │   ├── agent-tools.ts          # Tool registry (12 tools)
│   │   ├── claude-client.ts        # Claude CLI wrapper
│   │   ├── tools/                  # Hardware-specific tools
│   │   │   ├── design-tool.ts      # suggest_design
│   │   │   ├── validate-tool.ts    # validate_design
│   │   │   ├── wiring-tool.ts      # generate_wiring
│   │   │   ├── camera-tool.ts      # capture_photo
│   │   │   └── verify-tool.ts      # verify_wiring
│   │   └── knowledge/              # Hardware knowledge base
│   │       ├── boards.ts           # Board specs + auto-selection
│   │       ├── components.ts       # Component catalog + recommendation
│   │       ├── pins.ts             # Pin validation engine
│   │       └── data/
│   │           ├── boards.json     # Board definitions
│   │           └── components.json # Component definitions
│   └── common/protocol/
│       └── agent-service.ts        # RPC interface
├── electron-app/                   # Electron entry point
├── .github/workflows/ci.yml       # CI pipeline
├── CONTRIBUTING.md                 # How to contribute
├── AGENTS.md                       # Agent architecture docs
├── LICENSE                         # AGPLv3
└── README.md                       # This file
```

---

## Roadmap

### v0.1.0 (Launch)
- [x] AI agent chat panel
- [x] 7 core tools (compile, upload, serial, file ops, library suggest)
- [ ] Hardware knowledge base (boards, components)
- [ ] Pin validation engine
- [ ] Design suggestion tool
- [ ] Wiring instruction generator
- [ ] Camera verification

### v0.2.0
- [ ] More boards (Arduino Mega, Nano, Raspberry Pi Pico)
- [ ] More components (BME280, MPU6050, GPS, RFID, stepper motors)
- [ ] Syntax highlighting in agent chat code blocks
- [ ] Diff view for AI-proposed code changes

### v0.3.0
- [ ] Datasheet intelligence (upload PDF, extract register maps)
- [ ] Circuit diagram generation (SVG schematic from design)
- [ ] Phone camera support (HTTP endpoint for mobile photos)
- [ ] FreeRTOS deadlock detection
- [ ] ARM HardFault decoder

### Future
- [ ] Real-time video stream analysis
- [ ] Multi-board orchestration (distributed IoT systems)
- [ ] PCB layout generation
- [ ] Community component library (user-contributed specs)

---

## Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for how to get started.

Good first issues are labeled [`good first issue`](https://github.com/jainapurva/agent-aki/labels/good%20first%20issue).

---

## License

This project is licensed under the **GNU Affero General Public License v3.0** — see [LICENSE](LICENSE) for details.

This is a fork of [Arduino IDE 2.x](https://github.com/arduino/arduino-ide), which is also licensed under AGPLv3.

Copyright (C) 2026 Apurva Jain and contributors.
Based on Arduino IDE, Copyright (C) Arduino SA.

---

## Acknowledgements

- [Arduino IDE 2.x](https://github.com/arduino/arduino-ide) — the foundation this project is built on
- [Eclipse Theia](https://theia-ide.org/) — the IDE framework
- [Anthropic Claude](https://www.anthropic.com/) — the AI that powers the agent
- The Arduino and ESP32 communities
