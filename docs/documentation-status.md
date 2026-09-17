# Documentation status

The reference covers the identified P916F-STX investigation, not every firmware option or hardware behavior. Evidence classification and source availability are separate; see [methodology](reverse-engineering-methodology.md#evidence-classification). Unresolved behavior is recorded in [open questions](open-questions.md). This register identifies the source material needed to extend coverage without inventing results or repeating hardware experiments.

`NEEDS_SOURCE_EXPORT` means the source has been identified but its full text is unavailable here. `SOURCE_LOCATED` means the exact Project/Library object is known but its raw bytes could not be re-exported in the current audit. `SOURCE_RECOVERED` means an identified source or capture has been recovered sufficiently to support the bounded claim stated here. `NEEDS_EVIDENCE` means the required capture or artifact has not been supplied. `NEEDS_CONFIRMATION` means source association or a decision remains unresolved. `ADMIN_PENDING` concerns repository permissions or settings rather than technical evidence.

## Pending evidence

| ID | Topic and status | Material required | Reference |
|---|---|---|---|
| P01 | Full option audit — SOURCE_IMPORTED / CLOSED | Full authorized S2 raw export recovered. All 8 formset, 28 reachability, 151 SetupUtility, 204 PBS and 416 CBS data rows imported; source defaults remain static. Underlying firmware and live behavior are not certified. | [Setup options](bios-setup-options.md) |
| P02a | THMM source coverage — SOURCE_IMPORTED / CLOSED | Full S3 Project text extraction, `Pasted text(4).txt`, recovered from exact ID `file_00000000177881fd8806584f4122e837`. Complete THMM and query bodies are included; hash identifies export text, not raw File Library bytes. | [Thermal interfaces](thermal-performance.md) |
| P02b | Complete ACPI identity — PARTIAL / NEEDS_EVIDENCE | The September ACPI capture `Pasted text(3).txt` (file ID `file_00000000b85081fdba86540c8c835dd2`) records `acpidump`, `acpixtract`, `iasl -d dsdt.dat`, ACPICA `20251212`, and a DSDT header with length `0x833F` / 33,599 bytes, revision 2, OEM ID `XXXXXX`, OEM table ID `EDK2`, OEM revision `0x00040000`, compiler ID `ACPI`, compiler revision `0x00040000`. Original `dsdt.dat` bytes and their SHA-256 remain unavailable, and the capture alone does not assign a BIOS revision. | [ACPI/WMI](acpi-wmi.md), [sources](research-sources.md#project-sources) |
| P03 | Earlier SPFM mapping — SOURCE_RECOVERED / NEEDS_CONFIRMATION | Exact April source recovered as `Pasted text(16).txt`, file ID `file_00000000f5ac72069ac142ca1b550984`, 7,465 bytes, SHA-256 `21d40ec647d5859a2b6feb19ec206dd296eaba45666d7d3e8bc4b17da5966d34`; it contains `SPFM` using `ECMD(0x91)` / `ECMD(0x92)`. A complete historical `dsdt.dsl`, file ID `file_00000000d828720693d8a857cd39fefd`, was also recovered: 253,103 bytes, SHA-256 `43f4b40e70ac867416b9137e3d41ae12ead76224038dcfb6a4943bfc41466296`, DSDT length `0x836B`. Exact same-session firmware association remains unresolved; the April map must not be merged with the September `0x94` / `0x95` map. | [Thermal interfaces](thermal-performance.md), [sources](research-sources.md#project-sources) |
| P04a | Live GFNS/H2RAM correlation — SOURCE_RECOVERED / CLOSED | `Pasted text(7).txt`, file ID `file_000000004e1081fdb6c0290bf45d182f`, contains a direct H2RAM read at `0xFEEC233B..0xFEEC233E` (`91 0f cd 0e`) followed by live WMI GFNS calls. Selector 0 returned result bytes `99 0f`; selector 1 returned `e1 0e`, each with method status `0x00`. The nearby reads differ slightly because they were sequential, but they establish live correlation between GFNS and the two dynamic FNS fields. No independent RPM calibration is established. | [Thermal interfaces](thermal-performance.md) |
| P04b | Live Fn+X / GPFM profile transition — NEEDS_EVIDENCE | Original GPFM/Fn+X invocation, output and initial conditions for the previously reported `0x02 -> 0x01` transition. Do not reconstruct the historical log from memory. | [Thermal interfaces](thermal-performance.md) |
| P05 | EC tachometer/PWM/target tables — NEEDS_EVIDENCE | Exact EC parent digest, bank and address space, source bytes/disassembly and decoding rationale. ALIB parameters are not RPM tables. | [EC](embedded-controller.md), [thermal interfaces](thermal-performance.md) |
| P06 | Decoded BMOF schema — NEEDS_EVIDENCE | Original decode of `WQBA.bmof`, tool context and class/method declarations. AML framing alone does not establish extended WMI buffer sizes. | [ACPI/WMI](acpi-wmi.md) |
| P07 | Complete flashrom failure — NEEDS_EVIDENCE | Original command, terminal result, tool version and environment. The retained ROM Armor line and analysis reports are not the complete diagnostic capture. | [Firmware access](firmware-access.md) |
| P08 | PSP attributes — NEEDS_EVIDENCE | Original multi-attribute output for anti-rollback, RPMC and PSP versions. `rom_armor_enforced:1` does not establish those values. | [Firmware access](firmware-access.md) |
| P09 | H2OFFT execution and IHISI versions — NEEDS_EVIDENCE | Original invocation/output, utility and onboard IHISI version readings, and association with the acquired image. Embedded help and PE versions establish static utility properties only. | [Tooling](reproduction-tooling.md) |
| P10 | Protected regions and DXE equality — NEEDS_EVIDENCE | Original `-pq` output; both raw/updater parent identities, slicing ranges, lengths and comparison output or slice hashes. No inferred region map or new dump is required. | [Firmware access](firmware-access.md), [artifacts](research-artifacts.md) |
| P11 | Exact boot GIF extraction — NEEDS_EVIDENCE | Identified resource bytes or extraction record with digest, size, container chain and offset origins. Existing geometry/frame metadata remains recorded evidence. | [Boot graphics](boot-logo-research.md) |
| P12 | Live setup overlay — NEEDS_EVIDENCE | Original variable captures beyond Quiet Boot, identifying variable/VarStore, GUID, offset and width. IFR defaults cannot fill this gap. | [Setup options](bios-setup-options.md) |
| P13 | Successful SREP configuration — PHOTO_RECOVERED / NEEDS_CONFIRMATION | The post-reveal BIOS photograph is recovered as `image-1789545520742.jpg`, file ID `file_00000000969082309523c0a97474cb7f`, 373,251 bytes, SHA-256 `38b11bb1a736a9373632c78b1949128164cfb7a842b0c67068196cc1fa5d52db`; it visibly contains the revealed Boot page. Two candidate Quiet-Boot configs are also located, but the exact patcher build, final configuration used and session-to-photo association remain unproven. | [Runtime visibility](srep-runtime-reveal.md) |
| P14 | Speaker layout and OEM tuning — PARTIAL / NEEDS_EVIDENCE | `NahimicExport.zip` is recovered and independently hashed: 20,004,555 bytes, SHA-256 `4099d9631368deff8bc39ee07a948134097d56b75db39f6e8b0ff3b0a9de0906`. It contains Nahimic/A-Volute application state, APO/service registry evidence, Realtek endpoint integration and ten-band EQ preset files; `EQPresets.json` records Communication=`Custom`, Gaming=`Custom`, Movie=`Dialogues`, Music=`Dynamic`. This recovers application-level EQ state and preset tables, not the complete active DSP graph, algorithm internals, amplifier programming, physical driver count or wiring. Those remain unresolved. | [Audio](audio.md) |
| P15a | Device/audio enumeration — SOURCE_IMPORTED / CLOSED | Full S11 raw export, `Pasted text(71).txt`, exact ID `file_000000007c24720bb23039daf2e6b088`: V4L2 entries, digital/stereo microphones, ALSA/PCI bindings, loaded modules, PipeWire server `1.6.7` and bounded `xingyao.fw` log observation. Not physical camera/speaker topology or patch contents. | [Audio](audio.md), [Linux](linux.md) |
| P15b | Environment and remaining inventory — PARTIAL / NEEDS_EVIDENCE | An exact-machine boot capture identifies the installed storage device as `YMTC PC41Q-1TB-B`. Exact kernel, ALSA and installed WirePlumber package versions, panel model/EDID/refresh and adapter identity/rating remain unresolved. PipeWire client version displays are not package-version captures. | [Hardware](hardware-platform.md), [Linux](linux.md) |
| P15c | Standard platform profile — NEEDS_EVIDENCE | Original `platform_profile` sysfs inspection associated with this unit/kernel. OEM GPFM does not establish standard ABI exposure. | [Linux](linux.md#standard-profile-portability) |
| P16 | Current raw ROM and EC source objects — SOURCE_LOCATED / REHASH_PENDING | Library objects are located: `P916F-STX-current-ROM.bin`, file ID `file_00000000004c8206847bb2992bb1feaa`, 33,554,432 bytes; and `P916F-IT5571-EC-1.09.bin`, file ID `file_00000000da5882069264633b870b5570`, 131,072 bytes. Raw-byte materialization was not authorized in this audit, so historical SHA-256 values were not independently recomputed. They are retained privately, not publicly distributed. | [Artifacts](research-artifacts.md) |
| P17a | `P916F-charge-reverse.tar.gz` parent — SOURCE_LOCATED / REHASH_PENDING | Exact Library object located: file ID `file_0000000099ac8209b2e8bf87fb8aec70`, 10,092,171 bytes. Raw-byte materialization was not authorized in this audit, so the bundle SHA-256 remains unretained; child hashes must not substitute for it. | [Artifacts](research-artifacts.md) |
| P17b | `STX_SKU2_1.15.zip` outer archive — NEEDS_EVIDENCE | Exact outer ZIP bytes or original digest record. The measured nested `STX_SKU2_1.15.exe` digest does not identify the missing outer archive. | [Artifacts](research-artifacts.md) |

Private source identifiers are retrieval locators, not public download links. The absence of a vendor download URL is not a documentation defect. Proprietary binaries, private machine dumps and private original reports remain outside the distributed repository.

### Recovery assessment scope

P02, P04, P15 and P17 remain parent identifiers for their explicit subitems above. Reassessment covers the supplied full exports, retained AML/BIOS excerpts, recovered April DSDT material, the live GFNS/H2RAM capture, the recovered SREP photograph and candidate configuration records, the Nahimic export, source index, ROM Armor line, original mounted analysis reports and offline artifact inventories available with this revision. Locating a private Library object is not equivalent to rehashing its raw bytes.

| Gates reviewed | Available source result | Missing evidence retained |
|---|---|---|
| P01 | Full S2 inventory found and imported | No remaining audit-import gap; P12 remains separate |
| P02 | Full S3 text and complete THMM found; September ACPI command/header capture recovered | Original `dsdt.dat` bytes/hash and exact BIOS association remain absent |
| P03 | Original April `0x91/0x92` source and complete historical `dsdt.dsl` recovered | Exact same-session firmware identity remains unresolved |
| P04–P05 | Live GFNS/H2RAM correlation recovered; AML fields, dispatch and ALIB parameters retained | Fn+X/GPFM live transition and EC tach/PWM/table decoding remain absent |
| P06 | BMOF prefix/size and AML framing found | No complete decoded class/method schema |
| P07–P08 | ROM Armor line and mounted explanatory report found | No complete flashrom transcript or multi-attribute PSP capture |
| P09–P10 | Offline H2OFFT metadata and package identities found | No live IHISI/version/region output or raw/updater slice comparison |
| P11 | Logo report with resource identity and geometry found | No GIF bytes/digest/complete extraction record |
| P12–P13 | Static defaults, Quiet Boot observation, candidate configs and the revealed Boot-page photograph found | No additional live overlay or exact successful build/config/session association |
| P14–P15 | Linux device capture plus Nahimic/A-Volute export and installed SSD model recovered | No physical speaker topology, complete DSP/amplifier graph, remaining environment data or standard-profile capture |
| P16 | Exact private ROM and EC Library objects located with expected sizes | Raw-byte export was unavailable, so no new digest verification |
| P17 | Charge-reverse parent located; nested 1.15 executable remains measured | Charge-reverse rehash and original `STX_SKU2_1.15.zip` identity remain unresolved |
| P18 | Existing bounded external-reference checks retained | No new external claim introduced by recovery |
| P19–P20 | Existing administrative observations retained | Source recovery changes no owner-workflow or protection decision |

Publication state is tracked by actual Git history and pull requests rather than treated as hardware evidence. A merged revision or snapshot tag does not imply that the remaining research gates are closed.

## Reference checks

P18 is resolved for the external references used in this revision: CC BY 4.0 legal terms, deed and licensing-scope guidance were compared; Linux power-supply unit definitions were checked against the sysfs ABI and the retained telemetry. These checks do not establish hardware compatibility, current kernel support or rights in third-party firmware. See [external references](research-sources.md).

## Administration

| ID | Topic | State |
|---|---|---|
| P19 | Description, topics and repository features | Description and 14 relevant topics updated and read back. Issues remain enabled. Wiki and Projects remain enabled: the Wiki endpoint exposed no content, but usage was not conclusively established; Projects inspection requires the unavailable `read:project` token scope. Feature removal remains ADMIN_PENDING rather than assuming unused content. |
| P20 | Optional branch protection | NEEDS_CONFIRMATION. `main` was observed unprotected. No protection, approval requirement or mandatory CI was introduced without an agreed owner workflow. |

Documentation commits and snapshot tags identify the text revision, not completion of the pending research. Publication identity is recorded in Git history and the delivery report rather than a self-referential commit hash in this file.
