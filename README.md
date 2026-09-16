# MECHREVO Xingyao 14 / P916F-STX Technical Documentation

Technical reference for the MECHREVO Xingyao 14 (机械革命 星耀14) based on the `P916F-STX` platform.

The repository documents machine-specific hardware, firmware, ACPI/WMI, embedded-controller interfaces and behavior that has been validated on the actual platform or recovered from its exact firmware images. It intentionally contains no driver, utility or application source code.

Personal desktop configuration, dual-boot history, one-off troubleshooting steps and unrelated setup notes are outside the scope of this repository.

## Reference platform

| Component | Value |
|---|---|
| Product | MECHREVO Xingyao 14 / 机械革命 星耀14 |
| Platform | `MECHREVO XINGYAO Series-P916F-STX` |
| CPU | AMD Ryzen AI 9 365 |
| CPU family | AMD Strix Point |
| iGPU | Radeon 880M |
| Memory | 32 GiB on the documented unit |
| Internal display | 2880×1800 |
| Tested BIOS | `1.15` |
| EC version reported by firmware | `1.15` |
| BIOS build-date string | `05/07/2026` |
| EC silicon | ITE `0x5571`, revision `0x07` |

The IT557x firmware contains its own internal build strings such as `IT557x V1.09 E00 - 20230831`; those are a different version namespace from the system firmware's `EC 1.15` label.

## Documentation

- [`docs/hardware-platform.md`](docs/hardware-platform.md) — hardware and firmware identity.
- [`docs/firmware-bios.md`](docs/firmware-bios.md) — BIOS 1.15 image, boot graphics, SetupUtility and hidden setup findings.
- [`docs/boot-logo-research.md`](docs/boot-logo-research.md) — Insyde logo-update mechanisms and P916F-specific results.
- [`docs/embedded-controller.md`](docs/embedded-controller.md) — IT5571 firmware, H2RAM, PMC2 and EC charge-control internals.
- [`docs/battery-charge-limit.md`](docs/battery-charge-limit.md) — battery charge-limit command protocol and semantics established so far.
- [`docs/validation.md`](docs/validation.md) — live validation results for the EC and battery-limit protocol.
- [`docs/acpi-wmi.md`](docs/acpi-wmi.md) — ACPI EC fields, battery object, OEM control methods and WMI findings.
- [`docs/control-center.md`](docs/control-center.md) — relevant static findings from MECHREVO Windows control software.
- [`docs/linux.md`](docs/linux.md) — Linux-visible platform interfaces and current kernel exposure.
- [`docs/audio.md`](docs/audio.md) — codec, logical speaker topology and OEM DSP status.
- [`docs/research-artifacts.md`](docs/research-artifacts.md) — source-image provenance, offsets and hashes.
- [`docs/evidence-matrix.md`](docs/evidence-matrix.md) — validation status of important claims.
- [`docs/open-questions.md`](docs/open-questions.md) — unresolved technical behavior only.

## Evidence terminology

- **Live-confirmed** — observed on the actual P916F-STX hardware.
- **Static-confirmed** — established from the exact machine firmware or ACPI tables.
- **Inferred** — strongly suggested by implementation or measured behavior but not fully proven.
- **Comparative only** — comes from generic OEM software or another platform and is not P916F proof.
- **Rejected / superseded** — tested or re-evaluated and should not be used as the P916F implementation.

## Battery charge-limit result

The P916F-STX implements a firmware-level battery charge limiter through ITE PMC2:

```text
PMC2 DATA            = 0x68
PMC2 COMMAND/STATUS  = 0x6C
```

The validated configuration:

```text
state = enabled
T1    = 80
T2    = 100
```

stops charging around the displayed 79–80% boundary, permits charging again below that boundary, and persists across a normal reboot.

Only the `80/100` threshold pair has been behaviorally validated. The firmware accepts threshold values from 0 through 100, but the general semantics of arbitrary T1/T2 pairs are not yet established.
