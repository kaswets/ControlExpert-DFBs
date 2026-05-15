# SwitchScreens — Multi-HMI Screen Control DFB

**Version:** 1.47  
**Platform:** Schneider Electric Control Expert V14.1  
**HMI platform:** Vijeo Designer  
**Last modified:** 2026-05-13

---

## Description

Function block that manages up to **4 identical HMI panels** connected to a single PLC.  
All panels run the same Vijeo Designer application — the panel's own IP address is the only identifier.

Only **one panel at a time** has control (the active screen). The others are in view-only mode.  
Any view-only panel can request a take-over via a confirmation dialog (Yes/No).

---

## Key features

- **Self-registration** — panels register themselves by writing their IP address into the PLC array on startup. No manual configuration needed in the PLC.
- **Single active screen** — only one panel at a time can send commands; others are read-only.
- **Take-over by request** — a view-only panel presses "I want control", the active screen gets a countdown and can confirm or deny.
- **Block take-over** — the active screen can block take-over requests from other panels. Optional password protection via `PassWordActive` input.
- **Default IP positions** — up to 4 IP addresses can be pinned to fixed slot numbers (`DefaultIpNo1..4`), so panels always get the same slot regardless of boot order.
- **Screen alive monitoring** — the PLC detects when a panel goes offline and removes it from the list.
- **Auto switch** — optional automatic screen switching after a configurable time (`AutoSwitchTime`).
- **Mirror screen** — a panel can request mirror mode (view-only copy of active screen).

---

## Interface

### Inputs

| Name | Type | Description |
|------|------|-------------|
| `Reset` | `BOOL` | Resets all screen registrations |
| `AutoSwitchTime` | `INT` | Seconds before automatic screen switch (0 = disabled) |
| `DefaultIpNo1` | `STRING` | IP address that is always placed at slot 1 |
| `DefaultIpNo2` | `STRING` | IP address that is always placed at slot 2 |
| `DefaultIpNo3` | `STRING` | IP address that is always placed at slot 3 |
| `DefaultIpNo4` | `STRING` | IP address that is always placed at slot 4 |
| `PassWordActive` | `BOOL` | Enables the "Block take-over" button on the active screen |

### InOut

| Name | Type | Description |
|------|------|-------------|
| `SwitchScreenInOut` | `ARRAY[1..4] OF ScreenSwitch` | Shared data array — one element per HMI panel |

### Outputs

| Name | Type | Description |
|------|------|-------------|
| `ScreenActive` | `INT` | Slot number of the currently active screen (1–4, 0 = none) |
| `WantsControl` | `INT` | Slot number of the screen requesting take-over |
| `PressedWhileBlocked` | `BOOL` | One-cycle pulse when a panel presses take-over while blocked |

---

## ScreenSwitch struct

Each element of `SwitchScreenInOut` is a `ScreenSwitch` struct:

| Field | Type | Description |
|-------|------|-------------|
| `ConnectedScreen` | `STRING[16]` | IP address of the registered HMI panel |
| `ActiveScreen` | `INT` | Number of the currently active screen |
| `SwitchScreenCnt` | `INT` | Countdown counter shown during take-over |
| `Buttons` | `ScreenSwitchButtons` | Button and status bits (see below) |

### ScreenSwitchButtons (WORD with named bits)

| Bit | Name | Description |
|-----|------|-------------|
| 0 | `BtnYes` | Panel pressed Yes (confirm take-over) |
| 1 | `BtnNo` | Panel pressed No (deny take-over) |
| 2 | `BtnWantControl` | Panel requests take-over |
| 3 | `OtherScreenWantsControl` | Notification: another panel wants control |
| 4 | `BtnTakeOverNotAllowed` | Active screen pressed "Block take-over" |
| 5 | `BlockBtnTakeOverNotAllowed` | Blocks the block-button (password logic) |
| 6 | `pressedTakeOverNotAllowedWhileBlocked` | Pressed while blocked (triggers `PressedWhileBlocked` output) |
| 7 | `TakeOverIsBlocked` | Take-over is currently blocked |
| 8 | `btnMirrorScreen` | Panel requests mirror mode |
| 9 | `mirrorScreen` | Mirror mode active |
| 15 | `ThisScreenIsAlive` | Heartbeat bit — toggled by HMI every 0.5s |

---

## How it works (take-over sequence)

1. View-only panel presses **"I want control"** → `BtnWantControl` goes high
2. PLC sets `OtherScreenWantsControl` on all other panels → countdown starts
3. Active screen can press **Yes** or **No** (or ignore — countdown runs out)
4. On Yes (or timeout): active screen changes, new panel gets control
5. On No: request is cancelled

---

## HMI scripts (Vijeo Designer)

Five periodic scripts run on each panel (all identical across panels):

| Action | Interval | Description |
|--------|----------|-------------|
| Action 1 | 0.5s | Register this panel's IP address in the PLC array |
| Action 2 | 0.5s | Increment heartbeat counter (alive monitoring) |
| Action 3 | 0.5s | Handle take-over request logic |
| Action 4 | 0.5s | Handle Yes/No confirmation |
| Action 5 | 0.5s | Show/hide control elements based on active screen |

All scripts use `Sys.getInfoString(IP_ADDRESS_1)` to identify the panel by its own IP address.

---

## Program sections inside the DFB

| Section | Language | Description |
|---------|----------|-------------|
| `Info` | ST | Documentation, HMI script code, changelog |
| `CheckDefaultIps` | ST | Moves panels with default IPs to their fixed slot |
| `ScreenAlive` | ST | Monitors heartbeat counters, removes offline panels |
| `SwitchLogic` | ST | Core take-over logic and screen switching |
| `PressWhileBlocked` | ST | Detects press while take-over is blocked |
| `ReleaseBlocked` | ST | Updates block-button availability based on password |
| `BlockedToOtherScreens` | ST | Propagates block status to all connected panels |
| `SomethingWrong` | ST | Watchdog — resets stuck bits and clears impossible states |

---

## Included DDTs

| DDT | Description |
|-----|-------------|
| `ScreenSwitch` | Per-panel data struct (IP, active screen, countdown, buttons) |
| `ScreenSwitchButtons` | WORD with 16 named button/status bits |

---

## How to import

1. Open your project in Control Expert
2. **File → Import → Import DFB...**
3. Select `switchscreens.xdb`
4. `ScreenSwitch` and `ScreenSwitchButtons` DDTs are imported automatically

---

## Changelog

| Version | Date | Notes |
|---------|------|-------|
| 1.47 | 2026-05-13 | Current version |
| 1.46 | — | Previous stable release |
| — | 2021-03-15 | Array pointer fix to prevent %S20 alarms |
| — | 2020-10-28 | Block take-over + password feature added |
| — | 2020-10-27 | Default IP address inputs added |
