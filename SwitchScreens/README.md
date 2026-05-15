# SwitchScreens — Multi-HMI Screen Control

**Platform:** Schneider Electric Control Expert V14.1  
**HMI platform:** Vijeo Designer  
**Last modified:** 2026-05-13

This folder contains two companion function blocks that work together:

| DFB | Version | Description |
|-----|---------|-------------|
| `SwitchScreens` | 1.47 | Core screen switching logic |
| `SwitchScreenMemmory` | 0.13 | Per-panel persistent settings (optional companion) |

---

## SwitchScreens (v1.47)

Function block that manages up to **4 identical HMI panels** connected to a single PLC.  
All panels run the same Vijeo Designer application — the panel's own IP address is the only identifier.

Only **one panel at a time** has control (the active screen). The others are in view-only mode.  
Any view-only panel can request a take-over via a confirmation dialog (Yes/No).

### Key features

- **Self-registration** — panels register themselves by writing their IP address into the PLC array on startup. No manual configuration needed in the PLC.
- **Single active screen** — only one panel at a time can send commands; others are read-only.
- **Take-over by request** — a view-only panel presses "I want control", the active screen gets a countdown and can confirm or deny.
- **Block take-over** — the active screen can block take-over requests. Optional password protection via `PassWordActive` input.
- **Default IP positions** — up to 4 IP addresses can be pinned to fixed slot numbers (`DefaultIpNo1..4`), so panels always get the same slot regardless of boot order.
- **Screen alive monitoring** — the PLC detects when a panel goes offline and removes it from the list.
- **Auto switch** — optional automatic screen switching after a configurable time (`AutoSwitchTime`).
- **Mirror screen** — a panel can toggle mirror mode (e.g. port/starboard orientation).

### Interface

#### Inputs

| Name | Type | Description |
|------|------|-------------|
| `Reset` | `BOOL` | Resets all screen registrations |
| `AutoSwitchTime` | `INT` | Seconds before automatic screen switch (0 = disabled) |
| `DefaultIpNo1` | `STRING` | IP address that is always placed at slot 1 |
| `DefaultIpNo2` | `STRING` | IP address that is always placed at slot 2 |
| `DefaultIpNo3` | `STRING` | IP address that is always placed at slot 3 |
| `DefaultIpNo4` | `STRING` | IP address that is always placed at slot 4 |
| `PassWordActive` | `BOOL` | Enables the "Block take-over" button on the active screen |

#### InOut

| Name | Type | Description |
|------|------|-------------|
| `SwitchScreenInOut` | `ARRAY[1..4] OF ScreenSwitch` | Shared data array — one element per HMI panel |

#### Outputs

| Name | Type | Description |
|------|------|-------------|
| `ScreenActive` | `INT` | Slot number of the currently active screen (1–4, 0 = none) |
| `WantsControl` | `INT` | Slot number of the screen requesting take-over |
| `PressedWhileBlocked` | `BOOL` | One-cycle pulse when a panel presses take-over while blocked |

### Program sections

| Section | Description |
|---------|-------------|
| `Info` | Documentation, HMI script code, changelog |
| `CheckDefaultIps` | Moves panels with default IPs to their fixed slot |
| `ScreenAlive` | Monitors heartbeat counters, removes offline panels |
| `SwitchLogic` | Core take-over logic and screen switching |
| `PressWhileBlocked` | Detects press while take-over is blocked |
| `ReleaseBlocked` | Updates block-button availability based on password |
| `BlockedToOtherScreens` | Propagates block status to all connected panels |
| `SomethingWrong` | Watchdog — resets stuck bits and clears impossible states |

---

## SwitchScreenMemmory (v0.13) — Companion block

Solves one specific problem: all panels run the same application, but some settings are **physical location specific** (e.g. port/starboard mirror orientation). These settings must survive an HMI reboot — so they are stored in the PLC, tied to the panel's IP address, not its slot number.

### How it works

Bits 9–14 of `ScreenSwitchButtons.Wrd` are the memory bits. This block maintains a shadow copy in `MemoryWrd[1..4]` per IP address:

- **SAVE:** after a panel has been online for 2.5 seconds → `MemoryWrd[n] := Buttons.Wrd AND mask(bits 9-14)`
- **RESTORE:** on rising edge of panel coming online → `Buttons.Wrd := Buttons.Wrd OR MemoryWrd[n]`

The 2.5s delay is intentional — it ensures the RESTORE completes before SAVE happens, so the stored values are not overwritten with zeros on reconnect.

### Available memory bits

| Bit | Name | Status | Description |
|-----|------|--------|-------------|
| 9 | `MirrorScreen` | In use | Screen orientation (0=port, 1=starboard) |
| 10 | `res10` | Free | Available for custom per-screen setting |
| 11 | `res11` | Free | Available for custom per-screen setting |
| 12 | `res12` | Free | Available for custom per-screen setting |
| 13 | `res13` | Free | Available for custom per-screen setting |
| 14 | `res14` | Free | Available for custom per-screen setting |

To use a free bit: add a button in the HMI + display logic. No PLC changes needed — the bit is automatically saved and restored.

### Interface

#### Inputs

| Name | Type | Description |
|------|------|-------------|
| `DefaultIpNo1` | `STRING` | IP address mapped to `MemoryWrd[1]` |
| `DefaultIpNo2` | `STRING` | IP address mapped to `MemoryWrd[2]` |
| `DefaultIpNo3` | `STRING` | IP address mapped to `MemoryWrd[3]` |
| `DefaultIpNo4` | `STRING` | IP address mapped to `MemoryWrd[4]` |

#### InOut

| Name | Type | Description |
|------|------|-------------|
| `SwitchScreenInOut` | `ARRAY[1..4] OF ScreenSwitch` | **Same array** as SwitchScreens FB |
| `MemoryWrd` | `ARRAY[1..4] OF WORD` | Persistent storage — **declare as RETAIN!** |

> ⚠️ `MemoryWrd` must be declared as **RETAIN** in the PLC program, otherwise settings are lost on power cycle.

### Program sections

| Section | Description |
|---------|-------------|
| `Info` | This documentation |
| `ReadMirrorButton` | Toggles `MirrorScreen` (bit 9) on `BtnMirrorScreen` (bit 8) press |
| `DefaultScreenOnline` | Save/restore memory bits per IP address |

---

## Shared DDTs

Both blocks use the same DDTs (bundled in each `.xdb`):

| DDT | Description |
|-----|-------------|
| `ScreenSwitch` | Per-panel struct: IP address, active screen, countdown, buttons |
| `ScreenSwitchButtons` | WORD with 16 named button/status bits |

---

## How to import

Import both blocks separately:

1. **File → Import → Import DFB...** → select `switchscreens.xdb`
2. **File → Import → Import DFB...** → select `switchscreenmemmory.xdb`

> DDTs only need to be imported once — Control Expert will skip duplicates if versions match.

---

## HMI scripts (Vijeo Designer)

Five periodic scripts (0.5s interval) run identically on every panel.  
Each script uses `Sys.getInfoString(IP_ADDRESS_1)` to identify the panel by its own IP address.

| Action | Description |
|--------|-------------|
| Action 1 | Register panel IP in PLC array |
| Action 2 | Increment heartbeat counter (alive monitoring) |
| Action 3 | Handle take-over request |
| Action 4 | Handle Yes/No confirmation |
| Action 5 | Show/hide control elements based on active screen |

---

## Changelog

### SwitchScreens
| Version | Date | Notes |
|---------|------|-------|
| 1.47 | 2026-05-13 | Current version |
| 1.46 | — | Previous stable release |
| — | 2021-03-15 | Array pointer fix — prevents %S20 alarms |
| — | 2020-10-28 | Block take-over + password feature added |
| — | 2020-10-27 | Default IP address inputs added |

### SwitchScreenMemmory
| Version | Date | Notes |
|---------|------|-------|
| 0.13 | 2026-05-13 | Current version |
| 0.11 | 2023-12-01 | Initial release |
