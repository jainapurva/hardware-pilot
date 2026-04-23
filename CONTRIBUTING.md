# Contributing to Hardware Pilot

Thank you for your interest in contributing to Hardware Pilot! This guide will help you get started.

## Table of Contents

- [Development Setup](#development-setup)
- [Project Structure](#project-structure)
- [How to Contribute](#how-to-contribute)
  - [Adding a New Board](#adding-a-new-board)
  - [Adding a New Component](#adding-a-new-component)
  - [Adding a New Agent Tool](#adding-a-new-agent-tool)
- [Code Style](#code-style)
- [Pull Request Process](#pull-request-process)
- [Issue Guidelines](#issue-guidelines)

---

## Development Setup

### Prerequisites

- Node.js 18+
- Yarn 1.x (`npm install -g yarn`)
- Python 3.x (required for native module compilation)
- Git

### Build from Source

```bash
git clone https://github.com/jainapurva/hardware-pilot.git
cd hardware-pilot
yarn install       # 5-10 min first time
yarn build         # 3-5 min
yarn start         # Launch the Electron app
```

### Development Mode

```bash
yarn watch         # Watch mode with hot reload
```

### Run Tests

```bash
yarn test          # Run all tests
yarn lint          # Run linter
yarn lint:fix      # Auto-fix lint issues
```

---

## Project Structure

```
arduino-ide-extension/src/
├── browser/                    # Frontend (React + Theia)
│   ├── agent/                  # AI Agent panel UI
│   └── arduino-ide-frontend-module.ts  # Frontend DI bindings
├── node/                       # Backend (Node.js)
│   ├── agent/                  # Agent service + tools
│   │   ├── knowledge/          # Hardware knowledge base
│   │   │   ├── data/           # JSON data files
│   │   │   │   ├── boards.json
│   │   │   │   └── components.json
│   │   │   ├── boards.ts
│   │   │   ├── components.ts
│   │   │   └── pins.ts
│   │   └── tools/              # Agent tool implementations
│   └── arduino-ide-backend-module.ts   # Backend DI bindings
└── common/protocol/            # Shared RPC interfaces
```

---

## How to Contribute

### Adding a New Board

This is one of the easiest ways to contribute.

1. Open `arduino-ide-extension/src/node/agent/knowledge/data/boards.json`
2. Add a new entry following the existing format:

```json
{
  "id": "arduino_mega",
  "name": "Arduino Mega 2560",
  "fqbn": "arduino:avr:mega",
  "voltage": 5.0,
  "features": [],
  "price": 15,
  "complexity": "beginner",
  "max3v3CurrentMa": 50,
  "strappingPins": [],
  "inputOnlyPins": [],
  "i2cBuses": 1,
  "i2sPorts": 0,
  "spiBuses": 1,
  "pins": {
    "GPIO0": { "type": "io", "buses": ["uart_rx"], "pwm": false, "adc": false, "notes": "Serial RX" },
    "...": "..."
  }
}
```

3. Add a test case in `boards.test.ts`
4. Run `yarn test` to verify
5. Submit a PR

**Key fields to get right:**
- `fqbn` — the Fully Qualified Board Name used by arduino-cli (find yours with `arduino-cli board listall`)
- `strappingPins` — GPIOs that affect boot behavior (ESP32-specific, empty for AVR boards)
- `inputOnlyPins` — GPIOs that cannot be used as outputs
- `features` — capabilities like `wifi`, `bluetooth`, `camera`, `sd_card`, `built_in_mic`

### Adding a New Component

1. Open `arduino-ide-extension/src/node/agent/knowledge/data/components.json`
2. Add a new entry:

```json
{
  "id": "bme280",
  "name": "BME280 Temperature/Pressure/Humidity Sensor",
  "bus": "i2c",
  "busDirection": "input",
  "voltage": "3.3V-5V",
  "currentDrawMa": 3.6,
  "pinsNeeded": ["SDA", "SCL", "VCC", "GND"],
  "defaultAddress": "0x76",
  "libraries": ["Adafruit BME280 Library", "Adafruit Unified Sensor"],
  "notes": "More accurate than DHT11/22. Some boards use address 0x77."
}
```

3. Add a test case in `components.test.ts`
4. Run `yarn test` to verify
5. Submit a PR

**Key fields:**
- `bus` — one of: `i2c`, `i2s`, `spi`, `digital`, `analog`, `pwm`, `single_wire`
- `busDirection` — `input`, `output`, or `bidirectional`
- `pinsNeeded` — the pin roles this component requires (not specific GPIO numbers)
- `libraries` — exact Arduino library names as they appear in Library Manager

### Adding a New Agent Tool

This is a medium-difficulty contribution.

1. Create a new file in `arduino-ide-extension/src/node/agent/tools/`
2. Implement the `AgentTool` interface:

```typescript
import { AgentTool, ToolResult } from '../agent-tools';
import { AgentContext } from '../../../common/protocol/agent-service';

export function makeMyTool(): AgentTool {
  return {
    name: 'my_tool',
    description: 'What this tool does — be specific, the LLM reads this.',
    parameters: {
      param1: { type: 'string', description: 'What this param is for' },
    },
    required: ['param1'],
    async execute(params: Record<string, unknown>, context: AgentContext): Promise<ToolResult> {
      // Your implementation here
      return { success: true, output: 'Result text' };
    },
  };
}
```

3. Register it in `agent-tools.ts`:

```typescript
import { makeMyTool } from './tools/my-tool';

// In registerBuiltins():
this.register(makeMyTool());
```

4. Add tests
5. Submit a PR

---

## Code Style

This project follows the existing Arduino IDE code conventions:

- **TypeScript** for all source code
- **2-space indentation**
- **Single quotes** for strings
- **Semicolons** required
- **No unused variables** (prefix with `_` if intentionally unused)
- **Explicit return types** on public functions

We use ESLint to enforce style. Run before submitting:

```bash
yarn lint          # Check for issues
yarn lint:fix      # Auto-fix what's possible
```

### Naming Conventions

- Files: `kebab-case.ts` (e.g., `design-tool.ts`)
- Classes: `PascalCase` (e.g., `AgentServiceImpl`)
- Functions: `camelCase` (e.g., `assignPins`)
- Constants: `UPPER_SNAKE_CASE` (e.g., `MAX_ITERATIONS`)
- Interfaces: `PascalCase` with `I` prefix only for DI tokens (e.g., `BoardSpec`, not `IBoardSpec`)
- JSON data IDs: `snake_case` (e.g., `esp32_wroom_32`)

---

## Pull Request Process

1. **Fork** the repository
2. **Create a branch** from `feat/agentic-core`: `git checkout -b feature/my-feature`
3. **Make your changes** with tests
4. **Run checks**: `yarn lint && yarn build && yarn test`
5. **Commit** with a clear message:
   - `feat: add BME280 to component catalog`
   - `fix: pin validation false positive on ESP32 GPIO 33`
   - `docs: add Raspberry Pi Pico wiring example`
6. **Push** and open a PR against `feat/agentic-core`

### PR Requirements

- All CI checks must pass (lint, build, test)
- New features must include tests
- New boards/components must include JSON data + test cases
- Description should explain what and why, not just what changed

### Review Process

- PRs are reviewed by maintainers
- We aim to review within 48 hours
- Small, focused PRs are reviewed faster than large ones

---

## Issue Guidelines

### Bug Reports

Use the [bug report template](.github/ISSUE_TEMPLATE/bug_report.md). Include:
- Steps to reproduce
- Expected vs actual behavior
- Your OS, board, and Hardware Pilot version

### Feature Requests

Use the [feature request template](.github/ISSUE_TEMPLATE/feature_request.md). Include:
- What problem it solves
- Your proposed solution
- Any alternatives you considered

### Good First Issues

Look for issues labeled [`good first issue`](https://github.com/jainapurva/hardware-pilot/labels/good%20first%20issue). These are specifically scoped for new contributors:
- Adding a board or component to the knowledge base
- Improving error messages
- Adding test cases
- Small UI improvements

---

## Code of Conduct

We follow the [Contributor Covenant Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/). Be kind, be respectful, and help each other build something great.

---

## Questions?

- Open a [Discussion](https://github.com/jainapurva/hardware-pilot/discussions) for questions
- File an [Issue](https://github.com/jainapurva/hardware-pilot/issues) for bugs or feature requests
- Tag `@jainapurva` for maintainer attention
