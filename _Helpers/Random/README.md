# Random — Pseudo-Random Number Helper DFB

**Version:** 0.02  
**Platform:** Schneider Electric Control Expert V14.1  
**Last modified:** 2020-05-07

---

## Description

Generates a pseudo-random number between **1 and 10** using a simple mathematical formula.  
Used internally by `PasswordGenerator` to build a random Password ID each session.

The algorithm is a basic **linear congruential generator** — multiply by 7, modulo 11.  
This cycles through all values 1–10 without repeating before the full cycle completes.

---

## Interface

### InOut

| Name | Type | Description |
|------|------|-------------|
| `random` | `UDINT` | Seed value in, next pseudo-random value out |

> ⚠️ `random` is **InOut** — the output overwrites the input each call. The caller must store the value and pass it back next call to continue the sequence.

---

## How it works

```pascal
random := mod((7 * random), 11);
```

Multiplying by 7 modulo 11 produces a full cycle through all values 1–10:

```
seed=1 → 7 → 5 → 2 → 3 → 10 → 4 → 6 → 9 → 8 → 1 → ...
```

> ⚠️ Never seed with 0 — `mod(0, 11) = 0` forever. `PasswordGenerator` initialises seeds to non-zero values on first scan.

---

## Used in

| DFB | Purpose |
|-----|---------|
| `PasswordGenerator` | Six instances (`Random_getal1`–`Random_getal6`) feed into the Password ID calculation |

---

## How to import

This block is included in `passwordgenerator.xdb` and imported automatically with `PasswordGenerator`.  
For standalone use, export it separately from Control Expert as its own `.xdb`.
