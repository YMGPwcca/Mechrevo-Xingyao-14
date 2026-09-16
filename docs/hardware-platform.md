# Hardware and platform notes

## Identity

**Live-confirmed / package-confirmed where noted**

- Product family: **MECHREVO Xingyao 14 / 机械革命 星耀14**.
- Mainboard/platform identifier: **`MECHREVO XINGYAO Series-P916F-STX`**.
- Platform generation: AMD **Strix Point**.
- The researched machine has been reported by firmware/OS tooling as a **Ryzen AI 9 365 / Ryzen AI 9 H 365 class** configuration.
- Integrated GPU: **AMD Radeon 880M**.
- Installed memory on the researched unit: **32 GiB**.

The exact retail CPU naming used by MECHREVO can vary between product listings and OS strings. This repository records firmware and OS observations rather than normalizing the marketing name.

## Firmware identity

- Tested BIOS: **1.15**.
- Tested EC version exposed by BIOS: **1.15**.
- BIOS build date: **2026-05-07**.
- The current firmware image contains an ITE EC firmware identifying itself with strings including:
  - `ITE EC-V14.6`
  - `IT557x V1.09 E00 - 20230831`
- Live Super-I/O probing identifies the EC as **ITE `0x5571`, revision `0x07`**.

See [`firmware-bios.md`](firmware-bios.md) and [`embedded-controller.md`](embedded-controller.md) for the distinction between BIOS/EC package version numbers and the internal IT557x firmware string.

## Graphics and display observations

On Linux, the machine has been used with the integrated Radeon graphics stack. The display path is functional under Wayland/Hyprland.

A previously observed internal panel configuration on the user's Linux installation was `1920×1080 @ 144 Hz`. This is recorded as an observation of the researched unit, not a claim that every Xingyao 14 SKU uses the same panel.

## Audio hardware

Linux exposes the internal speaker path through the AMD/Ryzen HD-audio controller; ALSA probing identified a **Realtek ALC256 Analog** codec path. The physical machine has multiple speaker drivers, but Linux presents the internal speakers as a normal stereo FL/FR sink rather than exposing a separate subwoofer/LFE channel.

Windows audio quality is substantially better because the OEM stack includes **Nahimic** tuning/DSP. Linux playback itself works, but without the Windows DSP profile the sound has been observed as thinner/weaker. This behavior reproduced on an Ubuntu live environment as well, so it is not specific to CachyOS.

See [`audio.md`](audio.md).

## Storage / dual-boot context

The researched machine has been used in a Windows 11 + Linux dual-boot configuration with UEFI booting. Bootloader experiments included GRUB/shim/sbctl and later systemd-boot/Limine on different stages of the setup. These are installation choices on the researched unit, not firmware requirements of the platform.

## Related platforms

Firmware research encountered sibling P916F variants including names such as:

- `P916F-HPT-R`
- `P916F-ARL`

They should be treated as **different targets**. Similar naming does not make their BIOS or EC firmware safe to cross-flash onto `P916F-STX`.
