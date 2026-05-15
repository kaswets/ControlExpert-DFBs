# LookForBitChange — Bit Edge Detection Helper DFB

**Version:** 0.09  
**Platform:** Schneider Electric Control Expert V14.1  
**Last modified:** 2022-09-01

---

## Description

Utility block that detects **rising and falling edges** on all 16 bits of a WORD simultaneously.  

Where a standard `R_TRIG` or `F_TRIG` only works on a single BOOL, this block handles an entire WORD in one call — useful when multiple HMI buttons or status bits are packed into a single WORD and you need to know which ones changed this scan cycle.

---

## Interface

### Inputs

| Name | Type | Description |
|------|------|-------------|
| `ChoiceWrd` | `WORD` | Current value of the word this scan cycle |
| `PreChoiceWrd` | `WORD` | Value of the word from the previous scan cycle |

### InOut

| Name | Type | Description |
|------|------|-------------|
| `ChangeOnWrd` | `WORD` | Bits that turned ON this cycle (0→1) are set to 1 |
| `ChangeOffWrd` | `WORD` | Bits that turned OFF this cycle (1→0) are set to 1 |

> ⚠️ `ChangeOnWrd` and `ChangeOffWrd` are **InOut** — the caller is responsible for clearing them after processing. The block only **sets** bits, never clears them.

---

## How it works

For each of the 16 bits:
- If `ChoiceWrd.N = 1` and `PreChoiceWrd.N = 0` → rising edge → `ChangeOnWrd.N := 1`
- If `ChoiceWrd.N = 0` and `PreChoiceWrd.N = 1` → falling edge → `ChangeOffWrd.N := 1`

The caller must:
1. Call `LookForBitChange` each scan cycle
2. Process `ChangeOnWrd` and `ChangeOffWrd`
3. Clear the change words after processing
4. Copy `ChoiceWrd` to `PreChoiceWrd` for the next cycle

---

## Example usage

```pascal
(* Each scan cycle: *)
LookForBitChange(
    ChoiceWrd    := HMI_ButtonWord,
    PreChoiceWrd := Prev_ButtonWord,
    ChangeOnWrd  := ButtonPressed,
    ChangeOffWrd := ButtonReleased
);

(* Process changes *)
IF ButtonPressed.3 THEN
    (* Button 3 was just pressed *)
    ButtonPressed.3 := FALSE;  (* clear after processing *)
END_IF;

(* Save current for next cycle *)
Prev_ButtonWord := HMI_ButtonWord;
```

---

## How to import

1. Open your project in Control Expert
2. **File → Import → Import DFB...**
3. Select `lookforbitchange.xdb`

---

## Changelog

| Version | Date | Notes |
|---------|------|-------|
| 0.09 | 2022-09-01 | Current version |
