# MODBUS_Shift — Modbus Byte Splitter Helper DFB

**Version:** 0.07  
**Platform:** Schneider Electric Control Expert V14.1  
**Last modified:** 2025-11-05

---

## Description

Utility block that splits a 16-bit **INT** value into a **high byte** and a **low byte**.  
Used to build Modbus `DATA_EXCH` frames where address and data values need to be split across multiple WORD fields.

---

## Interface

### Input

| Name | Type | Description |
|------|------|-------------|
| `In` | `INT` | 16-bit value to split |

### Outputs

| Name | Type | Description |
|------|------|-------------|
| `out1` | `INT` | High byte — bits 15–8 (upper 8 bits) |
| `out2` | `INT` | Low byte — bits 7–0 (lower 8 bits) |

---

## How it works

```
out1 := In AND 2#1111111100000000   (high byte)
out2 := In AND 2#0000000011111111   (low byte)
```

Simple AND mask — no shifting. The high and low bytes are kept in their original bit positions, ready to be combined with other values using OR in the `DATA_EXCH` frame.

---

## Used in

| DFB | Purpose |
|-----|---------|
| `CommunicationAerzen` | Splits Modbus register address and RPM setpoint for `DATA_EXCH` Modbus FC06 frame |

---

## How to import

This block is included in `communicationaerzen.xdb` and imported automatically with `CommunicationAerzen`.  
For standalone use, export it separately from Control Expert as its own `.xdb`.
