# ControlExpert-DFBs

A personal library of **Derived Function Blocks (DFBs)** for Schneider Electric **Control Expert V14.1**.  
These blocks are used in real industrial installations — mainly maritime and process automation.

---

## 📦 Available DFBs

| DFB | Version | Description |
|-----|---------|-------------|
| [CommunicationAerzen](./CommunicationAerzen/) | 0.53 | Modbus TCP communication with Aerzen compressors (Aertronic + WEG VFD) |
| [DValve](./DValve/) | 2.42 | Digital valve control — normally closed, with feedback, alarms, manual override and HMI status |
| [Emotor](./Emotor/) | 0.84 | Electric motor control with sequenced starting, alarms, and PMS integration |
| [PasswordGenerator](./PasswordGenerator/) | 1.82 | Dynamic session-based password system for HMI access control |
| [SwitchScreens](./SwitchScreens/) | 1.47 | Multi-HMI screen control — up to 4 identical Vijeo Designer panels sharing one PLC |

---

## 🔧 Helper DFBs

Small utility blocks used internally by the DFBs above. See [_Helpers](./_Helpers/) for details.

| DFB | Version | Description | Used in |
|-----|---------|-------------|---------|
| [BitCheck](./_Helpers/BitCheck/) | 0.11 | Splits a WORD into 16 BOOLs + counts active bits | DValve, CommunicationAerzen |
| [LookForBitChange](./_Helpers/LookForBitChange/) | 0.09 | Detects rising and falling edges on all 16 bits of a WORD | — |
| [MODBUS_Shift](./_Helpers/Modbus_Shift/) | 0.07 | Splits a 16-bit INT into high and low byte for Modbus frames | CommunicationAerzen |
| [Random](./_Helpers/Random/) | 0.02 | Pseudo-random number generator 1–10 | PasswordGenerator |
| [RotateValve](./_Helpers/RotateValve/) | 0.07 | Packs rotation angle (0–360°) into DValve HMI status word | DValve |

---

## 🚀 How to import a DFB

1. Open your project in **Control Expert**
2. Go to **File → Import → Import DFB...**  
   *(or right-click in the project tree → Import)*
3. Select the `.xdb` file from the DFB folder
4. All DDTs and the DFB are imported in one go

> ⚠️ If shared DDTs already exist in your project, Control Expert may warn about version conflicts. Always check before overwriting.

---

## 🗂️ Repository structure

```
ControlExpert-DFBs/
├── CommunicationAerzen/
│   ├── communicationaerzen.xdb
│   └── README.md
├── DValve/
│   ├── dvalve.xdb
│   └── README.md
├── Emotor/
│   ├── emotor.xdb
│   └── README.md
├── PasswordGenerator/
│   ├── passwordgenerator.xdb
│   └── README.md
├── SwitchScreens/
│   ├── switchscreens.xdb
│   ├── switchscreenmemmory.xdb
│   └── README.md
├── _Helpers/
│   ├── BitCheck/
│   ├── LookForBitChange/
│   ├── Modbus_Shift/
│   ├── Random/
│   ├── RotateValve/
│   └── README.md
└── README.md
```

---

## 🔧 Platform

| | |
|---|---|
| **Software** | Schneider Electric Control Expert V14.1 |
| **PLC hardware** | Tested on BME P58 2040 (M580) / BMX P34 2020 (M340) |
| **HMI** | Vijeo Designer (for SwitchScreens, PasswordGenerator) |
| **Languages used** | Structured Text (ST) + Ladder (LD) |

---

*By [kaswets](https://github.com/kaswets) — arjan swets*
