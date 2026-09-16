# Embedded controller (ITE IT5571)

## Live identity

The `P916F-STX` machine exposes an ITE Super-I/O / EC.

Live configuration-space probing found:

| Item | Value |
|---|---:|
| Super-I/O config port | `0x4E` |
| Chip ID | `0x5571` |
| Revision | `0x07` |
| PMC2 logical device | `0x12` |
| PMC2 active | `0x01` |
| PMC2 I/O #0 | `0x0068` |
| PMC2 I/O #1 | `0x006C` |
| PMC2 I/O #2 | `0x0000` |
| IRQ | `0x00` |

The standard ITE config port at `0x2E` returned `0xFFFF` on this machine; `0x4E` is the live configuration interface.

## EC firmware image

The current 32 MiB ROM contains an IT557x firmware image.

Carved image:

- ROM offset: `0x081000`
- length: `0x20000` bytes (128 KiB)
- research filename: `P916F-IT5571-EC-1.09.bin`
- SHA-256: `42c117f00c130c5e533be93ee1657401ac4d687255ed1b2250f74d3cc79397ea`

Strings inside the firmware include:

- `ITE EC-V14.6`
- `IT557x V1.09 E00 - 20230831`
- `ITE Tech. Inc.`
- `AMD Motherboard`
- `MECHREVO`
- `VER:01.0F.00`

The firmware is MCS-51/8051-family code. Its reset entry begins with `LJMP 0x0070`.

## ACPI-visible shared RAM / H2RAM

The DSDT declares:

```text
OperationRegion (ERAM, SystemMemory, 0xFEEC2300, 0x100)
```

Static EC analysis found code configuring a 256-byte H2RAM window with the corresponding EC-side base at `0x0300`.

The established mapping is:

```text
host physical 0xFEEC2300 + N  <->  EC XRAM 0x0300 + N
```

This was live-confirmed earlier by comparing host MMIO data with firmware-known EC values.

### Battery percentage anchor

`EC XRAM[0x0394]` is strongly established as the EC-side battery percentage value used by charging logic.

Because of the H2RAM mapping above, this is visible to the host at:

```text
0xFEEC2394
```

Live reads matched the user-visible battery percentage closely enough to establish the mapping.

## ACPI battery-region fields

The ACPI-visible EC region contains battery/AC state and telemetry. Relevant DSDT fields include:

```text
0x30: ACIN bit6, ACLW bit7
0x80: BATI bit0, BAII bit1, BACG bit2, BAIC bit3
0x81+: battery telemetry words
0x90: BTPL/BTPH battery trip point
0x92: SRNM
0x94: RSOL/RSOH
0xA0: BDVN model string area
0xCF: STAS
0xFE: TEMP
```

`_BTP` writes the ACPI battery trip-point fields `BTPL/BTPH`; this is an ACPI notification/trip-point mechanism, **not** the charge-limit feature documented elsewhere.

## PMC2 transport

The machine's live ITE logical-device configuration proves that PMC2 is active at:

```text
DATA       = 0x68
COMMAND/STATUS = 0x6C
```

This interface is the host transport used by the battery charge-limit command family documented in [`battery-charge-limit.md`](battery-charge-limit.md).

This finding supersedes an earlier working assumption that the platform had no useful `0x68/0x6C` interface. The earlier statement was based on ACPI-path inspection; live Super-I/O configuration-space probing later proved PMC2 exists and is active.

## Dedicated I2EC path: investigated, not usable in stock state

Static analysis showed EC code writing I2EC-related control registers around:

- `XRAM[0x200D]`
- `XRAM[0x2012]`
- `XRAM[0x2014]`
- `XRAM[0x2015]`

An early interpretation guessed a dedicated I2EC base of `0x380`. A live read-only test of ports `0x381..0x383` returned `0xFF` for all attempted EC memory reads while the MMIO battery percentage returned a real value.

That test **failed the cross-check** and therefore disproved the guessed stock I2EC path.

The corrected interpretation is that the relevant dedicated-port enable bit is not set in the observed stock configuration. Do not use `0x380` as an EC memory access path on this machine.

## Safety notes

- Do not raw-write unknown bytes to `0xFEEC2300`.
- Do not enable `ec_sys` write support as a shortcut for battery control.
- Do not brute-force EC command bytes.
- Prefer an exact, firmware-understood host command over direct XRAM manipulation whenever such a command exists.
- Do not assume register addresses from another IT5571 laptop apply to this P916F firmware.

The charge-limit work is a good example: old Uniwill-style charge-limit offsets were found in generic software, but the actual P916F firmware exposes a different command-controlled subsystem.
