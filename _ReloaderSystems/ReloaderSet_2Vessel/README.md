# ReloaderSet — 2-Vessel Bulk Cement Reloader System

**Version:** 6.86  
**Platform:** Schneider Electric Control Expert V14.1  
**Last modified:** 2026-05-11

---

## Description

Function block for a **2-vessel pneumatic bulk cement reloader system** as supplied by VAMO (Van Aalst Marine & Offshore).

The system consists of two pressure vessels (Reloader A and Reloader B) that work alternately — while one vessel is being filled with cement via vacuum suction, the other is pressurised and discharging cement to the hold or shore connection. This alternating cycle ensures a continuous cement flow.

Each vessel has a **filter on top** to prevent cement dust from entering the vacuum pump during the suction phase. The filters are cleaned automatically by a pulse of compressed air (filter cleaning cycle).

---

## System cycle per vessel

Each vessel cycles through the following states:

```
Init → Pressurising → Discharging → Emptying → Filling → (back to Pressurising)
```

| State | Description |
|-------|-------------|
| **Init** | Vessel at rest, waiting to start |
| **Pressurising** | Vessel is being pressurised with compressed air |
| **Discharging** | Vessel pressure reached — cement is pushed out to hold or shore |
| **Emptying** | Vessel nearly empty — finishing discharge |
| **Filling** | Vessel depressurised — vacuum applied, cement sucked in from hold |

While vessel A is Discharging, vessel B is Filling — and vice versa.  
The **switchover** happens automatically when the discharging vessel reaches the low pressure setpoint.

---

## Interface

### Inputs

| Name | Type | Description |
|------|------|-------------|
| `ReloaderA_In` | `Reloader_In` | Hardware inputs for vessel A (pressure, levels, valve feedback) |
| `ReloaderB_In` | `Reloader_In` | Hardware inputs for vessel B |
| `Set_In` | `ReloaderSet_In` | Shared system inputs (vacuum pump status, blower status, etc.) |
| `Settings` | `ReloaderSet_Settings` | Pressure setpoints and timing settings |
| `System` | `INT` | System group — determines which hold is active (Shore=0, Hold1=1, Hold2=2, etc.) |

### InOut

| Name | Type | Description |
|------|------|-------------|
| `ReloaderA_InOut` | `Reloader_InOut` | HMI commands and counters for vessel A |
| `ReloaderB_InOut` | `Reloader_InOut` | HMI commands and counters for vessel B |
| `Set` | `ReloaderSet_InOut` | Shared HMI commands (Start, Stop, hold selection, manual valves) |
| `General` | `General` | Shared system struct |

### Outputs

| Name | Type | Description |
|------|------|-------------|
| `ReloaderA_Out` | `Reloader_Out` | Hardware outputs for vessel A (valves, motor commands) |
| `ReloaderB_Out` | `Reloader_Out` | Hardware outputs for vessel B |
| `Set_Out` | `ReloaderSet_Out` | Shared outputs (vacuum pump command, blower command, etc.) |

---

## Pressure setpoints

The system uses three pressure setpoints, switchable between **Shore discharge** and **Hold discharge**:

| Setpoint | Description |
|----------|-------------|
| `Settings.HighShore` / `Settings.HighHold` | Pressure at which discharging starts |
| `Settings.MidShore` / `Settings.MidHold` | Intermediate pressure — controls discharge rate |
| `Settings.LowShore` / `Settings.LowHold` | Low pressure — triggers switchover to other vessel |

---

## Key features

- **Automatic alternating cycle** — A and B switch automatically based on pressure and level signals
- **Vacuum management** — vessels request vacuum from the shared vacuum pump; only one vessel gets vacuum at a time
- **Equalizer valve** — pressure equalisation between vessels before switchover (`DValve` instance inside the DFB)
- **Filter cleaning** — automatic filter pulse cleaning during the filling phase (`Purge` DFB instance)
- **Hold selection** — up to 5 holds (Hold 1–5), switchable while running
- **Discharge counter** — counts switchovers and high-level events per vessel
- **Dust detection** — monitors filter condition

---

## Program sections

| Section | Language | Description |
|---------|----------|-------------|
| `ThroughToDeeper` | ST | Passes System value to sub-DFBs |
| `Transitions` | LD | State machine transitions, counters, pressure setpoint selection |
| `ReloaderA` | LD/ST | Vessel A state machine and valve commands |
| `ReloaderB` | LD/ST | Vessel B state machine and valve commands |
| `VacuumManagement` | LD/ST | Vacuum pump requests and allocation |
| `HoldValves` | LD/ST | Hold inlet valve control (Hold 1–5) |
| `Alarms` | ST | Alarm detection and handling |

---

## Internal DFB instances

This DFB uses several other DFBs internally:

| Instance | Type | Description |
|----------|------|-------------|
| `Reloader_A_IDB` | `Reloader` | Individual vessel control for vessel A |
| `Reloader_B_IDB` | `Reloader` | Individual vessel control for vessel B |
| `Equalizer` | `DValve` | Equalizer valve between the two vessels |
| `Purge_IDB` | `Purge` | Filter cleaning control |
| `Hold1`–`Hold5` | `DValve` | Hold inlet valves |

---

## Included DDTs

| Name | Description |
|------|-------------|
| `Reloader_In` | Hardware inputs per vessel |
| `Reloader_InOut` | HMI commands and counters per vessel |
| `Reloader_Out` | Hardware outputs per vessel |
| `ReloaderSet_In` | Shared system inputs |
| `ReloaderSet_InOut` | Shared HMI commands |
| `ReloaderSet_Out` | Shared outputs |
| `ReloaderSet_Settings` | Pressure setpoints and timing |

---

## Difference with other reloader system types

| System | Vessels | Filter location | Vacuum pump |
|--------|---------|-----------------|-------------|
| **2-vessel (this)** | 2 | Inside vessel top | Shared |
| 3-vessel | 3 | — | — |
| 5-vessel | 5 | — | — |

> The 3-vessel and 5-vessel variants are documented in their own subfolders.

---

## How to import

1. Open your project in Control Expert
2. **File → Import → Import DFB...**
3. Select `reloaderset.xdb`
4. All DDTs and sub-DFBs are imported automatically

> ⚠️ This DFB uses `DValve`, `Reloader`, and `Purge` internally. If these already exist in your project, check for version conflicts before importing.

---

## Changelog

| Version | Date | Notes |
|---------|------|-------|
| 6.86 | 2026-05-11 | Current version |
