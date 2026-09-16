# Evidence matrix

This file is the quickest way to distinguish **what is proven**, **what is only static**, **what is inferred**, and **what has been rejected**.

## Platform identity

| Claim | Status | Evidence |
|---|---|---|
| Machine is MECHREVO Xingyao 14 / `P916F-STX` | **Live-confirmed** | Firmware/OS identity observed on the researched unit |
| Official AMD CPU model is Ryzen AI 9 365 | **Externally confirmed / platform-confirmed** | AMD official model naming; Radeon 880M pairing matches this SKU |
| iGPU is Radeon 880M | **Live/platform-confirmed** | Machine/platform observation; matches AMD's official Ryzen AI 9 365 specification |
| RAM is 32 GiB on this unit | **Live-confirmed** | OS observation |
| Internal panel is 2880×1800 | **Live-confirmed** | Prior display observation; also consistent with BGRT placement |
| Internal panel is 1920×1080@144 | **Rejected / wrong context** | This value came from another machine/context and was removed from the P916F docs |
| BIOS is 1.15 | **Live-confirmed** | Firmware UI |
| EC version shown by firmware UI is 1.15 | **Live-confirmed** | Firmware UI |
| BIOS build-date string is `05/07/2026` | **Live-confirmed** | Raw firmware UI string; date-order interpretation intentionally not forced |
| EC silicon is ITE 0x5571 rev 0x07 | **Live-confirmed** | ITE config space at port 0x4E |

## BIOS / boot graphics

| Claim | Status | Evidence |
|---|---|---|
| BIOS 1.15 contains an 800×600 animated GIF | **Static-confirmed** | Exact BIOS/ROM extraction |
| GIF has 60 frames and ~1.74 s duration | **Static-confirmed** | Extracted resource inspection |
| Animation resource GUID is `931F77D1-10FE-48BF-AB72-773D389E3FAA` | **Static-confirmed** | Firmware object analysis |
| `OemBadgingSupportDxe` is associated with the animation | **Static-confirmed** | Firmware module/resource analysis |
| Linux BGRT xoffset/yoffset is 1040/387 after BIOS 1.15 | **Live-confirmed** | `/sys/firmware/acpi/bgrt` observation |
| 800-pixel image is horizontally centered on 2880-wide panel | **Corroborating inference** | `1040 + 800 + 1040 = 2880` |
| Insyde H2OFFT generic `-edt4f` maps to logo-update type `0x54` | **Generic mechanism confirmed; comparative context** | H2OFFT/IHISI analysis |
| P916F `ChipsetSvcSmm` callback at RVA `0x221C` handles type `0x50` and returns `EFI_UNSUPPORTED` for other types including `0x54` | **Static-confirmed on exact P916F BIOS** | Exact `ChipsetSvcSmm` disassembly |
| P916F has a working raw-logo Type-54 writer | **Rejected** | Exact chipset callback rejects `0x54`; no project-specific writer found |
| Generic `-logoupdate` / type `0x6D` expects target GUID `DACFAB69-F977-4784-8AD8-7724A6F4B440` | **Generic mechanism confirmed; comparative context** | Insyde logo-update path analysis |
| Windows ESRT on this machine contains `DACFAB69...` | **Rejected / absent in observed ESRT** | Live Windows ESRT inspection |
| Raw-ROM FDM contains `DACFAB69...` | **Rejected / absent** | `HFDM` at raw-ROM offset `0x1D7C000`, 47 entries scanned |
| A provisioned logo-only update mechanism is known on P916F-STX BIOS 1.15 | **Not found / currently rejected for the two standard Insyde paths** | Type54 rejected; Type6D target region absent |
| Boot logo can be safely replaced without firmware-region rewrite | **Not established** | No enabled/provisioned runtime-only path found |

## Hidden / revealed setup findings

| Claim | Status | Evidence |
|---|---|---|
| SetupUtility GUID is `FE3542FE-C1D3-4EF8-657C-8048606FF670` | **Static-confirmed** | BIOS 1.15 firmware analysis |
| Boot formset GUID is `2D068309-12AC-45AB-9600-9187513CCDD8` | **Static-confirmed** | IFR analysis |
| Quiet Boot QuestionId is `0x1064` | **Static-confirmed** | IFR analysis |
| Quiet Boot `SystemConfig` offset is `0x6E` | **Static-confirmed** | IFR analysis |
| Quiet Boot values are 0=Disabled / 1=Enabled | **Static-confirmed** | IFR analysis |
| `Setup[0x6E]` was `0x01` on the researched machine | **Live-confirmed** | Runtime setup-variable read during BIOS investigation |
| Hidden Boot form could be exposed at runtime with SREP on the researched machine | **Live-confirmed** | Smokeless Runtime EFI Patcher reported successful search/patch and subsequent BIOS Boot-menu photo shows normally hidden options |
| A permanent firmware binary patch at SetupUtility PE offset ~`0x2636A0` was live-tested | **Not tested** | Candidate static suppression landmark only |
| `Dynamic LID` is at `AMD_PBS_SETUP + 0xDF`, default 0 | **Static-confirmed** | Firmware form analysis |
| Dynamic LID means "open lid to power on" | **Not established** | No live behavior mapping |

## EC image and H2RAM

| Claim | Status | Evidence |
|---|---|---|
| Current raw ROM is 32 MiB, SHA-256 `770435...cd7` | **Artifact-confirmed** | Exact dump/hash |
| Preferred EC carve is raw-ROM offset `0x081000`, length `0x20000` | **Static-confirmed** | ROM carve and code/data inspection |
| Preferred EC carve SHA-256 is `42c117...97ea` | **Artifact-confirmed** | Exact hash |
| EC code is MCS-51/8051-family | **Static-confirmed** | Reset/vector/opcode structure |
| H2RAM maps host `0xFEEC2300..23FF` to EC `0x0300..03FF` | **Static-confirmed + live-correlated** | DSDT region + EC config code + live MMIO values |
| EC `0x0394` is a SOC/percentage value used by charging logic | **Static-confirmed + live-correlated** | Charge logic comparisons + live host MMIO value |

## PMC2

| Claim | Status | Evidence |
|---|---|---|
| ITE config port is 0x4E | **Live-confirmed** | 0x2E returned 0xFFFF; 0x4E returned chip ID |
| PMC2 LDN is 0x12 and active | **Live-confirmed** | Super-I/O config read |
| PMC2 data port is 0x68 | **Live-confirmed** | Super-I/O config read + working transactions |
| PMC2 command/status port is 0x6C | **Live-confirmed** | Super-I/O config read + working transactions |
| OBF bit0 / IBF bit1 transaction flow works | **Live-confirmed** | Successfully used for F1/F2/F3 command family |

## Battery charge limit

| Claim | Status | Evidence |
|---|---|---|
| `0x0D01.bit4` is enable/state | **Static-confirmed + host-state correlated** | EC handlers + F1 state query |
| `0x0D13` is threshold #1 | **Static-confirmed + live set/readback** | EDBA/F526 + F2/F1-13 |
| `0x0D14` is threshold #2 | **Static-confirmed + live set/readback** | EDDF/F621 + F3/F1-14 |
| Threshold setters accept 0..100 inclusive | **Static-confirmed** | Exact 8051 `SUBB` range-check logic |
| `F1 11` enables the subsystem | **Live-confirmed** | state changed 0 -> 1 |
| `F1 12` reads state | **Live-confirmed** | coherent 0/1 responses |
| `F1 13` reads T1 | **Live-confirmed** | readback 0 then 80 |
| `F1 14` reads T2 | **Live-confirmed** | readback 0 then 100 |
| `F2 80` sets T1 to 80 | **Live-confirmed** | response/readback 80 |
| `F3 100` sets T2 to 100 | **Live-confirmed** | response/readback 100 |
| `F1 10` clears/disable-resets | **Static-confirmed only** | F508 clears enable bit + thresholds; not exercised live in the documented test |
| `T1=80,T2=100` caps charge around 80% | **Live-confirmed** | AC-connected transition to `Not charging`, plus lower-SOC recharge behavior |
| Any `T1=N,T2=100` gives an N% cap | **Not established** | Only 80/100 tested |
| Exact meaning of T2 is known | **Not established** | It is consumed by control logic, but user-facing semantics are not fully mapped |
| Exact hysteresis width is known | **Not established** | Linux `capacity` is integer-rounded; observed transition only around displayed 79–80% |
| State persists across normal reboot | **Live-confirmed** | post-reboot GET returned 1/80/100 |
| State persists across complete EC power loss | **Not established** | not tested |

## Power-path behavior

| Claim | Status | Evidence |
|---|---|---|
| At cap, AC can be online while battery reports `Not charging`, `power_now=0` | **Live-confirmed** | sysfs snapshot |
| Battery supplied energy during a five-minute full-CPU load | **Live-confirmed** | `energy_now` fell 63.154 Wh -> 62.661 Wh |
| Platform uses battery-assist/hybrid power under heavy load | **Inferred** | best explanation of energy drop + subsequent recharge; charger topology not fully decoded |

## Paths rejected or superseded

| Hypothesis/path | Status | Reason |
|---|---|---|
| Huawei charge-threshold GET `0x1103` | **Rejected for this feature** | live result returned unsupported/failure |
| Huawei SET `0x1003` | **Not tested intentionally** | GET failed, so write path was avoided |
| Generic Uniwill `0x07B9/0x07D0` as the P916F path | **Rejected as P916F evidence** | found only in generic multi-model software; exact P916F firmware exposes another subsystem |
| `INOU0000` / `ECRR` / `ECRW` on this laptop | **Absent** | P916F ACPI tables were searched and did not expose them |
| Dedicated I2EC at base `0x380` | **Rejected / superseded** | read-only live cross-check returned all 0xFF while MMIO returned real SOC |
| ACPI `_BTP` as charge cap | **Rejected semantic interpretation** | `_BTP` is the standard battery trip-point mechanism, not charger limit |

## Audio

| Claim | Status | Evidence |
|---|---|---|
| Internal codec path includes Realtek ALC256 Analog | **Live-confirmed** | `aplay -l` / ALSA observation |
| Linux exposes stereo FL/FR only | **Live-confirmed** | `wpctl status` |
| Linux exposes a separate LFE/subwoofer channel | **Rejected** | no such channel in live PipeWire graph |
| Ubuntu Live reproduces the weak/odd speaker sound | **Live-confirmed** | user test |
| Missing OEM Nahimic/A-Volute tuning is the primary cause | **Strong inference** | Windows-vs-Linux behavior + basic codec path works; exact OEM DSP coefficients not recovered |

## How to use this matrix

When a future document contradicts this table:

1. prefer newer live evidence over older assumptions;
2. prefer the exact P916F ROM over generic Control Center constants;
3. keep static code facts separate from behavioral semantics;
4. record failed hypotheses instead of deleting their history;
5. update both the detailed document and this matrix when new evidence changes confidence.
