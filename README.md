# ControlExpert-DFBs

A personal library of **Derived Function Blocks (DFBs)** for Schneider Electric **Control Expert V14.1**.  
These blocks are used in real industrial installations — mainly maritime and process automation.

---

## 📦 Available DFBs

| DFB | Version | Description |
|-----|---------|-------------|
| [Emotor](./Emotor/) | 0.84 | Electric motor control with sequenced starting, alarms, and PMS integration |

---

## 🚀 How to import a DFB

1. Open your project in **Control Expert**
2. Go to **File → Import → Import DFB...**  
   *(or right-click in the project tree → Import)*
3. Select the `.xdb` file from the DFB folder
4. All DDTs and the DFB are imported in one go

> ⚠️ If shared DDTs like `General` already exist in your project, Control Expert may warn about version conflicts. Always check before overwriting.

---

## 🗂️ Repository structure

```
ControlExpert-DFBs/
├── Emotor/
│   ├── emotor.xdb          ← Import this into Control Expert
│   └── README.md           ← Emotor documentation
└── README.md               ← This file
```

---

## 🔧 Platform

| | |
|---|---|
| **Software** | Schneider Electric Control Expert V14.1 |
| **PLC hardware** | Tested on BME P58 / BMX P34 series |
| **Languages used** | Structured Text (ST) + Ladder (LD) |
| **Diagnostics** | UREGDFB compatible (`IsTypeDiagnostic = TRUE`) |

---

*By [kaswets](https://github.com/kaswets) — arjan swets*
