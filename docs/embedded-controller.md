# Embedded controller (ITE IT5571)

## Live silicon and Super-I/O identity

The `P916F-STX` exposes an ITE Super-I/O / embedded controller.

Live configuration-space probing found:

| Item | Value | Confidence |
|---|---:|---|
| ITE config port | `0x4E` | Live-confirmed |
| Chip ID | `0x5571` | Live-confirmed |
| Revision | `0x07` | Live-confirmed |
| PMC2 logical device | `0x12` | Live-confirmed |
| PMC2 active | `0x01` | Live-confirmed |
| PMC2 I/O #0 | `0x0068` | Live-confirmed |
| PMC2 I/O #1 | `0x006C` | Live-confirmed |
| PMC2 I/O #2 | `0x0000` | Live-confirmed |
| PMC2 IRQ | `0x00` | Live-confirmed |

The usual ITE configuration port at `0x2E` returned `0xFFFF` on this machine. The working config interface is `0x4E`.

The PMC2 result matters because it independently validates the host interface later used successfully for battery-limit commands:

```text
PMC2 DATA            = 0x68
PMC2 COMMAND/STATUS  = 0x6C
```

## EC firmware provenance

The current 32 MiB ROM contains a banked IT557x firmware image.

Preferred carved image:

```text
source ROM:  P916F-STX-current-ROM.bin
ROM offset:  0x081000
length:      0x20000 bytes (128 KiB)
filename:    P916F-IT5571-EC-1.09.bin
SHA-256:     42c117f00c130c5e533be93ee1657401ac4d687255ed1b2250f74d3cc79397ea
```

The first 64 KiB bank is densely populated. The second bank is much sparser but contains real code/data, which is why the final carve is kept as the full 128 KiB image rather than treating the later region as an unrelated second EC blob.

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

These are internal EC build strings and should not be confused with the laptop firmware UI's separate `EC 1.15` version number.

## H2RAM / host-visible shared memory

The DSDT declares:

```text
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

This configures an H2RAM window whose EC-side base is `0x0300` and whose host-visible size matches the 256-byte ACPI region.

The established mapping is therefore:

```text
host physical 0xFEEC2300 + N  <->  EC XRAM 0x0300 + N
```

for the exposed `0x00..0xFF` window.

This mapping is supported by both the DSDT and the EC's own H2RAM configuration code, and live MMIO reads returned plausible changing battery/EC state.

## Battery percentage anchor

The firmware repeatedly references:

```text
EC XRAM[0x0394]
```

as a battery-percentage/SOC value in charge-control comparisons.

Because `0x0394` lies inside the `0x0300..0x03FF` H2RAM window, the corresponding host address is:

```text
0xFEEC2394
```

A live `/dev/mem` read at that address returned a normal battery percentage value (for example `92` during one later cross-check), rather than the `0xFF` seen through the rejected I2EC hypothesis.

## ACPI-visible battery field map

The DSDT's EC field layout includes:

```text
0x30: ACIN bit6, ACLW bit7

0x80: BATI bit0, BAII bit1, BACG bit2, BAIC bit3

0x81: BFCL/BFCH
0x83: BRML/BRMH
0x85: BDCL/BDCH
0x87: BMNF
0x88: BVLL/BVLH
0x8A: BCRL/BCRH
0x8C: BDVL/BDVH
0x8E: BACL/BACH

0x90: BTPL/BTPH     ACPI battery trip point
0x92: SRNM
0x94: RSOL/RSOH
0xA0: BDVN          battery/model string region
0xCF: STAS
0xFE: TEMP
```

`_BTP` writes `BTPL/BTPH`; that is the ACPI battery trip-point/notification facility, **not** the charging-cap subsystem.

## Charge-limit state inside XRAM

Static reverse engineering of the exact EC image found a dedicated battery-limit subsystem centered on:

```text
XRAM[0x0D01].bit4   enable/state bit
XRAM[0x0D13]        threshold value #1
XRAM[0x0D14]        threshold value #2
XRAM[0x0394]        live SOC used by the decision logic
```

Important handler locations include:

```text
CODE:0xED60...      enable/status family
CODE:0xED7A         sets 0x0D01.bit4
CODE:0xED8E         checks 0x0D01.bit4
CODE:0xEDBA         validates/sets 0x0D13
CODE:0xEDDF         validates/sets 0x0D14
CODE:0xF508         disable/reset path; clears bit4 and both thresholds
CODE:0xF526         read helper for 0x0D13
CODE:0xF621         read helper for 0x0D14
CODE:0xC063         core SOC/threshold decision routine
```

The public host protocol that reaches these handlers is documented separately in [`battery-charge-limit.md`](battery-charge-limit.md).

## Threshold validation logic

Both threshold setters use the same 8051 range check before storing the value. The important pattern is equivalent to:

```asm
MOV  A,value
CLR  C
SUBB A,#0
JC   invalid

MOV  A,value
SETB C
SUBB A,#0x64
JNC  invalid
```

On the 8051, `SUBB` computes `A - operand - C`. Because the second comparison deliberately sets carry first, a value of exactly `100` still produces a borrow and remains on the valid path, while `101` does not.

Therefore the firmware accepts the inclusive range:

```text
0 .. 100
```

This proves the stored fields are percentage-like numeric values. It does **not** by itself prove how every possible pair maps to user-facing charging behavior.

## Charge-control working values

The threshold decision routine feeds working words around:

```text
0x0D54..0x0D57
0x0D65..0x0D68
```

Helpers around `CODE:0xC249` and `CODE:0xC27A` clear/copy these values depending on the SOC/threshold branch taken.

A later worker uses the resulting words while staging charger transactions associated with command numbers `0x14` and `0x15`. Those command numbers are consistent with common Smart Battery charger `ChargingCurrent` / `ChargingVoltage` conventions, but the exact semantic naming is still best treated as **inferred** unless the complete transaction path is decoded end to end.

## PMC2 host transport

Live Super-I/O configuration proves that logical device `0x12` is active at:

```text
DATA            0x68
COMMAND/STATUS  0x6C
```

The successful battery-limit experiments used the conventional PMC status bits:

```text
bit0 OBF  = output buffer full / EC has a response byte
bit1 IBF  = input buffer full / host must wait before sending another byte
```

The host transaction pattern that worked live was:

1. wait for `IBF=0`;
2. write command byte to `0x6C`;
3. wait for `IBF=0`;
4. write the command's data/subcommand byte to `0x68`;
5. for commands that return a value, wait for `OBF=1` and read `0x68`.

The command family itself is documented in [`battery-charge-limit.md`](battery-charge-limit.md).

## Rejected dedicated-I2EC hypothesis

Static EC code writes I2EC-related interface registers including:

```text
XRAM[0x200D]
XRAM[0x2012]
XRAM[0x2014]
XRAM[0x2015]
```

One initialization sequence writes values that originally led to the hypothesis that a dedicated four-port I2EC window might exist at base `0x380`.

A **read-only** live test selected EC addresses through candidate ports `0x381/0x382` and read `0x383`. Every target returned `0xFF`:

```text
EC[200D] = 0xFF
EC[0394] = 255
EC[0D13] = 255
EC[0D14] = 255
```

while the known host MMIO battery byte simultaneously returned:

```text
FEEC2394 = 92
```

The cross-check therefore failed decisively.

The static initialization also writes `0x2012 = 0x20`; the dedicated-port enable interpretation used during the investigation places the enable on another bit, so the candidate path was not actually shown to be live-enabled.

**Conclusion:** `0x380` is rejected/superseded as a usable stock I2EC path on this machine. Do not use it for EC memory access.

## Safety boundaries

- Do not raw-write unknown values into the `0xFEEC2300` MMIO aperture.
- Do not enable `ec_sys` write support as a shortcut for charge control.
- Do not brute-force PMC2 commands.
- Do not copy `0x07B9`, `0x07D0` or other generic Uniwill/Tongfang offsets onto this machine merely because the EC family is similar.
- Prefer the exact firmware-defined PMC2 command path when a known command exists.

The charge-limit work demonstrates why: generic Windows software contained familiar old offsets, but the actual P916F-STX firmware exposes a distinct command-controlled subsystem at `0x0D13/0x0D14`.
