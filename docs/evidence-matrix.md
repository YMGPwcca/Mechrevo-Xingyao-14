# Evidence matrix

Validation status for machine-specific technical claims.

## Platform

| Claim | Status | Evidence |
|---|---|---|
| Product/platform is MECHREVO Xingyao 14 / `P916F-STX` | **Live-confirmed** | Firmware/OS identity |
| CPU is AMD Ryzen AI 9 365 | **Platform-confirmed** | AMD official model naming |
| iGPU is Radeon 880M | **Platform/live-confirmed** | AMD specification and machine observation |
| Memory is 32 GiB on the documented unit | **Live-confirmed** | OS observation |
| Internal panel is 2880×1800 | **Live-confirmed** | Display observation |
| BIOS is 1.15 | **Live-confirmed** | Firmware UI |
| EC version reported by firmware is 1.15 | **Live-confirmed** | Firmware UI |
| EC silicon is ITE `0x5571` rev `0x07` | **Live-confirmed** | Super-I/O configuration space at 0x4E |

## BIOS and boot graphics

| Claim | Status | Evidence |
|---|---|---|
| BIOS 1.15 contains an 800×600 animated GIF | **Static-confirmed** | Exact BIOS/ROM extraction |
| GIF has 60 frames and ~1.74 s duration | **Static-confirmed** | Extracted resource inspection |
| Animation resource GUID is `931F77D1-10FE-48BF-AB72-773D389E3FAA` | **Static-confirmed** | Firmware object analysis |
| `OemBadgingSupportDxe` is associated with the resource | **Static-confirmed** | Firmware module/resource analysis |
| Linux BGRT xoffset/yoffset is 1040/387 | **Live-confirmed** | ACPI BGRT sysfs data |
| Generic H2OFFT `-edt4f` maps to IHISI type `0x54` | **Comparative only** | Generic Insyde H2OFFT/IHISI analysis |
| Exact P916F `ChipsetSvcSmm` callback rejects type `0x54` | **Static-confirmed** | P916F BIOS disassembly |
| Generic `-logoupdate` / type `0x6D` expects GUID `DACFAB69-F977-4784-8AD8-7724A6F4B440` | **Comparative only** | Generic Insyde logo-update analysis |
| Required type-0x6D target is present in Windows ESRT | **Rejected** | Absent in live ESRT inspection |
| Required type-0x6D target is present in raw-ROM FDM | **Rejected** | Absent from 47-entry HFDM table at raw-ROM offset `0x1D7C000` |
| A working logo-only update path is established for BIOS 1.15 | **Not established** | Standard type-0x54 and type-0x6D paths are unavailable |

## SetupUtility

| Claim | Status | Evidence |
|---|---|---|
| SetupUtility GUID is `FE3542FE-C1D3-4EF8-657C-8048606FF670` | **Static-confirmed** | BIOS 1.15 firmware analysis |
| Boot formset GUID is `2D068309-12AC-45AB-9600-9187513CCDD8` | **Static-confirmed** | IFR analysis |
| Quiet Boot QuestionId is `0x1064` | **Static-confirmed** | IFR analysis |
| Quiet Boot is `SystemConfig + 0x6E` | **Static-confirmed** | IFR analysis |
| Quiet Boot values are 0=Disabled / 1=Enabled | **Static-confirmed** | IFR analysis |
| `Setup[0x6E] = 0x01` was observed | **Live-confirmed** | Runtime setup-variable read |
| Suppressed Boot form can be exposed at runtime with SREP | **Live-confirmed** | Successful runtime patch and visible hidden Boot form |
| Permanent patch at SetupUtility PE offset ~`0x2636A0` is validated | **Not tested** | Static landmark only |
| `Dynamic LID` maps to `AMD_PBS_SETUP + 0xDF`, default 0 | **Static-confirmed** | Firmware form analysis |
| User-facing behavior of Dynamic LID is known | **Not established** | No live behavior mapping |

## EC image and H2RAM

| Claim | Status | Evidence |
|---|---|---|
| Raw ROM is 32 MiB, SHA-256 `77043505b6f42e4a482110a7ba0c7e12ba6b1db28fdaed2743c28578bbf76cd7` | **Artifact-confirmed** | Exact dump/hash |
| Preferred EC carve is raw-ROM offset `0x081000`, length `0x20000` | **Static-confirmed** | ROM carve and code/data inspection |
| EC carve SHA-256 is `42c117f00c130c5e533be93ee1657401ac4d687255ed1b2250f74d3cc79397ea` | **Artifact-confirmed** | Exact hash |
| EC code is MCS-51/8051-family | **Static-confirmed** | Reset/vector/opcode structure |
| H2RAM maps host `0xFEEC2300..0xFEEC23FF` to EC XRAM `0x0300..0x03FF` | **Static-confirmed + live-correlated** | DSDT, EC initialization and live MMIO |
| EC `0x0394` is the SOC value used by charge logic | **Static-confirmed + live-correlated** | Charge comparisons and MMIO cross-check |

## PMC2

| Claim | Status | Evidence |
|---|---|---|
| ITE configuration port is 0x4E | **Live-confirmed** | Valid chip ID only at 0x4E |
| PMC2 LDN is 0x12 and active | **Live-confirmed** | Super-I/O configuration read |
| PMC2 data port is 0x68 | **Live-confirmed** | Super-I/O configuration and working transactions |
| PMC2 command/status port is 0x6C | **Live-confirmed** | Super-I/O configuration and working transactions |
| OBF bit0 / IBF bit1 transaction flow works | **Live-confirmed** | Successful battery command transactions |

## Battery charge limit

| Claim | Status | Evidence |
|---|---|---|
| `0x0D01.bit4` is enable/state | **Static-confirmed + live-correlated** | EC handlers and state query |
| `0x0D13` is threshold #1 | **Static-confirmed + live set/readback** | EC handlers and `F2` / `F1 13` |
| `0x0D14` is threshold #2 | **Static-confirmed + live set/readback** | EC handlers and `F3` / `F1 14` |
| Threshold setters accept 0..100 inclusive | **Static-confirmed** | Exact 8051 range-check logic |
| `F1 11` enables the subsystem | **Live-confirmed** | State changed 0 -> 1 |
| `F1 12` reads state | **Live-confirmed** | Coherent 0/1 responses |
| `F1 13` reads T1 | **Live-confirmed** | Readback 0 then 80 |
| `F1 14` reads T2 | **Live-confirmed** | Readback 0 then 100 |
| `F2 80` sets T1 to 80 | **Live-confirmed** | Response/readback 80 |
| `F3 100` sets T2 to 100 | **Live-confirmed** | Response/readback 100 |
| `F1 10` disable/reset clears enable and both thresholds | **Static-confirmed only** | EC reset handler |
| `T1=80,T2=100` caps charging around 80% | **Live-confirmed** | Charging/no-charging transition |
| Arbitrary `T1=N,T2=100` produces an N% cap | **Not established** | Only 80/100 behaviorally validated |
| Exact semantic role of T2 is known | **Not established** | Control-flow participation is known; user-facing semantics are not |
| Exact hysteresis width is known | **Not established** | Linux SOC display is integer-rounded |
| State persists across normal reboot | **Live-confirmed** | Post-reboot readback 1/80/100 |
| State persists across complete EC power loss | **Not established** | Not tested |

## Power path

| Claim | Status | Evidence |
|---|---|---|
| AC can be online while battery reports `Not charging`, `power_now=0` | **Live-confirmed** | Sysfs measurement |
| Battery supplied net energy during a five-minute full-CPU load | **Live-confirmed** | `energy_now` decreased 63.154 Wh -> 62.661 Wh |
| The exact electrical power-path mechanism is fully characterized | **Not established** | Charger topology not fully decoded |

## Rejected charge-control paths

| Path | Status | Evidence |
|---|---|---|
| Huawei threshold GET `0x1103` | **Rejected** | Live failure/unsupported result |
| Huawei SET `0x1003` | **Not tested** | Write avoided after GET failed |
| Generic Uniwill offsets `0x07B9/0x07D0` as the P916F path | **Rejected as P916F evidence** | Generic multi-model software only; exact P916F firmware uses another subsystem |
| `INOU0000` / `ECRR` / `ECRW` | **Absent** | Not present in P916F ACPI tables |
| Dedicated I2EC at base `0x380` | **Rejected** | Read-only cross-check returned `0xFF` while MMIO returned valid SOC |
| ACPI `_BTP` as charge cap | **Rejected** | `_BTP` is the ACPI battery trip-point mechanism |

## Audio

| Claim | Status | Evidence |
|---|---|---|
| Internal codec is Realtek ALC256 | **Live-confirmed** | ALSA enumeration |
| Linux exposes stereo FL/FR | **Live-confirmed** | PipeWire/WirePlumber enumeration |
| Linux exposes a separate LFE/four-channel speaker endpoint | **Rejected** | Not present in the logical audio topology |
| Exact OEM Nahimic/A-Volute DSP profile is recovered | **Not established** | OEM coefficients/configuration not recovered |
