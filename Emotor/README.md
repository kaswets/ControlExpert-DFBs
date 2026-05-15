# Emotor — Electric Motor Control DFB

**Version:** 0.84  
**Platform:** Schneider Electric Control Expert V14.1  
**Type:** Derived Function Block (DFB) — Diagnostic enabled  
**Last modified:** 2026-05-13

---

## Description

Function block for controlling a single electric motor.  
Handles start/stop commands, sequenced starting with other motors, alarm detection, and PMS integration.

Originally the alarm handling was done outside the function block via separate `Alrm_dia` blocks and alarm words. This version uses the **UREGDFB diagnostic system** directly inside the DFB (`IsTypeDiagnostic = TRUE`), which greatly simplifies the overall program structure.

---

## Interface

### Inputs

| Name | Type | Description |
|------|------|-------------|
| `Inputs` | `Emotor_In` | Hardware feedback from the field (Running, PowerAvailable, GenAlarm) |
| `EmotNr` | `INT` | Unique motor number — used to identify this motor in the start queue |
| `MainTxt` | `STRING[40]` | Motor name shown in alarm messages |

### InOut

| Name | Type | Description |
|------|------|-------------|
| `InOut` | `Emotor_InOut` | HMI commands: Start, Stop, Alarm reset, PulsOnOff |
| `General` | `General` | Shared system struct — passed to all motor blocks in the program |
| `StartRow` | `ARRAY[0..30] OF INT` | Shared start queue — also passed to all motor blocks |

### Outputs

| Name | Type | Description |
|------|------|-------------|
| `Outputs` | `Emotor_Out` | Hardware outputs: Start, Stop, PowerRequest, Alarms |

### Public local variables (Parameters)

| Name | Type | Description |
|------|------|-------------|
| `Parameters.WaitOnPMS` | `BOOL` | Motor waits for PMS power available before starting |
| `Parameters.StartupDelayTime` | `INT` | Seconds other motors must wait after this one starts |
| `Parameters.OneOutput` | `BOOL` | Use single combined output instead of separate Start/Stop |
| `Parameters.OffByFireSwitch` | `BOOL` | Motor stops on fire switch signal |
| `Parameters.DontReactOnLostRun` | `BOOL` | Ignore lost run feedback signal |
| `AREA_NR` | `INT` | Area number for alarm routing (default: 1) |
| `OP_CTRL` | `BOOL` | Operator control enabled (default: TRUE) |

---

## How the start queue works

All motor instances in the program share one `StartRow[0..30]` array (passed as InOut).

- When a motor receives a **Start** command, it adds its `EmotNr` to the back of the queue
- The queue shifts forward automatically — the motor at position `[0]` is the one allowed to start
- When a motor reaches position `[0]` and conditions are met, it sets `Starting` and activates the output
- On **Stop** or **Alarm**, the motor removes itself from the queue and the others shift up

This allows sequential starting of multiple motors without any external sequencer logic.

---

## Alarm handling

This DFB uses `IsTypeDiagnostic = TRUE`, which allows placing UREGDFB diagnostic blocks directly inside the function block.

- Alarm text is passed via the `MainTxt` input string
- The `General` struct contains shared alarm bits (e.g. `EMStopOKay`, `SimulationOn`)
- Alarm reset is handled via `General.Alarm_Reset`

---

## Program sections inside the DFB

| Section | Language | Description |
|---------|----------|-------------|
| `info` | ST | Development history and comments |
| `ExtraForAlarms` | ST | Enables internal alarms when `MainTxt` is set |
| `OneButtunStartStop` | ST | Handles pulse toggle button logic |
| `ShiftStartArrayUp` | ST | Adds motor to start queue on rising edge of Start |
| `EmotIsInList` | ST | Checks if this motor is currently in the queue |
| `AbortEmot` | ST | Removes motor from queue on Stop or Alarm |
| `Prg_Emot` | LD | Main motor control logic (start/stop outputs, timers, alarms) |

---

## Included DDTs

All data types are bundled inside `emotor.xdb` and will be imported automatically:

| DDT | Description |
|-----|-------------|
| `Emotor_In` | Hardware inputs struct |
| `Emotor_InOut` | HMI command struct |
| `Emotor_Out` | Hardware outputs struct |
| `General` | Shared system struct |
| `Parameters` | Motor configuration parameters |
| `AlarmWord` | 16-bit alarm word with named bits |
| `manualWrd` | Manual control word (up to 4 systems) |

---

## How to import

1. Open your project in Control Expert
2. **File → Import → Import DFB...**
3. Select `emotor.xdb`
4. All DDTs and the DFB are imported in one step

> ⚠️ If `General` or other DDTs already exist in your project, Control Expert will warn about version conflicts. Compare versions before overwriting.

---

## Changelog

| Version | Date | Notes |
|---------|------|-------|
| 0.84 | 2026-05-13 | Alarm handling moved into DFB via UREGDFB (IsTypeDiagnostic) |
| 0.04 | ~2019 | Original version with external AlarmWord approach |
