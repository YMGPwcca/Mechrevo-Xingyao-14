# Documentation status

The reference covers the identified P916F-STX investigation, not every firmware option or hardware behavior. Evidence classification and source availability are separate; see [methodology](reverse-engineering-methodology.md#evidence-classification). Unresolved behavior is recorded in [open questions](open-questions.md). This register identifies the source material needed to extend coverage without inventing results or repeating hardware experiments.

`NEEDS_SOURCE_EXPORT` means the source has been identified but its full text is unavailable here. `NEEDS_EVIDENCE` means the required capture or artifact has not been supplied. `NEEDS_CONFIRMATION` means source association or a decision remains unresolved. `ADMIN_PENDING` concerns repository permissions or settings rather than technical evidence.

## Pending evidence

| ID | Topic and status | Material required | Reference |
|---|---|---|---|
| P01 | Full option audit — SOURCE_IMPORTED / CLOSED | Full authorized S2 raw export recovered. All 8 formset, 28 reachability, 151 SetupUtility, 204 PBS and 416 CBS data rows imported; source defaults remain static. Underlying firmware and live behavior are not certified. | [Setup options](bios-setup-options.md) |
| P02a | THMM source coverage — SOURCE_IMPORTED / CLOSED | Full S3 Project text extraction, `Pasted text(4).txt`, recovered from exact ID `file_00000000177881fd8806584f4122e837`. Complete THMM and query bodies are included; hash identifies export text, not raw File Library bytes. | [Thermal interfaces](thermal-performance.md) |
| P02b | Complete ACPI identity — NEEDS_EVIDENCE | Original DSDT header, complete table bytes/export and SHA-256 associated with S3/S4/S5. S3 contains concatenated ranges and discontinuities, not a complete DSDT. Collection timestamps do not establish BIOS revision. | [ACPI/WMI](acpi-wmi.md), [sources](research-sources.md#project-sources) |
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
| P15a | Device/audio enumeration — SOURCE_IMPORTED / CLOSED | Full S11 raw export, `Pasted text(71).txt`, exact ID `file_000000007c24720bb23039daf2e6b088`: V4L2 entries, digital/stereo microphones, ALSA/PCI bindings, loaded modules, PipeWire server `1.6.7` and bounded `xingyao.fw` log observation. Not physical camera/speaker topology or patch contents. | [Audio](audio.md), [Linux](linux.md) |
| P15b | Environment and remaining inventory — NEEDS_EVIDENCE | Exact kernel, ALSA and installed WirePlumber package versions, storage identifiers, panel model/EDID/refresh and adapter identity/rating. PipeWire client version displays are not package-version captures. | [Hardware](hardware-platform.md), [Linux](linux.md) |
| P15c | Standard platform profile — NEEDS_EVIDENCE | Original `platform_profile` sysfs inspection associated with this unit/kernel. No such capture occurs in S2/S3/S11; OEM GPFM does not establish standard ABI exposure. | [Linux](linux.md#standard-profile-portability) |
| P16 | Current raw ROM and EC rehash — NEEDS_EVIDENCE | Exact raw-ROM/current 128 KiB EC source bytes corresponding to the recorded digests. Historical identities remain valid as retained reports, not new measurements. | [Artifacts](research-artifacts.md) |
| P17 | Outer archive identities — NEEDS_EVIDENCE | Exact `STX_SKU2_1.15.zip` and `P916F-charge-reverse.tar.gz` bytes or original digest records. Child EXE/member hashes do not identify their parents. | [Artifacts](research-artifacts.md) |

Private source identifiers are retrieval locators, not public download links. The absence of a vendor download URL is not a documentation defect. Proprietary binaries and private original reports remain outside the distributed repository.

### Recovery assessment scope

P02 and P15 remain parent identifiers for their explicit subitems above. Reassessment covers the three supplied full exports, retained AML/BIOS excerpts, source index, SREP candidate, ROM Armor line, original mounted analysis reports and offline artifact inventories available with this revision. It does not claim access to unexported Project collection records.

| Gates reviewed | Available source result | Missing evidence retained |
|---|---|---|
| P01 | Full S2 inventory found and imported | No remaining audit-import gap; P12 remains separate |
| P02 | Full S3 text and complete THMM found; table identity only partially present | Complete DSDT/header/binary identity absent |
| P03 | Older mapping survives as source-locator/report only | No original April firmware association |
| P04–P05 | AML fields, dispatch and ALIB parameters found | No raw fan invocation/transition or EC tach/PWM/table decoding |
| P06 | BMOF prefix/size and AML framing found | No complete decoded class/method schema |
| P07–P08 | ROM Armor line and mounted explanatory report found | No complete flashrom transcript or multi-attribute PSP capture |
| P09–P10 | Offline H2OFFT metadata and package identities found | No live IHISI/version/region output or raw/updater slice comparison |
| P11 | Logo report with resource identity and geometry found | No GIF bytes/digest/complete extraction record |
| P12–P13 | Static defaults, retained Quiet Boot observation and SREP candidate found | No additional live overlay or successful build/config/session association |
| P14–P15 | S11 interface capture recovered, including codec autoconfiguration and patch filename | No physical topology, OEM coefficients, patch contents, remaining environment data or standard-profile capture |
| P16–P17 | Historical ROM/EC and selected package/member digests found | No exact raw ROM/current EC or missing outer-archive identity material |
| P18 | Existing bounded external-reference checks retained | No new external claim introduced by recovery |
| P19–P20 | Existing administrative observations retained | Source exports add no permission or owner-workflow decision; settings unchanged |

P21 publication is tracked by the actual commit and pull request, not treated as hardware evidence. Submission for review does not imply merge or a new main-branch snapshot tag.

## Reference checks

P18 is resolved for the external references used in this revision: CC BY 4.0 legal terms, deed and licensing-scope guidance were compared; Linux power-supply unit definitions were checked against the sysfs ABI and the retained telemetry. These checks do not establish hardware compatibility, current kernel support or rights in third-party firmware. See [external references](research-sources.md).

## Administration

| ID | Topic | State |
|---|---|---|
| P19 | Description, topics and repository features | Description and 14 relevant topics updated and read back. Issues remain enabled. Wiki and Projects remain enabled: the Wiki endpoint exposed no content, but usage was not conclusively established; Projects inspection requires the unavailable `read:project` token scope. Feature removal remains ADMIN_PENDING rather than assuming unused content. |
| P20 | Optional branch protection | NEEDS_CONFIRMATION. `main` was observed unprotected. No protection, approval requirement or mandatory CI was introduced without an agreed owner workflow. |

Documentation commits and snapshot tags identify the text revision, not completion of the pending research. Publication identity is recorded in Git history and the delivery report rather than a self-referential commit hash in this file.
