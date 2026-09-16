# Embedded-controller architecture (ITE IT5571)

## Scope and source coverage

This page describes the `P916F-STX` embedded controller, its host-visible windows and the static charge-control landmarks recovered from the investigated firmware. Machine-specific live results and the historical traces are retained from `SRC-BASELINE` ([S1](research-sources.md#project-sources)); the ACPI field and method excerpts are selected source material identified as `SRC-AML-A` through `SRC-AML-D` ([S3](research-sources.md#project-sources), [S4](research-sources.md#project-sources), [S5](research-sources.md#project-sources)). The independently measured package and carve metadata is `SRC-BINARIES`/`SRC-SFX` ([S9](research-sources.md#project-sources)); artifact identity and extraction coverage are recorded in [research artifacts](research-artifacts.md#current-raw-32-mib-rom). A source excerpt or artifact identity establishes a static property; it does not by itself establish a live setter or a safe write procedure.

## Silicon and host interfaces

The `P916F-STX` exposes an ITE Super-I/O / embedded controller. Live configuration-space probing found:

| Item | Recorded value | Evidence |
|---|---:|---|
| Working ITE configuration port | I/O `0x4E` | Live-confirmed |
| Chip ID | `0x5571` | Live-confirmed |
| Revision | `0x07` | Live-confirmed |
| Alternate configuration port | I/O `0x2E` returned `0xFFFF` | Live negative result |
| PMC2 logical device | `0x12` | Live configuration-space result |
| PMC2 active | `0x01` | Live-confirmed |
| PMC2 I/O #0 | `0x0068` | Data port; live-confirmed |
| PMC2 I/O #1 | `0x006C` | Command/status port; live-confirmed |
| PMC2 I/O #2 | `0x0000` | Recorded configuration |
| PMC2 IRQ | `0x00` | Recorded configuration |

Three interfaces must remain distinct:

| Interface | Host address space | Recorded role |
|---|---|---|
| Standard ACPI EC | I/O `0x62` / `0x66` | Conventional AML EC transactions |
| H2RAM | Physical memory `0xFEEC2300..0xFEEC23FF` | SystemMemory-backed shared EC state |
| ITE PMC2 | I/O `0x68` / `0x6C` | Command transport used by the battery charge limiter |

The working PMC2 roles are:

```text
PMC2 DATA            = 0x68
PMC2 COMMAND/STATUS  = 0x6C
```

The PMC2 result independently validates the host interface later used successfully for battery-limit commands. An ACPI-table search does not substitute for live Super-I/O discovery, and the later PMC2 finding superseded the earlier ACPI-only assumption that `0x68`/`0x6C` was irrelevant.

## EC firmware image and address spaces

The current 32 MiB ROM contains a banked IT557x firmware image. The preferred carved image is:

```text
source ROM:  P916F-STX-current-ROM.bin
ROM offset:  0x081000
length:      0x20000 bytes (128 KiB)
filename:    P916F-IT5571-EC-1.09.bin
SHA-256:     42c117f00c130c5e533be93ee1657401ac4d687255ed1b2250f74d3cc79397ea
```

The first 64 KiB bank is densely populated. The second bank is sparser but contains real code and data, so the final carve remains a full 128 KiB image rather than an unrelated second EC blob. The parent raw-ROM identity and the earlier updater-image extraction are separate artifact records; do not substitute the updater's `0x18000`-byte extraction for this raw-ROM carve.

The MCS-51 reset entry starts with:

```text
02 00 70    LJMP 0x0070
```

Strings in the image include:

```text
ITE EC-V14.6
IT557x V1.09 E00 - 20230831
ITE Tech. Inc.
AMD Motherboard
MECHREVO
VER:01.0F.00
```

These are internal EC build strings. They must not be confused with the separate firmware UI `EC 1.15` version label. The EC is banked: a logical `CODE:` address requires bank context before it can be mapped to a byte offset in a firmware container.

## H2RAM mapping

The DSDT declares:

```asl
OperationRegion (ERAM, SystemMemory, 0xFEEC2300, 0x100)
```

Static analysis of the exact EC firmware found a configuration sequence around `CODE:0xDD50`:

```asm
MOV  DPTR,#0x105B
MOV  A,#0x30
MOVX @DPTR,A

MOV  DPTR,#0x105D
MOV  A,#0x04
MOVX @DPTR,A

MOV  DPTR,#0x105A
MOV  A,#0x01
MOVX @DPTR,A
RET
```

This configures an H2RAM window whose EC-side base is `0x0300` and whose host-visible size matches the 256-byte ACPI region. The established mapping is:

```text
host physical 0xFEEC2300 + N  <->  EC XRAM 0x0300 + N
N = 0x00..0xFF
```

This relationship is supported by the DSDT, the EC H2RAM initialization code and live MMIO reads that returned plausible changing battery/EC state. It describes only the exposed 256-byte aperture, not a generic mapping of all EC XRAM into host memory.

## Battery percentage anchor

The firmware repeatedly references:

```text
EC XRAM[0x0394]
```

as a battery-percentage/SOC value in charge-control comparisons. Because `0x0394` lies in the `0x0300..0x03FF` H2RAM window, the corresponding host physical address is:

```text
0xFEEC2394
```

A live `/dev/mem` read at that address returned a normal battery percentage value, for example `92` during one later cross-check, rather than the `0xFF` observed through the rejected I2EC hypothesis.

## ACPI field map

Offsets below are within the documented EC/H2RAM field window. Multi-byte names retain the AML spelling; meanings should be derived from their consumers rather than guessed solely from abbreviations.

| Window offset | Field(s) | Width or detail |
|---|---|---|
| `0x30` | ACIN, ACLW | Bits 6 and 7 |
| `0x31` | PBST, LIDS, PBS2 | Bits 0, 1 and 6 |
| `0x32` | TPST, KBFN | Bits 0 and 1 |
| `0x33` | KBBL | 8 bits |
| `0x34` | WINS, CPLS | Bits 0 and 1 |
| `0x35` | KBST | 16 bits |
| `0x3B` | FNS0 | 16 bits; fan-telemetry field |
| `0x3D` | FNS1 | 16 bits; fan-telemetry field |
| `0x3F` | FTVL | 8 bits; profile state |
| `0x70` | APFL, MSFL | Bits 0 and 1 |
| `0x80` | BATI, BAII, BACG, BAIC | Bits 0 through 3 |
| `0x81` | BFCL/BFCH | Two 8-bit fields |
| `0x83` | BRML/BRMH | Two 8-bit fields |
| `0x85` | BDCL/BDCH | Two 8-bit fields |
| `0x87` | BMNF | 8 bits |
| `0x88` | BVLL/BVLH | Two 8-bit fields |
| `0x8A` | BCRL/BCRH | Two 8-bit fields |
| `0x8C` | BDVL/BDVH | Two 8-bit fields |
| `0x8E` | BACL/BACH | Two 8-bit fields |
| `0x90` | BTPL/BTPH | ACPI battery trip-point fields |
| `0x92` | SRNM | 16 bits |
| `0x94` | RSOL/RSOH | Two 8-bit fields; SOC anchor at first byte |
| `0xA0` | BDVN | 128-bit battery/model-string region |
| `0xCF` | STAS | 8 bits |
| `0xFE` | TEMP | 8 bits |

The `_BTP` method writes `BTPL/BTPH`. That is the ACPI battery trip-point/notification facility, not the charge limiter's `XRAM[0x0D13]` and `XRAM[0x0D14]`, which lie outside the exposed `0x0300..0x03FF` window.

## Charge-control subsystem

The dedicated battery-limit state is:

```text
XRAM[0x0D01].bit4   enable/state bit
XRAM[0x0D13]        threshold value 1
XRAM[0x0D14]        threshold value 2
XRAM[0x0394]        live SOC used by the decision logic
```

Important handler locations include:

```text
CODE:0xED60...      enable/status family
CODE:0xED7A         sets XRAM[0x0D01].bit4
CODE:0xED8E         checks XRAM[0x0D01].bit4
CODE:0xEDBA         validates/sets XRAM[0x0D13]
CODE:0xEDDF         validates/sets XRAM[0x0D14]
CODE:0xF508         disable/reset path; clears bit4 and both threshold fields
CODE:0xF526         read helper for XRAM[0x0D13]
CODE:0xF621         read helper for XRAM[0x0D14]
CODE:0xC063         core SOC/threshold decision routine
CODE:0xF4D8         recorded PMC2 parser landmark
CODE:0xE65D         recorded PMC2 data-out helper landmark
```

The public command protocol that reaches this family is documented in [battery-charge-limit.md](battery-charge-limit.md).

The `CODE:0xF508` disable/reset association is static-confirmed only; the corresponding live reset operation was not exercised. The setter landmarks have static range checks, while live validation is limited to the recorded `T1=80%`, `T2=100%` setter/readback sequence described in [battery-charge-limit.md](battery-charge-limit.md).


### Threshold validation logic

Both threshold setters use the same 8051 range check before storing the value:

```asm
MOV  A,value
CLR  C
SUBB A,#0
JC   invalid

MOV  A,value
SETB C
SUBB A,#0x64
JNC invalid
```

On the 8051, `SUBB` computes `A - operand - C`. Because the second comparison deliberately sets carry first, a value of exactly 100 still produces a borrow and remains valid, while 101 does not and is rejected. The firmware therefore accepts the inclusive decimal range 0 through 100, encoded as `0x00` through `0x64`. This does not prove how every possible threshold pair maps to user-facing charging behavior.

### Charger-control working values

The threshold decision routine feeds working words around:

```text
XRAM[0x0D54..0x0D57]
XRAM[0x0D65..0x0D68]
```

Helpers around `CODE:0xC249` and `CODE:0xC27A` clear or copy these values depending on the SOC/threshold branch taken. A later worker stages charger transactions associated with command numbers `0x14` and `0x15`. Those numbers are consistent with common Smart Battery charger `ChargingCurrent` / `ChargingVoltage` conventions, but the exact semantic naming remains inferred until the complete transaction path is decoded end to end. These words must not be presented as a complete electrical power-path model.

## PMC2 host transport

Live Super-I/O configuration proves that logical device `0x12` is active at data I/O `0x68` and command/status I/O `0x6C`. The successful battery-limit experiments used:

```text
bit0 OBF  = output buffer full / EC has a response byte
bit1 IBF  = input buffer full / host must wait before sending another byte
```

The recorded host sequence was:

1. Wait for `IBF=0`.
2. Write the command byte to command/status I/O `0x6C`.
3. Wait for `IBF=0` again.
4. Write the command's data/subcommand byte to data I/O `0x68`.
5. For a returning command, wait for `OBF=1` and read `0x68`.

The complete timeout, stale-output, error and transaction-ownership contract was not retained. A known command path must not be replaced with an unrestricted raw-memory write.

## Thermal-profile interface boundary

The recovered AML exposes `FNS0`, `FNS1` and `FTVL` through `GFNS` and `GPFM`; `SPFM` invokes thermal handling and then dispatches profile-specific EC commands. Their exact source-level behavior is documented in [thermal/performance interfaces](thermal-performance.md).

A complete EC tachometer-register map, manual-PWM state model and target-RPM table extraction were not recovered with sufficient primary evidence. No direct PWM or fan-mode write procedure is established here. Static AML access to telemetry is not equivalent to validation of a raw XRAM setter, and ALIB parameters must not be treated as RPM tables.

## Rejected dedicated I2EC candidate

Static EC code writes I2EC-related interface registers including:

```text
XRAM[0x200D]
XRAM[0x2012]
XRAM[0x2014]
XRAM[0x2015]
```

One initialization sequence led to a hypothesis that a dedicated four-port I2EC window might exist at base `0x380`. A **read-only** live test selected EC addresses through candidate ports `0x381`/`0x382` and read `0x383`. Every tested target returned `0xFF`:

```text
EC[200D] = 0xFF
EC[0394] = 255
EC[0D13] = 255
EC[0D14] = 255
```

At the same time, the known host MMIO battery byte returned:

```text
FEEC2394 = 92
```

The cross-check therefore failed decisively. The static initialization also writes `XRAM[0x2012] = 0x20`; the dedicated-port enable interpretation used during the investigation places the enable on another bit, so the candidate path was not shown to be live-enabled.

**Conclusion:** the tested `0x380` route is rejected for the recorded stock configuration as a usable I2EC path. This does not prove the absence of every factory or debug transport.

## Access and safety boundaries

- Do not raw-write unknown values into the `0xFEEC2300` MMIO aperture.
- Do not enable `ec_sys` write support as a shortcut for charge control.
- Do not brute-force PMC2 commands.
- Do not copy `0x07B9`, `0x07D0` or other generic Uniwill/Tongfang offsets onto this machine merely because the EC family is similar.
- Do not equate a logical query with an absence of bus writes: selecting a register or submitting a getter can still write address/command ports.
- Prefer the exact firmware-defined PMC2 command path when a known command exists.

The charge-limit work demonstrates why: generic Windows software contained familiar old offsets, but the actual P916F-STX firmware exposes a distinct command-controlled subsystem at `0x0D13`/`0x0D14`.
