# Hardware and platform reference

This document describes the concrete hardware/firmware identity of the documented MECHREVO Xingyao 14 and the platform facts needed to interpret the lower-level reverse-engineering results elsewhere in the repository.

## 1. Platform identity

| Component | Value | Status |
|---|---|---|
| Product | MECHREVO Xingyao 14 / 机械革命 星耀14 | Live-confirmed |
| Board/platform | `P916F-STX` | Live-confirmed |
| Full platform string | `MECHREVO XINGYAO Series-P916F-STX` | Live-confirmed |
| CPU | AMD Ryzen AI 9 365 | AMD official model naming |
| CPU family | AMD Strix Point | Platform-confirmed |
| CPU topology | 10 cores / 20 threads | AMD official specification |
| iGPU | Radeon 880M | Platform/live-confirmed |
| Memory | 32 GiB on the documented unit | Live-confirmed |
| Internal display | 2880×1800 | Live-confirmed |
| Firmware environment | UEFI / Insyde H2O | Static/live-confirmed |
| Tested BIOS | `1.15` | Live-confirmed |
| EC version reported by firmware | `1.15` | Live-confirmed |
| BIOS build-date string | `05/07/2026` | Live-confirmed raw string |
| EC silicon | ITE `0x5571`, revision `0x07` | Live-confirmed |
| Internal analog codec | Realtek ALC256 | Live-confirmed |
| Battery object | `LCBT` | Live-confirmed under Linux |
| AC adapter object | `ACAD` | Live-confirmed under Linux |
| Battery model string | `588974-3S-G-A0` | Live-confirmed under Linux |

The raw BIOS build-date string is intentionally preserved without forcing a date interpretation. If interpreted in the common Insyde/SMBIOS `MM/DD/YYYY` convention it corresponds to 2026-05-07.

## 2. Firmware version namespaces

Several independent version identifiers appear on the platform:

```text
System BIOS:            1.15
EC version in firmware: 1.15
IT557x internal string: IT557x V1.09 E00 - 20230831
Additional EC strings:  ITE EC-V14.6
                        VER:01.0F.00
```

These belong to different build/version namespaces.

The embedded-controller string `V1.09` does **not** mean the machine is running system BIOS 1.09, nor does it contradict the firmware UI's `EC 1.15` label.

This distinction matters when comparing update packages, raw flash images and internal EC firmware revisions.

## 3. CPU / graphics platform

The CPU is AMD Ryzen AI 9 365 from the Strix Point / Ryzen AI 300 family.

Relevant platform characteristics for this repository:

```text
CPU:  Ryzen AI 9 365
cores: 10
threads: 20
iGPU: Radeon 880M
```

The integrated Radeon graphics path functions through the standard AMD Linux graphics stack. No P916F-specific graphics firmware override has been required by the work documented here.

## 4. Internal display

The documented unit's internal panel resolution is:

```text
2880 × 1800
```

No refresh-rate value is published here because the investigation did not retain one with the same evidence quality as the resolution.

### BGRT placement

After BIOS 1.15, Linux exposed:

```text
status  = 0
type    = 0
version = 1
xoffset = 1040
yoffset = 387
```

The BIOS contains an 800×600 OEM boot-animation resource.

The horizontal placement is consistent with exact centering on a 2880-pixel-wide panel:

```text
1040 + 800 + 1040 = 2880
```

This provides a useful geometric cross-check between the live ACPI BGRT data and the statically extracted boot-graphics resource.

The firmware resource itself is described in [`firmware-bios.md`](firmware-bios.md) and [`boot-logo-research.md`](boot-logo-research.md).

## 5. Embedded controller

Live ITE Super-I/O probing identifies:

```text
chip ID  = 0x5571
revision = 0x07
config   = 0x4E
```

The EC firmware is an IT557x 8051-family image embedded in the platform flash.

The working PMC2 logical device is:

```text
LDN             = 0x12
DATA            = 0x68
COMMAND/STATUS  = 0x6C
```

This interface is central to the battery charge-limit work.

The exact EC image, XRAM fields and H2RAM configuration are documented in [`embedded-controller.md`](embedded-controller.md).

## 6. Battery and adapter

Linux power-supply objects:

```text
Battery: /sys/class/power_supply/LCBT
AC:      /sys/class/power_supply/ACAD
```

Battery model:

```text
588974-3S-G-A0
```

Observed battery telemetry includes:

```text
capacity
status
voltage_now
power_now
energy_now
model_name
```

The machine does not expose the usual Linux charge-threshold attributes such as:

```text
charge_control_start_threshold
charge_control_end_threshold
charge_behaviour
```

The firmware nevertheless contains a working charge-limit subsystem accessed through the EC PMC2 command family. See [`battery-charge-limit.md`](battery-charge-limit.md).

## 7. Audio hardware

Internal analog codec:

```text
Realtek ALC256 Analog
```

The chassis has four physical speaker drivers, two per side.

Linux exposes the internal speaker path as a stereo endpoint:

```text
output_FL
output_FR
```

No separate LFE/4.0 endpoint was observed.

The Windows OEM stack uses Nahimic / A-Volute processing. Linux basic playback is functional, but the OEM-equivalent processing parameters have not been recovered.

See [`audio.md`](audio.md).

## 8. UEFI / Insyde firmware environment

The platform uses Insyde H2O firmware.

Relevant components identified in the BIOS 1.15 image include:

```text
SetupUtility
OemBadgingSupportDxe
BootGraphicsResourceTableDxe
ChipsetSvcSmm
Insyde H2OFFT / IHISI update infrastructure
```

The BIOS image also contains hidden setup forms and OEM-specific firmware resources described in the firmware documents.

This repository does not treat generic Insyde behavior as automatically enabled on the P916F. Each mechanism is traced into the exact machine firmware before being considered applicable.

## 9. ACPI integration

The platform exposes a standard ACPI embedded controller:

```text
_HID = PNP0C09
GPE  = 0x0B
```

and a separate SystemMemory-backed EC window:

```text
OperationRegion (ERAM, SystemMemory, 0xFEEC2300, 0x100)
```

The exact IT5571 firmware maps this host-visible region to:

```text
EC XRAM 0x0300..0x03FF
```

This H2RAM aperture is distinct from both the standard ACPI EC host interface and the ITE PMC2 command channel.

See [`acpi-wmi.md`](acpi-wmi.md) and [`embedded-controller.md`](embedded-controller.md).

## 10. Platform-family caution

Other firmware targets encountered under the broader P916F naming family include:

```text
P916F-HPT-R
P916F-ARL
```

Their existence does not imply firmware compatibility with `P916F-STX`.

BIOS images, EC binaries, GPIO assumptions, flash layouts and EC register maps must be treated as model-specific until validated directly.

The same rule applies to generic Tongfang/Uniwill/MECHREVO EC knowledge: architectural similarity is useful for orientation, but machine-specific evidence takes precedence.
