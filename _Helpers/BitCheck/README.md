# BitCheck — Word to Bits Helper DFB

**Version:** 0.11  
**Platform:** Schneider Electric Control Expert V14.1  
**Last modified:** 2026-05-11

---

## Description

Utility block that splits a single **WORD** into 16 individual **BOOL** outputs — one per bit.  
Also counts how many bits are active (`NumberOffBits`) and provides the raw word as `ValueOut`.

Used internally by `DValve` and other DFBs to process status words and system group assignments.

---

## Interface

### Input

| Name | Type | Description |
|------|------|-------------|
| `WrdIn` | `WORD` | Input word to split into bits |

### Outputs

| Name | Type | Description |
|------|------|-------------|
| `ValueOut` | `WORD` | Mirror of the input word |
| `Bit0`–`Bit15` | `BOOL` | Individual bits of the input word |
| `NumberOffBits` | `INT` | Count of bits that are 1 |
| `Even` | `BOOL` | Reserved — not yet implemented |

---

## How it works

Copies each bit from `WrdIn` directly to the corresponding `BitN` output.  
Then counts the number of active bits by shifting the word right 16 times and checking bit 0 each time.  
A jump-to-end safeguard prevents the loop from hanging if `Count` stops incrementing.

---

## Used in

| DFB | Purpose |
|-----|---------|
| `DValve` | Splits `System` word into group bits, splits `General.Manual.wrd` into manual group bits |
| `CommunicationAerzen` | — |

---

## How to import

This block is included in `dvalve.xdb` and imported automatically with `DValve`.  
For standalone use, export it separately from Control Expert as its own `.xdb`.
