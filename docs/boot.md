# UEFI / boot notes

This document records boot-related observations from the researched `P916F-STX` installation. These are not requirements for the hardware.

## UEFI environment

The laptop boots in UEFI mode and has been used in a Windows 11 + Linux dual-boot setup.

During Linux setup/recovery work, multiple boot approaches were tried over time:

- GRUB with Secure Boot/shim/sbctl experiments,
- systemd-boot,
- later Limine on the current installation.

The important firmware-level point is that the machine behaves as a normal UEFI laptop; bootloader choice is an OS installation decision.

## Secure Boot experiments

Attempts to use self-managed signing/MOK/sbctl with GRUB led at one point to an `invalid signature` boot failure and rescue work. The installation was later moved away from that exact setup.

This history is documented so it is not mistaken for a P916F hardware limitation: it was an installation/signing configuration issue, not evidence that Secure Boot cannot work on the laptop.

## Linux / Windows storage layout observed during one installation

A previous dual-boot layout used:

- Linux root on Btrfs,
- a VFAT EFI/boot partition,
- Windows 11 on another NVMe device,
- additional NTFS application/game partitions mounted from Linux.

Exact partition numbers and UUIDs are installation-specific and intentionally not treated as model documentation.

## Firmware boot branding

The pre-OS boot path uses OEM Insyde badging resources and, in BIOS 1.15, an embedded animated GIF. See [`firmware-bios.md`](firmware-bios.md).

## Lid / boot relation

Firmware contains `Dynamic LID` / `AmdDynamicLid` references, while ACPI exposes lid state normally. No confirmed setting has yet been established that implements "open lid to power on" specifically on this model.
