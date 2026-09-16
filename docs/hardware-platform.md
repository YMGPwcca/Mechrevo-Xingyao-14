# Hardware and platform

## Platform identity

| Component | Value | Status |
|---|---|---|
| Product | MECHREVO Xingyao 14 / 机械革命 星耀14 | Live-confirmed |
| Board/platform | `P916F-STX` | Live-confirmed |
| Full platform string | `MECHREVO XINGYAO Series-P916F-STX` | Live-confirmed |
| CPU | AMD Ryzen AI 9 365 | AMD official model naming |
| CPU family | AMD Strix Point | Platform-confirmed |
| iGPU | Radeon 880M | Platform/live-confirmed |
| Memory | 32 GiB on the documented unit | Live-confirmed |
| Internal display | 2880×1800 | Live-confirmed |
| Firmware environment | UEFI / Insyde H2O | Static/live-confirmed |
| Tested BIOS | `1.15` | Live-confirmed |
| EC version reported by firmware | `1.15` | Live-confirmed |
| EC silicon | ITE `0x5571`, revision `0x07` | Live-confirmed |

## Firmware version namespaces

The platform exposes several independent version identifiers:

```text
System BIOS:            1.15
EC version in firmware: 1.15
IT557x internal string: IT557x V1.09 E00 - 20230831
Additional EC strings:  ITE EC-V14.6
                        VER:01.0F.00
```

The IT557x strings identify the embedded-controller firmware build and are not the system BIOS version.

## Display and BGRT geometry

The internal panel is 2880×1800.

BIOS 1.15 exposes ACPI BGRT placement metadata:

```text
status  = 0
type    = 0
version = 1
xoffset = 1040
yoffset = 387
```

The firmware contains an 800×600 OEM boot-animation resource. The horizontal placement is consistent with a centered 800-pixel image on a 2880-pixel panel:

```text
1040 + 800 + 1040 = 2880
```

See [`firmware-bios.md`](firmware-bios.md).

## Battery and AC adapter

Linux exposes:

```text
Battery: /sys/class/power_supply/LCBT
AC:      /sys/class/power_supply/ACAD
```

Battery model string:

```text
588974-3S-G-A0
```

The firmware charge-limit implementation is documented in [`battery-charge-limit.md`](battery-charge-limit.md).

## Audio

Internal analog codec:

```text
Realtek ALC256 Analog
```

The chassis has four physical speaker drivers, two per side. Linux exposes the internal speaker path as a stereo FL/FR endpoint rather than four independent logical channels.

See [`audio.md`](audio.md).

## Platform-family caution

Other firmware targets encountered under the broader P916F naming family include:

```text
P916F-HPT-R
P916F-ARL
```

BIOS images, EC binaries and register maps for those targets must not be assumed compatible with `P916F-STX` without direct validation.
