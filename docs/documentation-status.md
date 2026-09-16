# Documentation status

The reference covers the identified P916F-STX investigation, not every firmware option or hardware behavior. Evidence classification and source availability are separate; see [methodology](reverse-engineering-methodology.md#evidence-classification). Unresolved behavior is recorded in [open questions](open-questions.md). This register identifies the source material needed to extend coverage without inventing results or repeating hardware experiments.

`NEEDS_SOURCE_EXPORT` means the source has been identified but its full text is unavailable here. `NEEDS_EVIDENCE` means the required capture or artifact has not been supplied. `NEEDS_CONFIRMATION` means source association or a decision remains unresolved. `ADMIN_PENDING` concerns repository permissions or settings rather than technical evidence.

## Pending evidence

| ID | Topic and status | Material required | Reference |
|---|---|---|---|
| P01 | Full option audit — NEEDS_SOURCE_EXPORT | `P916F-STX_BIOS_1.15_full_option_audit.md`, File Library `file_00000000179c8206a3dc01afcd82478c` (SRC-AUDIT). Export the complete identified source, not a new reverse-engineering audit. The available 8 option, 10 reachability and 8 formset rows do not constitute the source-reported 204 PBS / 416 CBS controls. | [Setup options](bios-setup-options.md) |
| P02 | Complete ACPI identity — NEEDS_EVIDENCE | Original table header, full export and SHA-256 corresponding to SRC-AML-A/B/C/D. Collection timestamps do not establish BIOS revision. Exact source IDs are in the source register. | [ACPI/WMI](acpi-wmi.md), [sources](research-sources.md#project-sources) |
| P03 | Earlier SPFM mapping — NEEDS_CONFIRMATION | Identify the firmware/table associated with `Pasted text.txt`, collection timestamp `2026-04-22T03:13:44Z` (SRC-AML-OLD), before reconciling its reported `0x91/0x92` with September excerpt `0x94/0x95`. | [Thermal interfaces](thermal-performance.md) |
| P04 | Live fan/profile observations — NEEDS_EVIDENCE | Original GFNS/GPFM and Fn+X invocation, output and initial conditions associated with the recorded observation around `2026-09-16T03:22:20Z`; no reconstructed log. | [Thermal interfaces](thermal-performance.md) |
| P05 | EC tachometer/PWM/target tables — NEEDS_EVIDENCE | Exact EC parent digest, bank and address space, source bytes/disassembly and decoding rationale. ALIB parameters are not RPM tables. | [EC](embedded-controller.md), [thermal interfaces](thermal-performance.md) |
| P06 | Decoded BMOF schema — NEEDS_EVIDENCE | Original decode of `WQBA.bmof`, tool context and class/method declarations. AML framing alone does not establish extended WMI buffer sizes. | [ACPI/WMI](acpi-wmi.md) |
| P07 | Complete flashrom failure — NEEDS_EVIDENCE | Original command, terminal result, tool version and environment. The retained ROM Armor line and analysis reports are not the complete diagnostic capture. | [Firmware access](firmware-access.md) |
| P08 | PSP attributes — NEEDS_EVIDENCE | Original multi-attribute output for anti-rollback, RPMC and PSP versions. `rom_armor_enforced:1` does not establish those values. | [Firmware access](firmware-access.md) |
| P09 | H2OFFT execution and IHISI versions — NEEDS_EVIDENCE | Original invocation/output, utility and onboard IHISI version readings, and association with the acquired image. Embedded help and PE versions establish static utility properties only. | [Tooling](reproduction-tooling.md) |
| P10 | Protected regions and DXE equality — NEEDS_EVIDENCE | Original `-pq` output; both raw/updater parent identities, slicing ranges, lengths and comparison output or slice hashes. No inferred region map or new dump is required. | [Firmware access](firmware-access.md), [artifacts](research-artifacts.md) |
| P11 | Exact boot GIF extraction — NEEDS_EVIDENCE | Identified resource bytes or extraction record with digest, size, container chain and offset origins. Existing geometry/frame metadata remains recorded evidence. | [Boot graphics](boot-logo-research.md) |
| P12 | Live setup overlay — NEEDS_EVIDENCE | Original variable captures beyond Quiet Boot, identifying variable/VarStore, GUID, offset and width. IFR defaults cannot fill this gap. | [Setup options](bios-setup-options.md) |
| P13 | Successful SREP configuration — NEEDS_CONFIRMATION | Exact build, final configuration digest and session/photo/result association. Candidate configuration text is not a validated session record. | [Runtime visibility](srep-runtime-reveal.md) |
| P14 | Speaker layout and OEM tuning — NEEDS_EVIDENCE | Attributed product specification or physical inspection for driver count; OEM processing coefficients, amplifier configuration and channel routing for tuning. Logical FL/FR alone establishes neither. | [Audio](audio.md) |
| P15 | Device inventory and environment — NEEDS_EVIDENCE | Correct-machine camera/microphone/module enumeration, kernel and audio-stack versions, display EDID/refresh information and adapter identity/rating. A retained `platform_profile` sysfs capture is also required before asserting presence or absence. | [Hardware](hardware-platform.md), [Linux](linux.md) |
| P16 | Current raw ROM and EC rehash — NEEDS_EVIDENCE | Exact raw-ROM/current 128 KiB EC source bytes corresponding to the recorded digests. Historical identities remain valid as retained reports, not new measurements. | [Artifacts](research-artifacts.md) |
| P17 | Outer archive identities — NEEDS_EVIDENCE | Exact `STX_SKU2_1.15.zip` and `P916F-charge-reverse.tar.gz` bytes or original digest records. Child EXE/member hashes do not identify their parents. | [Artifacts](research-artifacts.md) |

Private source identifiers are retrieval locators, not public download links. The absence of a vendor download URL is not a documentation defect. Proprietary binaries and private original reports remain outside the distributed repository.

## Reference checks

P18 is resolved for the external references used in this revision: CC BY 4.0 legal terms, deed and licensing-scope guidance were compared; Linux power-supply unit definitions were checked against the sysfs ABI and the retained telemetry. These checks do not establish hardware compatibility, current kernel support or rights in third-party firmware. See [external references](research-sources.md).

## Administration

| ID | Topic | State |
|---|---|---|
| P19 | Description, topics and repository features | Description and 14 relevant topics updated and read back. Issues remain enabled. Wiki and Projects remain enabled: the Wiki endpoint exposed no content, but usage was not conclusively established; Projects inspection requires the unavailable `read:project` token scope. Feature removal remains ADMIN_PENDING rather than assuming unused content. |
| P20 | Optional branch protection | NEEDS_CONFIRMATION. `main` was observed unprotected. No protection, approval requirement or mandatory CI was introduced without an agreed owner workflow. |

Documentation commits and snapshot tags identify the text revision, not completion of the pending research. Publication identity is recorded in Git history and the delivery report rather than a self-referential commit hash in this file.
