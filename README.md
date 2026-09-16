# MECHREVO Xingyao 14 / P916F-STX documentation

This repository is a **documentation-only** knowledge base for the MECHREVO Xingyao 14 based on the `P916F-STX` platform.

It contains observed hardware/firmware information, Linux notes, ACPI/WMI and embedded-controller reverse-engineering results, BIOS research, and experimentally validated battery charge-limit behavior. It intentionally contains **no application or driver source code**.

> [!WARNING]
> Some documents describe low-level firmware and EC interfaces. Read-only inspection is generally much safer than writes. Do not copy register offsets from other MECHREVO/Tongfang/Uniwill models and do not flash sibling-platform firmware. Where a finding is not live-validated, the document says so explicitly.

## Machine identity

| Item | Value |
|---|---|
| Product | MECHREVO Xingyao 14 (机械革命 星耀14) |
| Mainboard/platform | `MECHREVO XINGYAO Series-P916F-STX` |
| CPU platform | AMD Strix Point; machine reported as Ryzen AI 9 H 365 / Ryzen AI 9 365 |
| iGPU | Radeon 880M |
| RAM | 32 GiB on the researched machine |
| Current tested BIOS | `1.15` |
| Current tested EC version exposed by BIOS | `1.15` |
| BIOS build date | `2026-05-07` |
| EC silicon, live detected | ITE `0x5571`, revision `0x07` |
| Primary Linux used for testing | CachyOS |

## Documents

- [`docs/hardware-platform.md`](docs/hardware-platform.md) — machine identity, platform facts and hardware observations.
- [`docs/firmware-bios.md`](docs/firmware-bios.md) — BIOS 1.15 package, ROM layout, boot logo/animation and hidden setup findings.
- [`docs/embedded-controller.md`](docs/embedded-controller.md) — IT5571 EC architecture, H2RAM mapping, PMC2 transport and known EC internals.
- [`docs/battery-charge-limit.md`](docs/battery-charge-limit.md) — exact P916F battery-limit protocol and live validation.
- [`docs/acpi-wmi.md`](docs/acpi-wmi.md) — DSDT EC mapping, Huawei-compatible WMI and negative results from other charge-limit paths.
- [`docs/control-center.md`](docs/control-center.md) — MECHREVO Control Center packages and what was learned from them.
- [`docs/linux.md`](docs/linux.md) — Linux behavior, battery telemetry, lid/headless notes and platform integration status.
- [`docs/audio.md`](docs/audio.md) — Linux audio hardware/path observations and the Windows-vs-Linux tuning gap.
- [`docs/boot.md`](docs/boot.md) — UEFI/dual-boot observations and tested boot entries.
- [`docs/research-artifacts.md`](docs/research-artifacts.md) — hashes, offsets and provenance of firmware/reverse-engineering artifacts.
- [`docs/open-questions.md`](docs/open-questions.md) — facts still unverified or only partially understood.

## Confidence labels used in this repository

- **Live-confirmed** — observed on the actual P916F-STX machine.
- **Static-confirmed** — derived from this machine's own BIOS/EC image but not necessarily exercised on hardware.
- **Inferred** — strongly suggested by code/data flow, but exact semantics have not yet been experimentally proven.
- **Comparative only** — observed in another model/software path; useful as context, not valid P916F evidence by itself.

## Current headline result

The P916F-STX has a real firmware-level battery charge-limit subsystem reachable through ITE PMC2. On the researched machine, a limit configured as `T1=80`, `T2=100`, enabled through the EC protocol:

- stopped charging around 80%,
- resumed charging below the cap,
- survived a normal reboot,
- required no Windows Control Center service to remain active after reboot.

The exact protocol and evidence are documented in [`docs/battery-charge-limit.md`](docs/battery-charge-limit.md).
