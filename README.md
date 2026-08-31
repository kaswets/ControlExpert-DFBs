# BottomAeration_Hold — Smart Bottom Aeration Control DFB

**Version:** 4.59
**Platform:** Schneider Electric Control Expert V14.1
**Last modified:** 2026-08-31

---

## Description

Function block for pneumatic **bottom aeration of a single cargo hold** on a bulk cement carrier.
The hold bottom is hopper-shaped: sloped panels lead down to a central suction pipe. Those panels
contain fluidizing cloths (aeration panels) in **Port/Starboard pairs** — blowing air underneath
them lets the cement flow toward the centre.

The same block also drives a smaller, related **gutter aeration** subsystem (up to 4 valves) used
to keep the collection gutters clear.

A **smart mode** (`SmartBottomOn`) prioritises whichever panel still has the most back-pressure
relative to its own calibrated empty baseline, instead of a fixed round-robin. This matters
operationally: discharge rate is roughly 1200 t/h with a full hold, dropping to ~100 t/h or less
near-empty — so clearing the last bit of cement faster has an outsized effect on average tonnage/hour.

---

## Key concepts

- **Back-pressure as a proxy for "how full":** cement still on a panel resists the airflow, giving
  higher blower back-pressure. A bare panel gives low back-pressure — nothing left to aerate there.
- **Panel-specific empty baseline (`EmptyPressBothSides[]`):** panels aren't all the same size, so
  "empty" doesn't read the same back-pressure everywhere. Calibrated per panel via `FindEmpty`
  while the hold is genuinely empty; retentive (`Save`-flagged) so it survives a restart.
- **Smart valve selection:** on every `NextStep` pulse, compares `ActualPress / EmptyPress` per
  panel (as a %) and switches to whichever *other* panel has the highest ratio. The panel currently
  open is always excluded from re-selection, so every panel's reading stays fresh.
- **Clinometer interlock:** at excessive list angle (`Clino.X_PS_Off` / `X_SB_Off`), smart selection
  is disabled and the system falls back to a plain round-robin.
- **Empty detection (`InOut.Control.Empty`):** TRUE once every panel's actual pressure is below
  ~110% of its own empty baseline.

---

## Interface

### Inputs

| Name | Type | Description |
|------|------|-------------|
| `Inputs` | `Bottom_IN` | Hardware/feedback inputs for this hold |
| `PulseOn` | `BOOL` | Start pulsing bottom aeration |
| `Parameters` | `Bottom_Settings` | Configurable parameters for this hold |
| `Clino` | `Bottom_Clino` | Clinometer (list angle) status, Port/Starboard |
| `Blower` | `Aeration_Blower` | Blower run status and back-pressure (bottom + gutter) |
| `System` | `INT` | System group assignment (passed down to each `DValve`) |

### InOut

| Name | Type | Description |
|------|------|-------------|
| `General` | `General` | Shared system struct (EMStop, ServiceAir, AlarmReset, manual groups) |
| `InOut` | `Bottom_InOut` | HMI controls/status: `SmartBottomOn`, `FindEmpty`, `Empty`, per-valve pressure array, manual controls |

### Outputs

| Name | Type | Description |
|------|------|-------------|
| `Outputs` | `Bottom_Out` | Hardware outputs — one per valve (up to 12 PS + 12 SB + 4 gutter) |

### Public local variables

| Name | Type | Description |
|------|------|-------------|
| `NumberOfValves` | `INT` | Total bottom valves configured for this hold (PS+SB together) |
| `EmptyPressBothSides[1..12]` | `ARRAY OF INT` | Empty-baseline back-pressure per panel pair (retentive) |
| `ActualPressBothSides[1..12]` | `ARRAY OF INT` | Last measured back-pressure per panel pair |

---

## Program sections

| Section | Language | Description |
|---------|----------|--------------|
| `ThroughToDeeper` | ST | Copies `System` input to an internal working variable |
| `Main` | LD | Timer setup, instantiates the bottom (`PulseOrder`) and gutter (`PulseOrder`) step sequencers, gutter start/stop logic |
| `Valves` | ST | Instantiates each `DValve` (PS1–12, SB1–12), sets per-valve parameters (Clino signal, delays, monitor time) |
| `SmartBottom` | ST | Pressure-based valve selection — see Key concepts above |
| `smartBottomExtraInfo` | ST | Watch-window helper: exposes the pressure arrays for HMI/debug visibility |
| `Alarm` | LD | Alarm handling |

---

## Included DFBs and DDTs

This `.xdb` contains everything needed:

| Name | Type | Description |
|------|------|-------------|
| `BottomAeration_Hold` | DFB | Main block — one instance per cargo hold |
| `PulseOrder` | DFB | Helper — generic step sequencer, opens one valve at a time for a configurable pulse length; used for both the bottom valves and the gutter valves |
| `Int_To_Time2` | DFB | Helper — converts an `INT` (seconds) setpoint to a `TIME` value |
| `DValve` (v2.43) | DFB | Digital valve control — see the [DValve](../DValve/) entry for details. Bundled here one version ahead of the standalone `DValve` folder (2.42); the standalone copy hasn't been re-exported yet |
| `RotateValve` | DFB | Helper — packs rotation angle into the valve HMI status word (used by `DValve`) |
| `BitCheck` | DFB | Helper — splits a `WORD` into 16 `BOOL`s + counts active bits (used by `DValve`) |
| `Bottom_IN`, `Bottom_InOut`, `Bottom_Out`, `Bottom_Settings`, `Bottom_Clino` | DDT | I/O and settings structures for this block |
| `Aeration_Blower` | DDT | Blower run status + back-pressure readings |
| `ValveControl` | DDT | 32-bit step output word used by `PulseOrder` |

---

## How to import

1. Open your project in **Control Expert**
2. **File → Import → Import DFB...**
3. Select `bottomaeration_hold.xdb`
4. All DDTs and helper DFBs (`PulseOrder`, `Int_To_Time2`, `DValve`, `RotateValve`, `BitCheck`) are imported in one step

> ⚠️ Since this bundles `DValve` v2.43 (one version ahead of the standalone `DValve` folder), check for version-conflict warnings if `DValve` is already present in your target project.

---

## Changelog

| Version | Date | Notes |
|---------|------|-------|
| 4.59 | 2026-08-31 | Fixed `SmartBottom` valve selection: valve 1 was missing the same `ActiveStep` exclusion as every other valve in the search loop, which was masked by a hardcoded fallback (`HighestStep:=2`) that jumped to a fixed valve instead of the true runner-up — causing a valve-1 ↔ valve-2 oscillation whenever valve 1 remained the fullest panel late in discharge (exactly the phase where it matters most for average t/h). Valve 1 now goes through the same loop/exclusion as the rest, and `HighestValue`/`HighestStep` are re-initialised on every search instead of only on enable. A bilingual (NL/EN) fixed-width ASCII comment block documenting the logic and variables was added at the top of `SmartBottom`. |
| 4.57 | 2026-05-11 | Prior version |
