# Linux notes

The researched Xingyao 14 / `P916F-STX` has been used primarily with **CachyOS / Arch-family Linux**.

This file records laptop-specific observations rather than general Linux installation instructions.

## Battery and AC devices

The battery is exposed as:

```text
/sys/class/power_supply/LCBT
```

The AC adapter is exposed as:

```text
/sys/class/power_supply/ACAD
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

At the charge cap with AC connected, a representative live state was:

```text
LCBT capacity   = 79%
LCBT status     = Not charging
LCBT power_now  = 0
ACAD online     = 1
```

The generic Linux charge-threshold attributes were not exposed.

See [`battery-charge-limit.md`](battery-charge-limit.md).

## ACPI / EC behavior

Linux sees a conventional ACPI battery device, but the machine also exposes a separate memory-backed EC shared-RAM window at physical address:

```text
0xFEEC2300
```

This is backed by the IT5571 H2RAM configuration and is not equivalent to the byte range exported by the `ec_sys` driver's ordinary ACPI EC interface.

The charge-limit feature is not surfaced by stock Linux power-supply sysfs; it is implemented behind an ITE PMC2 protocol.

## Lid behavior for headless use

On systemd-based Linux, closing the lid can be configured not to suspend through `logind`.

The researched installation used an override equivalent to:

```text
[Login]
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

This was verified through `systemd-analyze cat-config systemd/logind.conf`.

This is an OS policy setting only; it does not modify firmware lid behavior.

## Graphics / Wayland

The integrated Radeon graphics stack has been used successfully under Wayland/Hyprland.

The internal panel was observed as `1920×1080 @ 144 Hz` on the researched unit.

## Power management

The machine has been used with Linux `powerprofilesctl` / AMD pstate profiles such as:

```text
performance
balanced
power-saver
```

These OS CPU/power-policy profiles are independent from the EC battery charge-limit feature.

## Audio

Linux audio works through ALSA/PipeWire, but OEM Windows audio processing is missing. See [`audio.md`](audio.md).

## Bluetooth / Wi-Fi observation

On one Linux installation the wireless device was identified with Realtek `8852AU` firmware present as `rtl8852au_fw.bin.zst`. A temporary Bluetooth audio stutter was observed and stopped after toggling Wi-Fi.

This is recorded as an observation of the researched installation, not yet as a diagnosed platform defect.

## What has not been upstreamed

At present there is no known upstream Linux driver exposing this machine's battery limit through the standard power-supply threshold interface.

The reverse-engineered EC protocol is sufficiently understood for documentation, but this repository intentionally contains **documentation only**, not a driver or userspace utility.
