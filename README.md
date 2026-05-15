# ControlExpert-DFBs

A personal library of **Derived Function Blocks (DFBs)** for Schneider Electric **Control Expert V14.1**.  
These blocks are used in real industrial installations — mainly maritime and process automation.

---

## 📦 Available DFBs

### `Emotor` — Electric Motor Control (v0.84)
> Full-featured function block for controlling a single electric motor with sequenced starting, alarm handling, and PMS integration.

**Key features:**
- **Sequenced start queue** — multiple motors share a `StartRow[0..30]` array; only the motor at position `[0]` is allowed to start. Motors register themselves automatically on a start command.
- **PMS integration** — optional wait for ship Power Management System (`WaitOnPMS`) before allowing start
- **One-button start/stop** — supports a single push-button (pulse) for toggle start/stop operation
- **Diagnostic alarms** — uses Schneider's UREGDFB diagnostic system (DFB `IsTypeDiagnostic = TRUE`), alarm texts passed via `MainTxt` string input
- **OneOutput mode** — optional single combined output instead of separate Start/Stop signals
- **Simulation mode** — when `General.SimulationOn` is active, startup delay is fixed at 10 seconds
- **Not-starting / not-stopping alarms** — TON timers monitor whether the motor actually responds

**Interface:**

| Pin | Type | Data Type | Description |
|-----|------|-----------|-------------|
| `Inputs` | Input | `Emotor_In` | Hardware feedback (Running, PowerAvailable, GenAlarm) |
| `EmotNr` | Input | `INT` | Unique motor number (used in start queue) |
| `MainTxt` | Input | `STRING[40]` | Motor name / alarm text label |
| `InOut` | InOut | `Emotor_InOut` | Commands from HMI (Start, Stop, Alarm, PulsOnOff) |
| `General` | InOut | `General` | Shared system struct (EMStop, SimulationOn, alarms, etc.) |
| `StartRow` | InOut | `ARRAY[0..30] OF INT` | Shared start queue across all motors |
| `Outputs` | Output | `Emotor_Out` | Hardware outputs (Start, Stop, PowerRequest, Alarms) |

**Included DDTs** (all bundled in the `.xdb` file):
`Emotor_In`, `Emotor_InOut`, `Emotor_Out`, `General`, `Parameters`, `AlarmWord`, `manualWrd`

---

## 🚀 How to import

1. Open your project in **Control Expert**
2. Go to **File → Import → Import DFB...**  
   *(or right-click in the project tree → Import)*
3. Select the `.xdb` file
4. All DDTs and the DFB are imported in one go

> ⚠️ If DDTs like `General` already exist in your project, Control Expert may warn about conflicts. Check version numbers before overwriting.

---

## 🗂️ Repository structure

```
ControlExpert-DFBs/
├── Emotor/
│   ├── emotor.xdb          ← Import this into Control Expert
│   └── README.md           ← DFB-specific documentation
└── README.md               ← This file
```

---

## 🔧 Platform

| | |
|---|---|
| **Software** | Schneider Electric Control Expert V14.1 |
| **PLC hardware** | Tested on BME P58 / BMX P34 series |
| **Language** | Structured Text (ST) + Ladder (LD) |
| **Diagnostic** | UREGDFB compatible (`IsTypeDiagnostic = TRUE`) |

---

## 📝 Changelog

| Version | Date | Notes |
|---------|------|-------|
| 0.84 | 2026-05-13 | Current version — diagnostic alarms via UREGDFB |
| — | — | Earlier versions pre-date this repository |

---

*By [kaswets](https://github.com/kaswets) — arjan swets*
