# Select_Valve — Selection Valve with Motor DFB

**Version:** 2.41  
**Platform:** Schneider Electric Control Expert V14.1  
**Type:** Derived Function Block (DFB) — Diagnostic enabled  
**Last modified:** 2026-05-11

---

## Description

Function block for controlling a **3-position selection valve** driven by an electric motor.  
The valve can move to three positions: **Left**, **Right**, or **Middle**.

Typical application: airslide diverter valves in bulk cement handling systems — directing cement flow to port side, starboard side, or a middle/neutral position.

> ⚠️ **Important during commissioning:** The command value represents where the **cement should go**, not the physical rotation direction of the valve. So if the command is "Port Side", the valve must physically move in whatever direction routes cement to port — regardless of which way the motor turns.

---

## Interface

### Inputs

| Name | Type | Description |
|------|------|-------------|
| `Enable` | `BOOL` | Valve enabled — alarms suppressed when FALSE |
| `Inputs` | `Select_Valve_In` | Hardware feedback (position switches Left, Middle, Right + motor running) |
| `Valve_Command` | `INT` | Position command: `1` = Right, `2` = Left, `3` = Middle |
| `MidPosition` | `BOOL` | Valve has a middle position available |
| `Test` | `BOOL` | Test mode — enables pulsed manual movement for commissioning |
| `System` | `INT` | System group assignment — same as DValve (Fore/Aft/Lub, Loose/Exact) |
| `MainTxt` | `STRING[40]` | Valve name for alarm messages — leave empty to disable internal alarms |

### InOut

| Name | Type | Description |
|------|------|-------------|
| `InOut` | `Select_Valve_InOut` | HMI commands (Manual, position requests) |
| `General` | `General` | Shared system struct |

### Outputs

| Name | Type | Description |
|------|------|-------------|
| `Outputs` | `Select_Valve_out` | Hardware outputs (motor Left, motor Right, alarm word) |
| `ShortTime` | `DINT` | Measured travel time for short moves (middle ↔ side) |
| `LongTime` | `DINT` | Measured travel time for long moves (side ↔ side) |

### Parameters (public, configurable per instance)

| Name | Type | Description |
|------|------|-------------|
| `Parameters.MonitorTime` | `TIME` | Max travel time — alarm if position not reached within this time |
| `Parameters.ReversalDelayTime` | `TIME` | Delay before reversing motor direction |
| `Parameters.Clino_Enable` | `BOOL` | Enable clinometer interlock |
| `Parameters.Clino_Signal` | `BOOL` | Move valve to safe position on ship inclination signal |

---

## Travel time logic

The DFB distinguishes between two move types:

| Move type | Route | Monitor time |
|-----------|-------|-------------|
| **ShortWay** | Side → Middle, or Middle → Side | `MonitorTime / 2` |
| **LongWay** | Side → Side (e.g. Left → Right) | `MonitorTime` |

Actual travel times are measured and available on `ShortTime` and `LongTime` outputs — useful for tuning `MonitorTime` during commissioning.

---

## Commissioning with Test mode

On a ship there are often multiple selection valves of the same type — but several of them are physically mounted 180° rotated compared to others. This means "Left" on one valve routes cement the opposite direction compared to "Left" on the next valve. Terms like Left/Right or Port/Starboard become meaningless without checking the actual cement flow direction.

The `Test` input solves this:

1. The on-site technician sets `Test = TRUE` in the PLC program
2. In the HMI, the valve can now be operated with the manual buttons — but only in **short pulses** instead of continuous movement. This prevents the valve from running past its end position before anyone can see what's happening.
3. Someone stands at the valve and watches which way the cement (or air flow) goes
4. If the valve moves the wrong way, swap the Left/Right outputs in the panel wiring — **don't change the PLC program**
5. Once confirmed correct, set `Test = FALSE` and the valve operates normally
6. Use the `ShortTime` and `LongTime` outputs to tune `MonitorTime` to the actual travel time of this specific valve

---

## Manual control

Same system group logic as `DValve`:
- `System` input assigns the valve to Fore, Aft, or Lubrication group
- `System < 7` = Loose matching (OR)
- `System ≥ 7` = Exact matching (AND)

---

## Program sections

| Section | Language | Description |
|---------|----------|-------------|
| `Info` | ST | Commissioning notes and usage instructions |
| `extra` | ST | Enables internal alarms when `MainTxt` is set |
| `NewManual` | ST | Determines manual active state based on System group |
| `prg_Selectvalve` | LD | Main valve logic — motor outputs, position detection, alarms |

---

## Included DDTs

| Name | Description |
|------|-------------|
| `Select_Valve_In` | Hardware inputs (position feedback switches) |
| `Select_Valve_InOut` | HMI commands |
| `Select_Valve_out` | Hardware outputs |
| `Select_Valve_Parameters` | Configuration parameters |
| `BitCheck` | Helper DFB (see `_Helpers/BitCheck/`) |

---

## How to import

1. Open your project in Control Expert
2. **File → Import → Import DFB...**
3. Select `select_valve.xdb`
4. All DDTs and helpers are imported automatically

---

## Changelog

| Version | Date | Notes |
|---------|------|-------|
| 2.41 | 2026-05-11 | Current version |
