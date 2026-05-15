# PasswordGenerator — Dynamic Password DFB

**Version:** 1.82  
**Platform:** Schneider Electric Control Expert V14.1  
**Last modified:** 2025-08-15

---

## Description

Function block that generates a **dynamic, time-based password** for HMI access control.  
Developed for Van Aalst Marine & Offshore installations.

Instead of a fixed password, the HMI shows a random **Password ID** each session. The correct password is calculated from that ID using a formula known only to authorised personnel. This means:
- The password changes every session
- Screenshots or notes of a password cannot be reused
- Three access levels: Operator, Operator 2, and System

---

## Access levels

| Level | Output | Description |
|-------|--------|-------------|
| **Operator** | `OprPWOkay` | Standard operator access — fixed password (`PassWordNormal`) |
| **Operator 2** | `OprPW2Okay` | Extended operator access — second fixed password (`PassWordExtra`), resets on manual mode exit |
| **System** | `SysPWOkay` | Service/engineer access — dynamic password calculated from `HmiID` |

---

## Interface

### Inputs

| Name | Type | Description |
|------|------|-------------|
| `Maal` | `INT` | Multiplier used in password calculation (`HmiID × Maal`) |
| `Aantal` | `INT` | Number of characters to extract from the result string |
| `Vanaf` | `INT` | Position from the end of the string to start extracting |
| `PassWordNormal` | `STRING` | Fixed password for Operator level access |
| `PassWordExtra` | `STRING` | Fixed password for Operator 2 level access |
| `ResetTime` | `TIME` | Timeout — password resets after this time of inactivity |
| `ResetSpecialPW` | `BOOL` | Force reset of Operator 2 password |

### InOut

| Name | Type | Description |
|------|------|-------------|
| `hmi` | `SystemPassWord` | HMI password exchange struct — contains ID, entered password, and status bits |
| `general` | `General` | Shared system struct |

### Outputs

| Name | Type | Description |
|------|------|-------------|
| `SysPWOkay` | `BOOL` | System password accepted |
| `OprPWOkay` | `BOOL` | Operator password accepted |
| `OprPW2Okay` | `BOOL` | Operator 2 password accepted |
| `ActSysPW` | `STRING` | The current correct system password (for commissioning/service use) |

---

## How the dynamic password works

1. On startup (or reset), a random **HmiID** is generated using 6 chained `Random` blocks seeded with PLC system clock values (`%SW18`, `%SW50`, `%SW51`)
2. The `HmiID` is shown on the HMI
3. The system calculates: `result = HmiID × Maal`
4. Converts result to string, then extracts `Aantal` characters starting `Vanaf` positions from the end
5. This extracted string is the correct system password
6. The operator enters a password on the HMI → if it matches `ActSysPW`, `SysPWOkay` goes TRUE

**Example** with `Maal=7`, `Aantal=4`, `Vanaf=2`:
- `HmiID = 12345`
- `12345 × 7 = 86415`
- String: `"86415"` → extract 4 chars from position 3 from end → `"6415"`... wait, `Vanaf=2` means starting 2 from end → `"641"` (4 chars from pos 3) — exact result depends on string length

---

## Password reset conditions

The password resets automatically when:
- `ResetTime` expires without screen activity (`ScreenIsTouched = FALSE`)
- The HMI screen changes (`ScreenIsChanged`)
- PLC first scan (`%S13`)
- `hmi.Reset` is set from HMI
- Operator 2 password exits manual mode (`fe(InManualMode)`)
- `ResetSpecialPW` rising edge

---

## Program sections

| Section | Description |
|---------|-------------|
| `PasswordProgram` | Main logic — calculates password, checks entries, handles reset |
| `RandomGenerator` | Generates random HmiID using 6 `Random` blocks + PLC clock XOR |

---

## Included DFBs and DDTs

| Name | Type | Description |
|------|------|-------------|
| `PasswordGenerator` | DFB | Main password block |
| `Random` | DFB | Helper — generates pseudo-random number 1–10 using `mod((7 × n), 11)` |
| `SystemPassWord` | DDT | HMI password exchange struct |
| `General` | DDT | Shared system struct |
| `manualWrd` | DDT | Manual control word |

---

## How to import

1. Open your project in Control Expert
2. **File → Import → Import DFB...**
3. Select `passwordgenerator.xdb`
4. All DDTs and the `Random` helper are imported automatically

---

## Changelog

| Version | Date | Notes |
|---------|------|-------|
| 1.82 | 2025-08-15 | Current version |
