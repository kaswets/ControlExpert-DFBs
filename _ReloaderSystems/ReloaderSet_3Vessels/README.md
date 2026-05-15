# ReloaderSet — 3-Vessel Bulk Cement Reloader System

**Version:** 10.34  
**Platform:** Schneider Electric Control Expert V14.1  
**Last modified:** 2026-02-11

---

## Description

Function block for a **3-vessel pneumatic bulk cement reloader system** as supplied by VAMO (Van Aalst Marine & Offshore).

The system consists of one **vacuum vessel** on top and two **pressure vessels** (Reloader A and Reloader B) underneath. This is a more advanced system than the 2-vessel type.

### How it works

```
    ┌─────────────────────────────────────┐
    │              Vacuum Vessel          │
    │ - - - - - - - - - - - - - - - - - - │ ← filter
    │    │                          │     │
    │    │      (vent leiding)      │     │ ← V12/V22 leiding loopt door tank
    │    │                          │     │   omhoog tot net onder filter
    └────┬────┴────────────────┴────┬─────┘
         │   V10              V20   │          ← fill kleppen (cement valt naar beneden)
        V12   │                │   V22         ← vent kleppen (lucht omhoog, afgezogen)
         │    │                │    │
    ┌────▼────▼────┐      ┌────▼────▼────┐
    │              │      │              │
    │  Reloader A  │      │  Reloader B  │
    │              │      │              │
    └──────────────┘      └──────────────┘
```

1. The **vacuum vessel** continuously sucks cement from the hold — it keeps sucking as long as at least one pressure vessel below has room
2. Cement falls by gravity from the vacuum vessel into a pressure vessel via the **fill valve** (V10 or V20)
3. At the same time, the air that was in the pressure vessel must escape upward — this happens via the **vent valve** (V12 or V22), which connects high up in the vacuum vessel just below the filter. The escaping air is immediately drawn away by the vacuum pump. Without this separate vent line, cement and air would have to pass through the same opening in opposite directions, blocking the flow.
4. When a pressure vessel is full, both its fill valve and vent valve close — the vacuum vessel then fills the other pressure vessel
5. The full pressure vessel is pressurised and discharges cement to shore or hold
6. The vacuum vessel only stops sucking when **all three vessels are full**
7. The **filter** is located in the vacuum vessel (not in the pressure vessels like the 2-vessel system) — it is cleaned automatically by a filter pulse system

This design allows a much more continuous cement flow because the vacuum vessel acts as a buffer between the suction and discharge cycles.

---

## Files in this folder

| File | Description |
|------|-------------|
| `reloader_3vessels.xdb` | Main ReloaderSet DFB — controls both pressure vessels and coordinates with vacuum vessel |
| `vacvessel.xdb` | VacVessel DFB — controls the vacuum vessel separately |

Both DFBs must be imported and used together in the project.

---

## ReloaderSet interface

### Inputs

| Name | Type | Description |
|------|------|-------------|
| `ReloaderA_In` | `Reloader_In` | Hardware inputs for pressure vessel A |
| `ReloaderB_In` | `Reloader_In` | Hardware inputs for pressure vessel B |
| `Set_In` | `ReloaderSet_In` | Shared system inputs |
| `Settings` | `Reloader_Settings` | Pressure setpoints and timing |

### InOut

| Name | Type | Description |
|------|------|-------------|
| `ReloaderA_InOut` | `Reloader_InOut` | HMI commands and counters for vessel A |
| `ReloaderB_InOut` | `Reloader_InOut` | HMI commands and counters for vessel B |
| `VacVessel` | `VacVessel_InOut` | Interface to the VacVessel DFB — shared state between the two DFBs |
| `Set` | `ReloaderSet_InOut` | Shared HMI commands |
| `General` | `General` | Shared system struct |

### Outputs

| Name | Type | Description |
|------|------|-------------|
| `ReloaderA_Out` | `Reloader_Out` | Hardware outputs for pressure vessel A |
| `ReloaderB_Out` | `Reloader_Out` | Hardware outputs for pressure vessel B |
| `Set_Out` | `ReloaderSet_Out` | Shared outputs |

### Public parameters

| Name | Type | Description |
|------|------|-------------|
| `MaxVac` | `INT` | Maximum vacuum pressure setpoint |
| `ReleaseVac` | `INT` | Vacuum release pressure setpoint |
| `MaxComp` | `INT` | Maximum compression pressure setpoint |

---

## VacVessel interface

### Inputs

| Name | Type | Description |
|------|------|-------------|
| `In` | `VacVessel_In` | Hardware inputs (pressure, level, valve feedback) |
| `Settings` | `VacVesselSettings` | Vacuum vessel settings |

### InOut

| Name | Type | Description |
|------|------|-------------|
| `InOut` | `VacVessel_InOut` | HMI commands and status — **shared with ReloaderSet via VacVessel pin** |
| `reloader1` | `Reloader_InOut` | State of pressure vessel A — used to decide when to stop suction |
| `reloader2` | `Reloader_InOut` | State of pressure vessel B |
| `ReloaderSetA` | `ReloaderSet_InOut` | Shared set state |
| `general` | `General` | Shared system struct |

### Outputs

| Name | Type | Description |
|------|------|-------------|
| `Out` | `VacVessel_Out` | Hardware outputs (vacuum valve, filter pulse, bypass valve) |

---

## Difference with 2-vessel system

| | 2-vessel | 3-vessel |
|--|---------|---------|
| **Vessels** | 2 pressure vessels | 1 vacuum vessel + 2 pressure vessels |
| **Filter location** | Inside each pressure vessel (top) | Inside vacuum vessel only |
| **Suction** | Each vessel sucks and pressurises alternately | Vacuum vessel sucks continuously |
| **Continuity** | Brief stop during switchover | More continuous — vacuum vessel is buffer |
| **DFBs needed** | `ReloaderSet` only | `ReloaderSet` + `VacVessel` |
| **Complexity** | Simpler | More complex but higher throughput |

---

## How to import

Import both DFBs separately:

1. **File → Import → Import DFB...** → select `reloader_3vessels.xdb`
2. **File → Import → Import DFB...** → select `vacvessel.xdb`

Connect them in the program via the shared `VacVessel_InOut` variable.

---

## Changelog

### ReloaderSet (3-vessel)
| Version | Date | Notes |
|---------|------|-------|
| 10.34 | 2026-02-11 | Current version |

### VacVessel
| Version | Date | Notes |
|---------|------|-------|
| 3.63 | 2026-02-11 | Current version |
