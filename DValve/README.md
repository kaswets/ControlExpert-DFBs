# DValve — Digital Valve Control DFB

**Version:** 2.42  
**Platform:** Schneider Electric Control Expert V14.1  
**Last modified:** 2026-05-13

---

## Description

Function block for controlling a **digital (on/off) valve**.  
Handles open/close commands, feedback monitoring, alarm detection, manual override, and HMI status communication.

Supports both **normally closed** and **normally open** valve types via the `Parameters.NormallyOpen` setting.  
This affects both the output logic and how the position feedback switches are interpreted:
- **Normally closed** (default) — valve is closed without power, output energised to open
- **Normally open** — valve is open without power, output energised to close. Feedback switches are interpreted inversely.

Supports both **single-output** (open only) and **dual-output** (separate open/close) valve types.  
The valve can be rotated on the HMI up to 360 degrees for correct visual representation of slide valves, butterfly valves, etc.

---

## Interface

### Inputs

| Name | Type | Description |
|------|------|-------------|
| `Enable` | `BOOL` | Valve enabled — alarms are suppressed when FALSE |
| `In` | `DValve_In` | Hardware feedback (FB_Open, FB_Closed) |
| `Valve_Command` | `BOOL` | Automatic open command |
| `InOut` | `DValve_InOut` | HMI signals (ManualControl, status word) |
| `General` | `General` | Shared system struct (EMStop, ServiceAir, AlarmReset, Manual groups) |
| `Angle` | `WORD` | Visual rotation angle on HMI (0–360 degrees) |
| `System` | `INT` | System group assignment — determines which manual group controls this valve |

### InOut

| Name | Type | Description |
|------|------|-------------|
| `InOut` | `DValve_InOut` | HMI commands and status word |
| `General` | `General` | Shared system struct |

### Outputs

| Name | Type | Description |
|------|------|-------------|
| `Out` | `DValve_Out` | Hardware outputs (Output, OutputOpen, OutputClose, Alarm) |
| `travelActiveTime` | `INT` | Measured travel time opening |
| `travelDeactiveTime` | `INT` | Measured travel time closing |

### Parameters (public, configurable per instance)

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `Parameters.MonitorTime` | `TIME` | — | Max travel time — alarm if valve doesn't reach position within this time |
| `Parameters.Delayed_Return` | `TIME` | — | Delay closing output AND status after command removed |
| `Parameters.Delayed_ReturnOutput` | `TIME` | — | Delay closing output only (not status) |
| `Parameters.NormallyOpen` | `BOOL` | FALSE | Set TRUE for normally open valves — inverts output logic and feedback interpretation |
| `Parameters.Clino_Enable` | `BOOL` | FALSE | Enable clinometer interlock |
| `Parameters.Clino_Signal` | `BOOL` | — | Close valve on clinometer signal (ship inclination) |
| `Parameters.2Outputs` | `BOOL` | FALSE | Use separate OutputOpen / OutputClose instead of single Output |

---

## HMI Status Word

The DFB sends a single WORD to the HMI with all valve status packed in bits:

| Bit | Description |
|-----|-------------|
| 0 | Valve position: 0 = closed (grey), 1 = open (green) |
| 1 | Moving (yellow) — both limit switches absent |
| 2 | Alarm (red) |
| 3 | Manual active — touch button visible and active |
| 4 | Manual touch button state |
| 5–13 | Valve angle on HMI (0–360 degrees) |

---

## System groups — Manual control

The `System` input determines which HMI manual group controls this valve:

| System value | Behaviour | Description |
|-------------|-----------|-------------|
| 1 | Loose | Fore system only |
| 2 | Loose | Aft system only |
| 3 | Loose | Fore OR Aft |
| 4 | Loose | Lubrication only |
| 11 | Exact | Fore AND Aft must both be active |
| 12 | Exact | Lubrication must be exactly active |

**Loose:** valve goes to manual if at least one of its group bits matches the active HMI manual buttons.  
**Exact:** the active HMI manual buttons must exactly match the valve's group assignment.

---

## Logic overview

1. **Manual control** — operator presses manual button on HMI → `BtnManual` toggles → valve can be opened/closed manually
2. **Auto control** — `Valve_Command` drives the output when not in manual and no alarm
3. **Air interlock** — valve only opens if service air is available (`General.ServiceAirForeOkay` / `ServiceAirAftOkay` based on `System`)
4. **EMStop** — valve closes on emergency stop (`General.EMStopOkay`)
5. **Clinometer** — optional: valve closes on ship inclination signal
6. **Alarm** — `TON_Alarm` monitors travel time; alarm if valve doesn't reach position within `MonitorTime`
7. **Alarm reset** — via `General.Alarm_Reset`

---

## Program sections

| Section | Language | Description |
|---------|----------|-------------|
| `Info` | ST | Documentation and HMI status word bit layout |
| `NewManual` | ST | Determines if this valve is in manual based on System group and HMI manual buttons |
| `ValveNew` | LD | Main valve logic — outputs, feedback, alarm, manual, timers |
| `Alarm` | ST | Travel time monitoring and alarm generation |
| `HMIStatus` | ST | Packs valve status into single WORD for HMI |
| `RotateValve` | ST | Sets valve rotation angle in HMI status word |

---

## Included DFBs and DDTs

This `.xdb` contains everything needed:

| Name | Type | Description |
|------|------|-------------|
| `DValve` | DFB | Main valve control block |
| `DValve_In` | DDT | Hardware inputs (FB_Open, FB_Closed) |
| `DValve_InOut` | DDT | HMI commands and status |
| `DValve_Out` | DDT | Hardware outputs |
| `DValve_Param` | DDT | Configuration parameters |
| `RotateValve` | DFB | Helper — packs rotation angle into HMI status word |
| `BitCheck` | DFB | Helper — splits a WORD into 16 individual bits + counts active bits |

---

## How to import

1. Open your project in Control Expert
2. **File → Import → Import DFB...**
3. Select `dvalve.xdb`
4. All DDTs and helper DFBs are imported in one step

---

## Changelog

| Version | Date | Notes |
|---------|------|-------|
| 2.42 | 2026-05-13 | Current version |
| 2.x | — | Added EXACT manual group mode (bit3 in System) |
| 1.x | — | Added 360° valve rotation for slide valves |
| 0.x | — | Original version — separate status words, 90° rotation only |
