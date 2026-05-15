# RotateValve — Valve Rotation Helper DFB

**Version:** 0.07  
**Platform:** Schneider Electric Control Expert V14.1  
**Last modified:** 2021-08-18

---

## Description

Utility block that packs a valve rotation **angle** into the HMI status word of `DValve`.  
The angle (0–360 degrees) is shifted into bits 5–13 of the status word so the HMI can visually rotate the valve symbol to the correct position.

Previously valves could only be rotated in steps of 90 degrees. This block allows any angle up to 360 degrees — useful for slide valves, butterfly valves, and any valve that needs a non-standard visual orientation.

---

## Interface

### Input

| Name | Type | Description |
|------|------|-------------|
| `angle` | `WORD` | Rotation angle in degrees (0–360) |

### InOut

| Name | Type | Description |
|------|------|-------------|
| `Status` | `WORD` | DValve HMI status word — angle is OR'd into bits 5–13 |

---

## How it works

The angle value is shifted left by 5 bits (`SHL_WORD`) and then OR'd into the `Status` word.  
This places the angle into bits 5–13, leaving bits 0–4 free for the valve status flags (open, moving, alarm, manual).

```
Status := Status OR (angle << 5)
```

---

## DValve status word layout

| Bit | Description |
|-----|-------------|
| 0 | Open (green) / Closed (grey) |
| 1 | Moving (yellow) |
| 2 | Alarm (red) |
| 3 | Manual active |
| 4 | Manual button state |
| **5–13** | **Valve angle (set by RotateValve)** |

---

## Used in

| DFB | Purpose |
|-----|---------|
| `DValve` | Sets visual rotation of the valve symbol on the HMI |

---

## How to import

This block is included in `dvalve.xdb` and imported automatically with `DValve`.  
For standalone use, export it separately from Control Expert as its own `.xdb`.
