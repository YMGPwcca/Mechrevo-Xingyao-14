# MECHREVO Xingyao 14 / P916F-STX technical reference

Technical investigation of the MECHREVO Xingyao 14 (机械革命 星耀14): platform identity, Insyde UEFI firmware, ACPI/WMI, the ITE embedded controller, battery charging, thermal interfaces and Linux integration.

The reference preserves register maps, implementation details, artifact identities, original evidence and rejected hypotheses. It contains documentation, not a driver or hardware-control toolkit. [Source coverage](docs/research-sources.md) and [outstanding evidence](docs/documentation-status.md) bound each conclusion.

## Applicability and safety

The principal baseline is **P916F-STX, BIOS 1.15, firmware-reported EC 1.15**. Compatibility with P916F-HPT-R, P916F-ARL, other Xingyao products or later firmware is not established.

Offline file inspection does not access the laptop. Hardware state queries may still require I/O-port writes; EC setters can change charging or thermal state. Saving setup changes can write NVRAM, while SPI flashing modifies firmware storage. These are distinct operations with distinct effects. A successful getter or a static implementation does not validate a setter. Untested setters and permanent patch landmarks are research evidence, not validated operating procedures.

## Reference platform

| Component | Recorded value |
|---|---|
| Product | MECHREVO Xingyao 14 / 机械革命 星耀14 |
| Platform string | `MECHREVO XINGYAO Series-P916F-STX` |
| Processor | AMD Ryzen AI 9 365, Strix Point |
| Processor topology | 10 cores / 20 threads, retained platform specification |
| Integrated graphics | Radeon 880M |
| Installed memory | 32 GiB on the investigated unit |
| Internal display | 2880 × 1800; exact refresh-rate evidence pending |
| System BIOS | `1.15` |
| Firmware-reported EC | `1.15` |
| BIOS build-date string | `05/07/2026`; date format not converted |
| EC silicon | ITE `0x5571`, revision `0x07` |
| Observed Linux environment | CachyOS / Arch-family; exact kernel version pending |

The EC image's internal string `IT557x V1.09 E00 - 20230831` is a separate version namespace. The earlier BIOS 1.09 package is a historical artifact, not proof of identical live behavior. See [hardware identity](docs/hardware-platform.md) for attribution and scope.

## Principal findings

- **Battery charge control:** the recorded PMC2 experiment at I/O ports `0x68`/`0x6C` established an enabled `T1=80%`, `T2=100%` configuration that stopped charging near the displayed 79–80% boundary and survived a normal reboot. Arbitrary threshold pairs, T2 semantics and complete EC-power-loss persistence remain unresolved. [Protocol](docs/battery-charge-limit.md) · [Raw validation](docs/validation.md).
- **EC access architecture:** standard ACPI EC traffic, SystemMemory-backed H2RAM and PMC2 are distinct interfaces. The documented aperture maps host physical `0xFEEC2300..0xFEEC23FF` to EC XRAM `0x0300..0x03FF`. The candidate I2EC path at I/O `0x380` failed its recorded cross-check. [EC reference](docs/embedded-controller.md).
- **Thermal interfaces:** DSDT excerpts establish `GFNS`, `GPFM`, `SPFM`, two fan-telemetry fields and the Fn+X dispatch. `SPFM` calls `THMM` before profile-specific command validation; a failure result does not prove no side effects. ALIB parameters are not measured RPM curves. [Thermal reference](docs/thermal-performance.md).
- **Firmware and setup:** the investigation identifies the boot animation, SetupUtility and separate AMD PBS/CBS formsets. The setup page preserves the available audit rows, not the complete option inventory. The two examined generic logo-update paths did not establish a supported logo-only update mechanism. [Firmware](docs/firmware-bios.md) · [Setup options](docs/bios-setup-options.md) · [Boot graphics](docs/boot-logo-research.md).
- **Firmware access:** the retained PSP attribute reports ROM Armor enforcement. It does not establish every protection field or prove that all acquisition methods fail. H2OFFT embedded capabilities are separate from a live acquisition transcript. [Access evidence](docs/firmware-access.md).
- **ACPI and Linux:** direct `WMAA` evaluation returns a two-element package, not an unconditional flat 256-byte buffer. The recorded Linux environment exposed battery telemetry but not generic charge-threshold attributes. [ACPI/WMI](docs/acpi-wmi.md) · [Linux](docs/linux.md).
- **Audio:** ALC256 playback and a logical stereo FL/FR endpoint were observed. The reported physical four-driver layout lacks independent source attribution; Windows-equivalent OEM tuning remains unresolved. [Audio reference](docs/audio.md).

## Documentation map

| Area | References |
|---|---|
| Platform | [Hardware](docs/hardware-platform.md), [Linux](docs/linux.md), [audio](docs/audio.md) |
| BIOS / UEFI | [Firmware structure](docs/firmware-bios.md), [setup options](docs/bios-setup-options.md), [access](docs/firmware-access.md), [boot graphics](docs/boot-logo-research.md), [runtime setup visibility](docs/srep-runtime-reveal.md) |
| Embedded controller | [Architecture/registers](docs/embedded-controller.md), [charge protocol](docs/battery-charge-limit.md), [thermal interfaces](docs/thermal-performance.md) |
| ACPI / OEM | [ACPI/WMI](docs/acpi-wmi.md), [Control Center analysis](docs/control-center.md) |
| Evidence | [Validation](docs/validation.md), [claim matrix](docs/evidence-matrix.md), [artifacts](docs/research-artifacts.md), [sources](docs/research-sources.md) |
| Research process | [Methodology](docs/reverse-engineering-methodology.md), [tooling](docs/reproduction-tooling.md), [technical questions](docs/open-questions.md), [documentation status](docs/documentation-status.md) |

## Reading conventions

Protocol bytes use hexadecimal `0x` notation: 80% encodes as `0x50`, not `0x80`; 100% encodes as `0x64`. Raw output remains unchanged and is annotated outside evidence blocks. Addresses retain their explicit ROM, updater, extracted PE, EC CODE/XRAM, host physical MMIO, I/O-port or VarStore context.

[Evidence classes](docs/reverse-engineering-methodology.md#evidence-classification) distinguish live observations, static implementation facts, artifact measurements, inference, comparative evidence, rejected hypotheses and untested behavior. Source availability is a separate property; a retained historical result is not a new hardware test.

## Distribution, contributions and license

Vendor executables, raw firmware dumps, EC binaries and proprietary extracted resources are not distributed. The [artifact registry](docs/research-artifacts.md) records available sizes, digests and derivation relationships. A matching digest identifies bytes; it does not certify authenticity, compatibility or safe flashing.

Original documentation is licensed under [CC BY 4.0](LICENSE). [NOTICE.md](NOTICE.md) defines the third-party boundary. Corrections and evidence contributions should follow [CONTRIBUTING.md](CONTRIBUTING.md).
