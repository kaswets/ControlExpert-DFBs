# ControlExpert-DFBs

A personal library of **Derived Function Blocks (DFBs)** for Schneider Electric **Control Expert V14.1**.  
These blocks are used in real industrial installations — mainly maritime and process automation.

---

## 📦 Available DFBs

| DFB | Version | Description |
|-----|---------|-------------|
| [CommunicationAerzen](./CommunicationAerzen/) | 0.53 | Modbus TCP communication with Aerzen compressors (Aertronic + WEG VFD) |
| [Emotor](./Emotor/) | 0.84 | Electric motor control with sequenced starting, alarms, and PMS integration |
| [SwitchScreens](./SwitchScreens/) | 1.47 | Multi-HMI screen control — up to 4 identical Vijeo Designer panels sharing one PLC |

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
│   ├── communicationaerzen.xdb ← Import this into Control Expert
│   └── README.md               ← CommunicationAerzen documentation
├── Emotor/
│   ├── emotor.xdb              ← Import this into Control Expert
│   └── README.md               ← Emotor documentation
├── SwitchScreens/
│   ├── switchscreens.xdb       ← Import this into Control Expert
│   ├── switchscreenmemmory.xdb ← Companion block
│   └── README.md               ← SwitchScreens documentation
└── README.md                   ← This file
```

---

## 🔧 Platform

| | |
|---|---|
| **Software** | Schneider Electric Control Expert V14.1 |
| **PLC hardware** | Tested on BME P58 2040 (M580) / BMX P34 2020 (M340) |
| **HMI** | Vijeo Designer (for SwitchScreens) |
| **Languages used** | Structured Text (ST) + Ladder (LD) |

---

*By [kaswets](https://github.com/kaswets) — arjan swets*
