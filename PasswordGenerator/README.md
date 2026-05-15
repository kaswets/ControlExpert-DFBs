# PasswordGenerator — Dynamic Password DFB

**Version:** 1.82  
**Platform:** Schneider Electric Control Expert V14.1  
**Last modified:** 2025-08-15

---

## Description

Function block that generates a **dynamic, session-based password** for HMI access control.  
Developed for Van Aalst Marine & Offshore installations.

The HMI shows a random **Password ID** each session. To get the correct password, the operator must contact B&T — who apply a formula to the ID and provide the code. This means:
- The password changes every session
- A code can only be used once — leaving the settings page generates a new ID
- The operator can never calculate the code themselves
- B&T always knows when settings access has been requested

---

## Access levels

| Level | Output | Description |
|-------|--------|-------------|
| **Operator** | `OprPWOkay` | Standard operator access — fixed password (`PassWordNormal`) |
| **Operator 2** | `OprPW2Okay` | Extended operator access — second fixed password (`PassWordExtra`), resets on manual mode exit |
| **System** | `SysPWOkay` | Service/engineer access — dynamic password calculated from `HmiID` |

---

## How it works in practice

1. Operator needs access to settings → HMI shows a random **Password ID**
2. Operator calls B&T and reads out the ID
3. B&T applies the formula → gives the operator the code
4. Operator enters the code on the HMI → `SysPWOkay` goes TRUE
5. Operator makes the changes needed
6. As soon as the operator leaves the settings page → new random ID is generated → old code is invalid

> 🔒 The formula is not documented here. Contact B&T for service access.

---

## Interface

### Inputs

| Name | Type | Description |
|------|------|-------------|
| `Maal` | `INT` | Multiplier used in password calculation |
| `Aantal` | `INT` | Number of characters to extract from the result |
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
| `ActSysPW` | `STRING` | The current correct system password — visible in PLC for commissioning use only |

---

## Password reset conditions

The password resets automatically when:
- `ResetTime` expires without screen activity (`ScreenIsTouched = FALSE`)
- The HMI screen changes (`ScreenIsChanged`)
- PLC first scan (`%S13`)
- `hmi.Reset` is set from HMI
- Operator 2 exits manual mode
- `ResetSpecialPW` rising edge

---

## Program sections

| Section | Description |
|---------|-------------|
| `PasswordProgram` | Main logic — calculates password, checks entries, handles reset |
| `RandomGenerator` | Generates random HmiID using 6 `Random` blocks seeded with PLC clock and uptime values |

---

## Included DFBs and DDTs

| Name | Type | Description |
|------|------|-------------|
| `PasswordGenerator` | DFB | Main password block |
| `Random` | DFB | Helper — pseudo-random number generator (see `_Helpers/Random/`) |
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
