# _ReloaderSystems

DFBs for pneumatic bulk cement reloader systems as supplied by VAMO (Van Aalst Marine & Offshore).  
Each subfolder contains a complete system variant with its own DFBs and documentation.

---

## System overview

A reloader system pneumatically transports bulk cement from a ship's cargo hold to shore (or hold-to-hold).  
The system uses vacuum to suck cement into pressure vessels, which are then pressurised to push the cement out.

Three system variants are available, differing in number of vessels and filter location:

| System | Vessels | Filter location | DFBs needed |
|--------|---------|-----------------|-------------|
| [2-Vessel](./ReloaderSet_2Vessel/) | 2 pressure vessels | Inside each pressure vessel (top) | `ReloaderSet` |
| [3-Vessel](./ReloaderSet_3Vessel/) | 1 vacuum vessel + 2 pressure vessels | Inside vacuum vessel | `ReloaderSet` + `VacVessel` |
| 5-Vessel | 1 vacuum vessel + 4 pressure vessels | Inside vacuum vessel | — |

---

## How the systems work

### 2-Vessel system
Two pressure vessels alternate — while one is being filled via vacuum suction (filter inside the vessel catches dust), the other is pressurised and discharging cement. Simple and reliable, slightly lower throughput due to brief stops during switchover.

### 3-Vessel system
A dedicated vacuum vessel sits on top of two pressure vessels. The vacuum vessel continuously sucks cement and feeds it into whichever pressure vessel has room — cement falls by gravity via a fill valve (V10/V20), while a separate vent valve (V12/V22) routes escaping air back up into the vacuum vessel just below the filter, where it is immediately drawn away by the vacuum pump. This separation of cement flow and air flow is essential — without it, cement and air would block each other in the same opening. The vacuum vessel acts as a buffer, giving higher throughput than the 2-vessel system.

### 5-Vessel system
Similar principle to the 3-vessel system but with four pressure vessels under the vacuum vessel — allowing even higher continuous discharge rates.

---

## Common DFBs used in all systems

These DFBs are used internally by the reloader systems and must be present in the project:

| DFB | Description |
|-----|-------------|
| `Reloader` | Individual pressure vessel control (state machine per vessel) |
| `DValve` | Digital valve control — used for fill, vent, equalizer and hold valves |
| `Purge` / `Filterpulse` | Filter cleaning pulse control |
| `Analog` | Pressure sensor reading and scaling |

---

*By [kaswets](https://github.com/kaswets) — arjan swets / B&T*
