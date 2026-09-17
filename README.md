# MECHREVO Xingyao 14 / P916F-STX technical reference

Technical investigation of the MECHREVO Xingyao 14 (机械革命 星耀14): platform identity, Insyde UEFI firmware, ACPI/WMI, the ITE embedded controller, battery charging, thermal interfaces and Linux integration.

The reference preserves register maps, implementation details, artifact identities, original evidence and rejected hypotheses. It contains documentation, not a driver or hardware-control toolkit. [Source coverage](docs/research-sources.md) and [outstanding evidence](docs/documentation-status.md) bound each conclusion.

## Applicability and safety

The principal baseline is **P916F-STX, BIOS 1.15, firmware-reported EC 1.15**. Compatibility with P916F-HPT-R, P916F-ARL, other Xingyao products or later firmware is not established.
`main` is the evolving technical reference. Frozen documentation snapshots use annotated Git tags named `p916f-stx-bios-1.15-reference-YYYY-MM-DD[-rN]`; a snapshot identifies the documentation state at that tag and does not imply that all open research questions are resolved.

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
| Installed storage | `YMTC PC41Q-1TB-B` on the investigated unit |
| Internal display | 2880 × 1800; exact refresh-rate evidence pending |
| System BIOS | `1.15` |
| Firmware-reported EC | `1.15` |
| BIOS build-date string | `05/07/2026`; date format not converted |
| EC silicon | ITE `0x5571`, revision `0x07` |
| Observed Linux environment | CachyOS / Arch-family; exact kernel version pending |

The EC image's internal string `IT557x V1.09 E00 - 20230831` is a separate version namespace. The earlier BIOS 1.09 package is a historical artifact, not proof of identical live behavior. See [hardware identity](docs/hardware-platform.md) for attribution and scope.

## Principal findings

- **Battery charge control:** the EC battery policy uses two independently programmable thresholds with three live-observed regions. In the validated `T1=85%`, `T2=90%` experiment, `SOC < T1` permitted charging; the T1..T2 region settled into `Not charging`; and `SOC > T2` produced sustained multi-watt battery discharge while AC remained online. As SOC returned to the T2 region, the sustained discharge was released and the machine settled back into hold. The earlier `80/100` result is therefore explained as an approximately-80% cap with the active-discharge region effectively unreachable at `T2=100`. Exact Linux-visible comparator timing, arbitrary threshold-pair behavior and the exact charger silicon remain unresolved. A normal reboot retained the programmed state, while one later complete-battery-depletion event returned `Enabled=0`, `T1=0`, `T2=0` on the next powered session. [Protocol](docs/battery-charge-limit.md) · [T1/T2 semantics](docs/battery-threshold-semantics.md) · [Raw validation](docs/validation.md) · [Power-loss observation](docs/battery-limit-power-loss-observation.md).
- **EC access architecture:** standard ACPI EC traffic, SystemMemory-backed H2RAM and PMC2 are distinct interfaces. The documented aperture maps host physical `0xFEEC2300..0xFEEC23FF` to EC XRAM `0x0300..0x03FF`. The candidate I2EC path at I/O `0x380` failed its recorded cross-check. [EC reference](docs/embedded-controller.md).
- **Thermal interfaces:** recovered AML establishes `GFNS`, `GPFM`, `SPFM`, two telemetry fields and Fn+X dispatch. A recovered live capture correlates GFNS results with the two H2RAM `FNS0`/`FNS1` fields; the raw word is **not** promoted to calibrated RPM. The complete recorded `THMM` body includes Balance, Performance and LID ALIB sequences. September `SPFM` uses `0x94`/`0x95`; a recovered April source uses `0x91`/`0x92`, with exact same-session firmware association still unresolved. [Thermal reference](docs/thermal-performance.md).
- **Firmware and setup:** the full identified static BIOS/HII audit is included: 8 formsets, 28 reachability rows, 151 SetupUtility questions/actions, 204 PBS and 416 CBS controls. Static defaults and presence are not live configuration or hardware-support evidence. A recovered photograph directly records the expanded SREP Boot page, while the exact successful patcher build/configuration remains unlinked. The two examined generic logo-update paths did not establish a supported logo-only update mechanism. [Firmware](docs/firmware-bios.md) · [Setup options](docs/bios-setup-options.md) · [Runtime visibility](docs/srep-runtime-reveal.md) · [Boot graphics](docs/boot-logo-research.md).
- **Firmware access:** the retained PSP attribute reports ROM Armor enforcement. It does not establish every protection field or prove that all acquisition methods fail. H2OFFT embedded capabilities are separate from a live acquisition transcript. The exact private raw-ROM and EC source objects are located, but were not rehashed because raw-byte materialization was unavailable in the recovery audit. [Access evidence](docs/firmware-access.md) · [Artifact registry](docs/research-artifacts.md).
- **ACPI and Linux:** direct `WMAA` evaluation returns a two-element package, not an unconditional flat 256-byte buffer. The September ACPI extraction/header capture is recovered, while raw `dsdt.dat` bytes/hash remain pending. The recorded Linux environment exposed battery telemetry but not generic charge-threshold attributes. [ACPI/WMI](docs/acpi-wmi.md) · [Linux](docs/linux.md).
- **Audio and device enumeration:** the recovered Linux capture identifies ALC256, digital/stereo microphone endpoints, ACP/HDA bindings, loaded sound modules and two V4L2 entries named FHD Camera. A recovered `NahimicExport.zip` establishes Windows Nahimic/A-Volute application-level EQ preset tables and Realtek endpoint/APO integration. It does not establish the complete active DSP graph, amplifier programming or physical speaker topology. Logical interfaces likewise do not establish physical camera/speaker counts; standard `platform_profile` exposure remains unresolved. [Audio](docs/audio.md) · [Linux](docs/linux.md).

## Documentation map

| Area | References |
|---|---|
| Platform | [Hardware](docs/hardware-platform.md), [Linux](docs/linux.md), [audio](docs/audio.md) |
| BIOS / UEFI | [Firmware structure](docs/firmware-bios.md), [setup options](docs/bios-setup-options.md), [access](docs/firmware-access.md), [boot graphics](docs/boot-logo-research.md), [runtime setup visibility](docs/srep-runtime-reveal.md) |
| Embedded controller | [Architecture/registers](docs/embedded-controller.md), [thermal interfaces](docs/thermal-performance.md) |
| Battery control | [Charge protocol](docs/battery-charge-limit.md), [T1/T2 semantics](docs/battery-threshold-semantics.md), [power-loss observation](docs/battery-limit-power-loss-observation.md) |
| ACPI / OEM | [ACPI/WMI](docs/acpi-wmi.md), [Control Center analysis](docs/control-center.md) |
| Evidence | [Validation](docs/validation.md), [claim matrix](docs/evidence-matrix.md), [artifacts](docs/research-artifacts.md), [sources](docs/research-sources.md) |
| Research process | [Methodology](docs/reverse-engineering-methodology.md), [tooling](docs/reproduction-tooling.md), [technical questions](docs/open-questions.md), [documentation status](docs/documentation-status.md) |

## Reading conventions

Protocol bytes use hexadecimal `0x` notation: 80% encodes as `0x50`, not `0x80`; 100% encodes as `0x64`. Raw output remains unchanged and is annotated outside evidence blocks. Addresses retain their explicit ROM, updater, extracted PE, EC CODE/XRAM, host physical MMIO, I/O-port or VarStore context.

[Evidence classes](docs/reverse-engineering-methodology.md#evidence-classification) distinguish live observations, static implementation facts, artifact measurements, inference, comparative evidence, rejected hypotheses and untested behavior. Source availability is a separate property; a retained historical result is not a new hardware test.

## Distribution, contributions and license

Vendor executables, raw firmware dumps, EC binaries and proprietary extracted resources are not distributed. The [artifact registry](docs/research-artifacts.md) records available sizes, digests and derivation relationships. A matching digest identifies bytes; it does not certify authenticity, compatibility or safe flashing.

Parts of this documentation were prepared with AI assistance. AI output is not treated as technical evidence; claims remain bounded by the cited primary sources and the repository's evidence classifications. See the [AI-assisted documentation disclosure](NOTICE.md#ai-assisted-documentation) for the full statement.

Original documentation is licensed under [CC BY 4.0](LICENSE). [NOTICE.md](NOTICE.md) defines the third-party boundary. Corrections and evidence contributions should follow [CONTRIBUTING.md](CONTRIBUTING.md).
