# Linux notes

The researched Xingyao 14 / `P916F-STX` has been used primarily with **CachyOS / Arch-family Linux**.

This document records laptop-specific behavior rather than general Linux installation instructions.

## Machine identity visible to Linux / platform identity

The retained platform identity is:

```text
MECHREVO XINGYAO Series-P916F-STX
AMD Ryzen AI 9 365
Radeon 880M
```

AMD's official model name is `Ryzen AI 9 365`. A photographed OEM BIOS screen appears to label the processor `AMD Ryzen AI 9 HX 365`; that OEM firmware string is recorded separately in [`hardware-platform.md`](hardware-platform.md) rather than used as the canonical AMD model name.

The internal panel on this unit was previously observed as:

```text
2880 × 1800
```

A previously documented `1920×1080 @ 144 Hz` value belonged to another machine/context and was removed during the documentation audit. No refresh-rate value is currently retained here with enough confidence to publish as a P916F-STX fact.

## Battery and AC devices

The battery is exposed as:

```text
/sys/class/power_supply/LCBT
```

The AC adapter is exposed as:

```text
/sys/class/power_supply/ACAD
```

Retained battery model string:

```text
588974-3S-G-A0
```

Useful battery attributes observed include:

```text
capacity
status
voltage_now
power_now
energy_now
```

`current_now` was not present in the observed sysfs device.

Representative capped state with AC connected:

```text
capacity   = 79%
status     = Not charging
power_now  = 0
energy_now = 63154000
ACAD       = online=1
```

During the later load test the battery changed to:

```text
78%
Charging
power_now  = 28128000
energy_now = 62661000
```

after a five-minute all-CPU stress interval.

The charge-limit experiment and interpretation of these values are documented in [`battery-charge-limit.md`](battery-charge-limit.md).

## Missing generic Linux charge-limit controls

The `LCBT` power-supply device did not expose:

```text
charge_control_start_threshold
charge_control_end_threshold
charge_behaviour
```

The platform also had a `huawei-wmi` device, but it did not expose usable battery-charge attributes.

Therefore stock Linux does not currently surface the P916F charge-limit feature through the generic power-supply threshold ABI.

## ACPI / EC behavior

Linux sees a conventional ACPI battery device, while the firmware also exposes an EC shared-memory region at physical address:

```text
0xFEEC2300
```

The DSDT declares it as a 256-byte `SystemMemory` operation region. Static EC analysis maps it to:

```text
EC XRAM 0x0300..0x03FF
```

This memory window is distinct from the ordinary ACPI EC byte interface and from the ITE PMC2 command interface.

The working battery-cap host path is ITE PMC2:

```text
DATA            0x68
COMMAND/STATUS  0x6C
```

## Headless / lid behavior

For headless use, systemd-logind can be configured not to suspend when the lid closes.

The researched installation used an override equivalent to:

```text
[Login]
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

The effective configuration was checked with:

```text
systemd-analyze cat-config systemd/logind.conf
```

This is purely an OS policy. It does not change the firmware's physical lid signal or the ACPI `_LID` implementation.

## BGRT observation after BIOS 1.15

Linux exposed the post-update ACPI BGRT metadata:

```text
status  0
type    0
version 1
xoffset 1040
yoffset 387
```

The `xoffset=1040` value is geometrically consistent with an 800-pixel-wide centered image on the observed 2880-pixel-wide internal panel. See [`firmware-bios.md`](firmware-bios.md).

## Graphics / Wayland

The integrated Radeon graphics stack has been used successfully under Wayland/Hyprland on CachyOS.

No P916F-specific graphics workaround has emerged from the firmware work described in this repository.

## Power profiles

The machine has been used with Linux AMD-pstate / `powerprofilesctl` profiles such as:

```text
performance
balanced
power-saver
```

These CPU/platform power-policy profiles are independent from the EC's battery charge-limit state.

## Audio

Linux audio works through ALSA/PipeWire. The internal codec path is Realtek ALC256 Analog, and PipeWire exposes ordinary stereo FL/FR speaker channels. The missing piece compared with Windows is primarily OEM Nahimic/A-Volute processing/tuning rather than basic codec detection. See [`audio.md`](audio.md).

## Bluetooth / Wi-Fi observation

On one Linux installation the wireless stack used Realtek `8852AU` firmware (`rtl8852au_fw.bin.zst`). A temporary Bluetooth headphone stutter was observed and stopped after toggling Wi-Fi.

This remains an installation/runtime observation rather than a diagnosed P916F hardware defect.

## Linux integration status

No upstream Linux driver is currently known from this investigation to expose the machine's EC charge limit as standard power-supply threshold files.

The protocol is documented well enough for future driver/userspace work, but this repository intentionally remains **documentation-only**.
