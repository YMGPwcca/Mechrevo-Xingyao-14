# MECHREVO Xingyao 14 / P916F-STX Technical Documentation

This repository is an **exhaustive technical reference** for the MECHREVO Xingyao 14 (机械革命 星耀14) based on the `P916F-STX` platform.

The goal is not to keep the documentation short. The goal is to preserve as much technically useful, reproducible information as possible about this machine: hardware identity, BIOS/UEFI internals, ACPI/WMI behavior, embedded-controller architecture, reverse-engineered protocols, Linux-visible interfaces, OEM software findings, exact firmware artifacts, failed hypotheses, and live validation evidence.

It intentionally contains no driver, utility or application source code. Command/output excerpts, disassembly fragments, register maps and protocol traces are included where they are useful as technical evidence.

## Documentation policy

This repository prefers **complete technical context over aggressive summarization**.

A detail belongs here when it helps another engineer:

- identify this exact platform or firmware revision;
- reproduce or verify a finding;
- understand an interface, register, method, protocol or firmware component;
- distinguish a machine-specific result from a generic OEM implementation;
- understand why a candidate path was accepted or rejected;
- avoid repeating a failed reverse-engineering route;
- continue the investigation from the same evidence base.

Personal desktop preferences and unrelated operating-system customization are out of scope, but technically relevant experiments are not removed merely because they failed. Failed paths are retained when they constrain the implementation or prevent future researchers from repeating the same assumption.

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

## Documentation map

### Platform and firmware

- [`docs/hardware-platform.md`](docs/hardware-platform.md) — hardware identity, display, battery, audio and firmware-version namespaces.
- [`docs/firmware-bios.md`](docs/firmware-bios.md) — BIOS 1.15 package structure, raw ROM, boot graphics, SetupUtility, hidden forms and Dynamic LID.
- [`docs/boot-logo-research.md`](docs/boot-logo-research.md) — detailed analysis of Insyde logo-update mechanisms and the exact P916F implementation.
- [`docs/research-artifacts.md`](docs/research-artifacts.md) — hashes, offsets, binary provenance and artifact relationships.

### Embedded controller and battery

- [`docs/embedded-controller.md`](docs/embedded-controller.md) — IT5571 firmware image, H2RAM mapping, PMC2 transport, XRAM fields and charge-control internals.
- [`docs/battery-charge-limit.md`](docs/battery-charge-limit.md) — reverse-engineered charge-limit protocol, static handler mapping and current semantic model.
- [`docs/validation.md`](docs/validation.md) — live hardware validation, raw outputs and measured behavior.

### ACPI, OEM software and Linux

- [`docs/acpi-wmi.md`](docs/acpi-wmi.md) — ACPI EC map, battery object, OEM control methods, events and Huawei-compatible WMI findings.
- [`docs/control-center.md`](docs/control-center.md) — static findings from MECHREVO Windows Control Center and the generic-vs-P916F distinction.
- [`docs/linux.md`](docs/linux.md) — Linux-visible platform interfaces, power-supply exposure, ACPI/EC integration and missing upstream abstractions.
- [`docs/audio.md`](docs/audio.md) — codec path, logical speaker topology, Windows OEM processing and Linux observations.

### Evidence and unresolved areas

- [`docs/evidence-matrix.md`](docs/evidence-matrix.md) — claim-by-claim confidence and provenance.
- [`docs/reverse-engineering-methodology.md`](docs/reverse-engineering-methodology.md) — how firmware, ACPI and EC findings were derived and cross-checked.
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

The static implementation, host protocol and live evidence are documented separately so readers can distinguish what the firmware code proves from what was actually exercised on hardware.
