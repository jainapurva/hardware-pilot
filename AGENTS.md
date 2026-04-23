# Agent Architecture — AGENTS.md

This document describes the AI agent system inside Hardware Pilot: how it works, how tools are structured, and how the autonomous workflow operates.

---

## Overview

Hardware Pilot embeds an AI agent directly into the Arduino IDE. The agent uses Claude (via the `claude` CLI) as its reasoning engine and has access to 12 tools that let it interact with the IDE, the filesystem, the compiler, the serial port, and the user's camera.

The agent operates in an **autonomous loop**: it receives a user prompt, makes decisions, calls tools, observes results, and iterates — up to 20 times per task — until the job is done.

---

## Agent Loop

```
User prompt
    │
    ▼
┌─────────────────────┐
│  Build System Prompt │ ← IDE context (board, sketch, serial, build output)
│  + conversation      │
│  + tool definitions  │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│   Call Claude CLI    │ ← `claude -p --output-format stream-json`
│   (stream response)  │
└─────────┬───────────┘
          │
          ├── Text response → display to user, done
          │
          └── Tool call → execute tool
                │
                ▼
          ┌───────────┐
          │ Tool runs  │ (compile, write_file, capture_photo, etc.)
          └─────┬─────┘
                │
                ▼
          Append result to conversation
                │
                ▼
          Loop back to Claude (up to 20 iterations)
```

### Key Design Decisions

1. **Claude CLI, not API** — Uses `claude -p` subprocess, not API keys. Runs on the user's existing Claude Code subscription. No API key management needed.

2. **Stateless tools** — Each tool call is independent. Context (board, sketch, serial) is rebuilt from IDE state on every `chat()` call.

3. **20-iteration limit** — Prevents infinite loops. The full autonomous workflow (design → validate → wiring → code → compile → fix → upload → test) typically needs 10-15 iterations.

4. **Backend-driven** — The agent loop runs in the Node.js backend, not the browser. The frontend only renders messages and tool calls streamed via JSON-RPC.

---

## System Prompt

The system prompt is dynamically built from live IDE state on every message. It includes:

```
┌──────────────────────────────────────────┐
│ Agent Identity + Rules                    │
│ - "You are Hardware Pilot"                    │
│ - "NEVER ask the user to write code"     │
│ - "NEVER ask the user to pick a board"   │
│ - 4-phase autonomous workflow            │
├──────────────────────────────────────────┤
│ Current Hardware Context                  │
│ - Board: ESP32-WROOM-32 (FQBN: ...)     │
│ - Port: /dev/cu.usbserial-0001          │
├──────────────────────────────────────────┤
│ Current Sketch                            │
│ - Path: /path/to/sketch/                 │
│ - Code: (full .ino content)              │
├──────────────────────────────────────────┤
│ Last Build Output / Errors                │
├──────────────────────────────────────────┤
│ Serial Monitor Buffer (last 200 lines)    │
└──────────────────────────────────────────┘
```

### Autonomous Workflow (4 Phases)

The system prompt instructs Claude to follow this workflow when a user describes a project:

**Phase 1: Design** (no user input needed)
1. `suggest_design` → board + components + pins + libraries
2. `validate_design` → check for errors
3. Present the design to the user

**Phase 2: Wiring** (user does physical work)
4. `generate_wiring` → step-by-step instructions
5. Walk user through one component at a time
6. Optionally `capture_photo` + `verify_wiring` to check

**Phase 3: Code** (fully autonomous)
7. `write_file` → complete Arduino sketch
8. `compile` → if errors, fix and recompile (loop)

**Phase 4: Deploy & Test**
9. `upload` → flash to board
10. `read_serial` → verify it works
11. If broken, diagnose and fix

---

## Tool Registry

Tools are registered in `AgentToolRegistry` (singleton, injectable). Each tool implements:

```typescript
interface AgentTool {
  name: string;
  description: string;              // LLM reads this to decide when to use the tool
  parameters: Record<string, {
    type: string;
    description: string;
  }>;
  required?: string[];
  execute(
    params: Record<string, unknown>,
    context: AgentContext
  ): Promise<ToolResult>;
}
```

### Tool Catalog

#### Core IDE Tools (Phase 1 — existing)

| Tool | Description | Implementation |
|------|-------------|----------------|
| `read_file` | Read a file from the sketch directory | `fs.readFileSync` |
| `write_file` | Write/overwrite a file in the sketch directory | `fs.writeFileSync` |
| `list_files` | List files in the sketch directory | `fs.readdirSync` |
| `compile` | Compile the current sketch | `arduino-cli compile` subprocess |
| `upload` | Compile + upload to connected board | `arduino-cli upload` subprocess |
| `read_serial` | Read recent serial monitor output | From `AgentContext.serialBuffer` |
| `suggest_library` | Recommend Arduino libraries for a functionality | Static keyword map |

#### Hardware Intelligence Tools (Phase 2 — new)

| Tool | Description | Implementation |
|------|-------------|----------------|
| `suggest_design` | From a prompt, pick board + components + pins | Knowledge base query + `assignPins()` + `validatePins()` |
| `validate_design` | Check a design for pin/power/voltage issues | `validatePins()` + `checkPowerBudget()` |
| `generate_wiring` | Generate step-by-step breadboard instructions | Format from `PinAssignment[]` |

#### Camera Tools (Phase 2 — new)

| Tool | Description | Implementation |
|------|-------------|----------------|
| `capture_photo` | Capture webcam image as base64 JPEG | `imagesnap` (macOS) subprocess |
| `verify_wiring` | Vision-verify breadboard against design | Anthropic SDK → Claude Vision API |

---

## Hardware Knowledge Base

### Board Specs (`boards.json`)

Each board entry contains:
- Pin map with capabilities (I2C, SPI, I2S, PWM, ADC)
- Strapping pins (ESP32: GPIO 0, 2, 12, 15)
- Input-only pins (ESP32: GPIO 34-39)
- Bus counts (I2C, I2S, SPI)
- Power budget (max 3.3V rail current)
- Features (wifi, bluetooth, camera, sd_card)
- FQBN for arduino-cli

### Component Specs (`components.json`)

Each component entry contains:
- Bus type and direction
- Voltage requirements
- Current draw
- Pin roles needed (SDA, SCL, BCLK, etc.)
- I2C address (if applicable)
- Required Arduino libraries

### Pin Validation Engine (`pins.ts`)

Three core functions:

1. **`assignPins(board, components)`** — Auto-assigns board pins to component needs. Respects bus sharing (I2C devices share SDA/SCL), avoids strapping pins, avoids input-only pins for outputs.

2. **`validatePins(board, assignments)`** — Checks for:
   - GPIO conflicts (two components on same pin)
   - Output on input-only pin
   - Strapping pin usage (warning)
   - I2C address collision
   - Voltage mismatch (5V component on 3.3V board)

3. **`checkPowerBudget(board, components)`** — Sums current draws, compares against board capacity.

---

## RPC Protocol

Frontend and backend communicate via JSON-RPC over WebSocket (Theia standard).

### AgentService (frontend → backend)

```typescript
interface AgentService {
  chat(sessionId: string, userMessage: string, context: AgentContext): Promise<void>;
  abort(sessionId: string): Promise<void>;
  createSession(): Promise<string>;
  getSession(sessionId: string): Promise<AgentSession | undefined>;
  clearSession(sessionId: string): Promise<void>;
}
```

### AgentServiceClient (backend → frontend, streaming)

```typescript
interface AgentServiceClient {
  onToken(sessionId: string, token: string): void;
  onToolStart(sessionId: string, toolName: string, params: Record<string, unknown>): void;
  onToolEnd(sessionId: string, toolName: string, result: string, success: boolean): void;
  onMessage(sessionId: string, message: AgentMessage): void;
  onDone(sessionId: string): void;
  onError(sessionId: string, error: string): void;
}
```

### Context Snapshot

```typescript
interface AgentContext {
  sketchCode: string;       // Full .ino content (read server-side)
  sketchPath: string;       // Sketch directory path
  boardFqbn: string;        // e.g., "esp32:esp32:esp32"
  boardName: string;        // e.g., "ESP32-WROOM-32"
  port: string;             // e.g., "/dev/cu.usbserial-0001"
  lastBuildOutput: string;
  lastBuildErrors: string;
  serialBuffer: string;     // Last 200 lines
}
```

---

## Adding a New Tool

1. Create `arduino-ide-extension/src/node/agent/tools/my-tool.ts`
2. Export a factory function that returns an `AgentTool`
3. Register in `agent-tools.ts` → `registerBuiltins()`
4. The tool automatically appears in the Claude system prompt
5. Write tests in `tools/__tests__/my-tool.test.ts`

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full guide with code examples.

---

## Session Model

- One session per IDE window
- Sessions are in-memory (not persisted to disk)
- Session contains full message history (user, assistant, tool results)
- `clearSession()` resets history
- `abort()` sets a flag checked between tool calls

---

## Future Architecture

### Planned: Datasheet Intelligence
- PDF upload → extract register maps → generate driver code
- Uses `pdf-parse` for PDF parsing

### Planned: HIL (Hardware-in-the-Loop) Mode
- Continuous serial monitoring loop
- ARM Cortex-M HardFault decoder
- FreeRTOS deadlock detection

### Planned: Multi-Agent
- Separate design agent and code agent
- Design agent specializes in component selection and circuit validation
- Code agent specializes in Arduino C++ generation and debugging
