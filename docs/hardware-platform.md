# Hardware and platform notes

## Identity

The researched machine is a **MECHREVO Xingyao 14 / 机械革命 星耀14** using the platform identifier:

```text
MECHREVO XINGYAO Series-P916F-STX
```

Confirmed machine-specific identity:

| Item | Observed value | Confidence |
|---|---|---|
| Product | MECHREVO Xingyao 14 | Live-confirmed |
| Board/platform | `P916F-STX` | Live-confirmed |
| CPU | AMD Ryzen AI 9 H 365 | Live-confirmed |
| CPU family | AMD Strix Point | Platform classification |
| iGPU | Radeon 880M | Live/platform-confirmed |
| RAM | 32 GiB on this unit | Live-confirmed |
| BIOS | `1.15` | Live-confirmed |
| EC version shown by firmware UI | `1.15` | Live-confirmed |
| BIOS build-date string | `05/07/2026` | Live-confirmed raw string |
| EC silicon | ITE `0x5571`, revision `0x07` | Live-confirmed through Super-I/O config space |

The raw build-date string is retained instead of silently converting it to an ISO date because firmware date formatting can be locale/vendor dependent. If interpreted in the common Insyde/SMBIOS `MM/DD/YYYY` form, it is 2026-05-07.

## BIOS/EC version namespaces

There are several different version strings in the machine and they should not be conflated:

```text
Laptop firmware UI:      BIOS 1.15 / EC 1.15
IT557x firmware string:  IT557x V1.09 E00 - 20230831
Other EC metadata:       ITE EC-V14.6
                         VER:01.0F.00
```

These values come from different layers/build systems. The internal `V1.09` string does not mean the laptop is still running the old system BIOS 1.09.

## Display

The researched unit's internal panel was previously observed as:

```text
2880 × 1800
```

No refresh-rate value is currently retained with the same confidence, so this repository deliberately does **not** assign one.

After the BIOS 1.15 update, Linux ACPI BGRT metadata was observed as:

```text
status  = 0
type    = 0
version = 1
xoffset = 1040
yoffset = 387
```

For a 2880-pixel-wide panel, `xoffset=1040` is consistent with an 800-pixel-wide centered boot image (`1040 + 800 + 1040 = 2880`), matching the 800×600 firmware animation resource found statically. The BGRT and animation details are documented in [`firmware-bios.md`](firmware-bios.md).

## Battery

The ACPI battery object is named `LCBT` under Linux.

A battery model string retained from the earlier machine investigation is:

```text
588974-3S-G-A0
```

The AC adapter is exposed as `ACAD`.

Linux battery telemetry used during the charge-limit validation includes:

```text
capacity
status
voltage_now
power_now
energy_now
```

On this machine `current_now` was not present in the observed `LCBT` sysfs directory.

## Audio hardware

ALSA probing identified the internal analog codec path as:

```text
Realtek ALC256 Analog
```

Live PipeWire/WirePlumber inspection exposed the internal speaker endpoint as ordinary stereo:

```text
output_FL
output_FR
```

No separate LFE, 2.1, 4.0 or discrete subwoofer channel was exposed to Linux.

The chassis uses multiple physical speaker drivers / a multi-speaker OEM layout, but Linux presents them as a stereo endpoint rather than individually controllable speakers. Windows uses the OEM Nahimic/A-Volute processing stack; Linux basic playback works but lacks the same tuning. See [`audio.md`](audio.md).

## Graphics and Linux desktop

The integrated Radeon graphics path has been used successfully under Wayland/Hyprland on CachyOS. Nothing found in the firmware research indicates that a proprietary graphics driver is required for the internal panel.

## Storage / dual-boot context

The researched machine has been used in a Windows 11 + Linux dual-boot configuration with UEFI booting. Different installation stages used GRUB/shim/sbctl, systemd-boot and later Limine. Those are installation choices, not platform requirements.

## Related platforms are not interchangeable

Research encountered sibling or similarly named variants including:

```text
P916F-HPT-R
P916F-ARL
```

They must be treated as **different firmware targets**. A common `P916F` prefix or similar chassis does not make their BIOS/EC images safe to cross-flash onto `P916F-STX`.

The same rule applies to EC register maps: generic Uniwill/Tongfang/other IT5571 offsets are comparative evidence only until the P916F-STX's own firmware or live hardware confirms them.
