# _Helpers

Small utility DFBs used internally by other function blocks in this library.  
These blocks are not intended to be used standalone — they are imported automatically as part of their parent DFB.

---

## Available helpers

| DFB | Version | Description | Included in |
|-----|---------|-------------|-------------|
| [BitCheck](./BitCheck/) | 0.11 | Splits a WORD into 16 individual BOOLs + counts active bits | `DValve`, `CommunicationAerzen` |
| [MODBUS_Shift](./Modbus_Shift/) | 0.07 | Splits a 16-bit INT into high byte and low byte for Modbus frames | `CommunicationAerzen` |
| [RotateValve](./RotateValve/) | 0.07 | Packs a rotation angle (0–360°) into the DValve HMI status word | `DValve` |

---

## How to import

These helpers are bundled inside their parent DFB's `.xdb` file and imported automatically.  
If you need a helper as a standalone block, export it separately from Control Expert:

1. Right-click the DFB in the project tree
2. **Export → Export DFB...**
3. Save as `.xdb`

---

*By [kaswets](https://github.com/kaswets) — arjan swets*
