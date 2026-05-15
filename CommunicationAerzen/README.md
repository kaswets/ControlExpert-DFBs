# CommunicationAerzen — Modbus Communication DFB

**Version:** 0.53  
**Platform:** Schneider Electric Control Expert V14.1  
**Last modified:** 2026-05-04

---

## Description

Function block for Modbus TCP communication with **Aerzen** bulk compressor units.  
Supports two device types on separate IP addresses:

- **Aertronic** — Aerzen's own controller (write: setpoint RPM via `DATA_EXCH`)
- **WEG VFD** — WEG frequency drive (read: status registers via `READ_VAR`)

All Aerzen machines in the installation use the same Modbus register addresses — so this one DFB covers all of them. If register addresses ever change, only this block needs to be updated.

> Originally the M580 PLC supports Modbus connections via the DTM browser, but that requires a lot of manual configuration per device. This DFB handles it programmatically — just pass the IP address and it works.

---

## Interface

### Inputs

| Name | Type | Description |
|------|------|-------------|
| `Adress_Aertronic` | `STRING[40]` | IP address of the Aertronic controller (e.g. `'192.168.4.10'`) |
| `Adress_Weg` | `STRING[40]` | IP address of the WEG frequency drive |
| `run` | `BOOL` | Enable communication — when FALSE, all data is cleared |
| `WantedRPM` | `INT` | Setpoint RPM to write to the Aertronic |

### Outputs

| Name | Type | Description |
|------|------|-------------|
| `cntMessage` | `INT` | Current Modbus read counter (0 to NumberToRead) |
| `Active` | `BOOL` | Communication active (mirrors `run` input) |
| `Data` | `ARRAY[0..10] OF INT` | Read data — one register per index |

---

## Modbus registers read (WEG VFD)

| Index | Register | Description |
|-------|----------|-------------|
| `Data[0]` | %MW2 | — |
| `Data[1]` | %MW3 | — |
| `Data[2]` | %MW10 | — |
| `Data[3]` | %MW3046 | — |
| `Data[4]` | %MW5000 | — |

> Register descriptions to be filled in based on WEG VFD documentation.

## Modbus register written (Aertronic)

| Register | Description |
|----------|-------------|
| %MW600 | Wanted RPM setpoint |

---

## How it works

The DFB cycles through the Modbus registers one at a time using `READ_VAR`:

1. Reads register `modbusAdress[cntMessage]` from the WEG VFD
2. Stores result in `Data[cntMessage]`
3. Increments `cntMessage` — when all registers are read, wraps back to 0
4. If a read returns 0, the previous value is kept (`PrevData`)

Writing the RPM setpoint to the Aertronic happens via two methods:
- **`WRITE_VAR`** — standard Modbus write to %MW600
- **`DATA_EXCH`** — single register write using Modbus function 06 (via `MODBUS_Shift` helper), triggered every 0.5s pulse (`%S5`)

Writing only happens when `WantedRPM` changes and the WEG has confirmed communication is OK (`ComWithThisIpOkay`).

A watchdog timer (`TonReset`, 500ms) detects stuck `WRITE_VAR` calls and resets them automatically.

---

## Program sections

| Section | Description |
|---------|-------------|
| `info` | Comments and documentation |
| `Adressen` | Converts IP strings to `ADDM_TYPE`, sets Modbus register addresses |
| `ActiveOrNot` | Sets `active` flag from `run` input |
| `ReadModbus` | Cycles through register reads via `READ_VAR` |
| `DataWissen` | Clears all data when `run` goes FALSE |
| `ResetWriteVar` | Watchdog — resets stuck `WRITE_VAR` after 500ms |
| `WriteModbus` | Writes RPM setpoint via `WRITE_VAR` |
| `WriteModbusSingle` | Writes RPM setpoint via `DATA_EXCH` (Modbus FC06) |

---

## Included DFBs

This `.xdb` also contains the `MODBUS_Shift` helper block:

### MODBUS_Shift (v0.07)

Splits a 16-bit INT into high byte and low byte for use in Modbus `DATA_EXCH` frames.

| Pin | Type | Description |
|-----|------|-------------|
| `In` | `INT` | Input value to split |
| `out1` | `INT` | High byte (bits 15–8) |
| `out2` | `INT` | Low byte (bits 7–0) |

---

## How to import

1. Open your project in Control Expert
2. **File → Import → Import DFB...**
3. Select `communicationaerzen.xdb`
4. Both `CommunicationAerzen` and `MODBUS_Shift` are imported in one step

---

## Changelog

| Version | Date | Notes |
|---------|------|-------|
| 0.53 | 2026-05-04 | Current version |
