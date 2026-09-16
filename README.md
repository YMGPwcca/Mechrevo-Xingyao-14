# MECHREVO Xingyao 14 / P916F-STX documentation

This repository is a **documentation-only** knowledge base for the MECHREVO Xingyao 14 (机械革命 星耀14) based on the `P916F-STX` platform.

It records facts observed on the researched machine, static findings from its own BIOS/EC images, Linux behavior, ACPI/WMI analysis, Windows Control Center clues, rejected hypotheses, and experimentally validated battery charge-limit behavior. It intentionally contains **no application, utility or driver source code**.

> [!WARNING]
> Some documents describe low-level firmware and embedded-controller interfaces. Read-only inspection is much safer than writes. Do not copy register offsets from other MECHREVO/Tongfang/Uniwill models, do not brute-force EC commands, and do not flash sibling-platform firmware. Claims that are not live-validated are labelled accordingly.

## Machine identity

| Item | Value | Evidence |
|---|---|---|
| Product | MECHREVO Xingyao 14 / 机械革命 星耀14 | user/live identity |
| Mainboard/platform | `MECHREVO XINGYAO Series-P916F-STX` | firmware/OS identity |
| CPU | AMD Ryzen AI 9 365 | AMD official model naming; platform characteristics match |
| OEM BIOS CPU label | a photographed BIOS screen appears to show `AMD Ryzen AI 9 HX 365` | live photo/OCR; preserved as the firmware-presented string rather than treated as AMD's official retail name |
| CPU family | AMD Strix Point | AMD/platform classification |
| iGPU | Radeon 880M | live/platform identity; matches Ryzen AI 9 365 |
| RAM | 32 GiB on the researched machine | live |
| Internal panel | 2880×1800 on the researched machine | prior live observation |
| Current tested BIOS | `1.15` | live |
| EC version reported by firmware UI | `1.15` | live |
| BIOS build-date string | `05/07/2026` | live; raw string retained because date format is firmware-dependent |
| EC silicon | ITE `0x5571`, revision `0x07` | live Super-I/O probing |
| Primary Linux used for testing | CachyOS | live |

AMD's official product name is **Ryzen AI 9 365**; AMD pairs that processor with Radeon 880M graphics. The separate `HX` token visible/recognized in the OEM BIOS screen is kept as an OEM firmware-label observation rather than silently rewriting AMD's model name.

The EC also contains its own internal firmware strings such as `IT557x V1.09 E00 - 20230831`. That internal IT557x build identifier is **not the same version namespace** as the `EC 1.15` value shown by the laptop firmware UI.

## Documents

- [`docs/evidence-matrix.md`](docs/evidence-matrix.md) — compact claim-by-claim evidence/confidence matrix; start here when checking what is actually proven.
- [`docs/live-validation-log.md`](docs/live-validation-log.md) — preserved raw outputs from the I2EC rejection, PMC2 discovery, battery-limit setup, reboot and power tests.
- [`docs/hardware-platform.md`](docs/hardware-platform.md) — machine identity, display, battery, audio and platform observations.
- [`docs/firmware-bios.md`](docs/firmware-bios.md) — BIOS 1.15 package, current ROM, boot animation/BGRT, SetupUtility findings and Dynamic LID.
- [`docs/boot-logo-research.md`](docs/boot-logo-research.md) — detailed analysis of the two Insyde logo-only update paths and why neither is provisioned/usable on this BIOS 1.15 build.
- [`docs/embedded-controller.md`](docs/embedded-controller.md) — IT5571 EC architecture, firmware carve, H2RAM mapping, PMC2 transport, charge-control internals and the rejected I2EC hypothesis.
- [`docs/battery-charge-limit.md`](docs/battery-charge-limit.md) — exact P916F battery-limit command family, static handlers and live validation.
- [`docs/acpi-wmi.md`](docs/acpi-wmi.md) — DSDT EC mapping, battery object, Huawei-compatible WMI and negative results from other charge-limit paths.
- [`docs/control-center.md`](docs/control-center.md) — MECHREVO Control Center packages and what they did—and did not—prove about this machine.
- [`docs/linux.md`](docs/linux.md) — Linux behavior, battery/AC telemetry, lid/headless notes and current integration status.
- [`docs/audio.md`](docs/audio.md) — ALC256/PipeWire observations and the Windows Nahimic-vs-Linux tuning gap.
- [`docs/boot.md`](docs/boot.md) — UEFI/dual-boot observations from the researched installation.
- [`docs/research-artifacts.md`](docs/research-artifacts.md) — hashes, offsets and provenance of firmware/reverse-engineering artifacts.
- [`docs/open-questions.md`](docs/open-questions.md) — facts still unverified or only partially understood.

## Confidence labels used in this repository

- **Live-confirmed** — directly observed on the actual P916F-STX machine.
- **Static-confirmed** — derived from this machine's own BIOS/EC image, but not necessarily exercised on live hardware.
- **Inferred** — strongly suggested by code/data flow or behavior, but exact semantics have not been fully proven.
- **Comparative only** — observed in generic OEM software, a sibling platform or another IT557x machine; useful context, but not P916F evidence by itself.
- **Rejected / superseded** — a hypothesis that was tested or re-evaluated and should no longer be used.

## Current headline result

The P916F-STX has a real firmware-level battery charge-limit subsystem reachable through the ITE **PMC2** host interface at data port `0x68` and command/status port `0x6C`.

On the researched machine, the specific configuration:

```text
state = enabled
T1    = 80
T2    = 100
```

was live-tested and:

- stopped charging around the displayed 79–80% boundary,
- allowed charging again below that boundary,
- survived a normal reboot without re-applying the commands,
- required no Windows Control Center process after reboot.

Only the `80/100` threshold pair has been live-validated so far. The firmware accepts numeric threshold values from 0 through 100, but behavior for arbitrary values has **not** yet been experimentally mapped.

The full protocol, static handler addresses and validation chronology are documented in [`docs/battery-charge-limit.md`](docs/battery-charge-limit.md); the exact live outputs are preserved in [`docs/live-validation-log.md`](docs/live-validation-log.md).
