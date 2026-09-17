# Research source register

This register identifies the sources used by the technical reference and separates source coverage from claim validation. File IDs are provenance locators for the project collection; they are not public proof by themselves. Public readers should use the linked technical pages.

## Project sources

| Stable ID | Exact source ID(s) and locator | Scope | Retention / coverage | Export identity | Public excerpt or canonical page |
|---|---|---|---|---|---|
| **S1** | `SRC-BASELINE` | Repository baseline, platform/firmware/EC/battery/audio reports | 15 complete files; byte identity verified by Git blob SHA-1 | not retained | [Baseline technical pages](validation.md) |
| **S2** | `SRC-AUDIT`; `P916F-STX_BIOS_1.15_full_option_audit.md`; file ID `file_00000000179c8206a3dc01afcd82478c`; collection locator `2026-09-13T20:57:03Z` | Complete static BIOS/HII audit | Full authorized raw-file export recovered; all five inventories published: 8 formsets, 28 reachability, 151 SetupUtility, 204 PBS, 416 CBS rows | Raw export SHA-256 `525c46baa22a0d868b56c36d4094cc13f314bdfccba239f4332c3159983e5271`, 140821 bytes, 859 lines | [Setup inventory](bios-setup-options.md); source risk annotations are not measured safety |
| **S3** | `SRC-AML-C`; `Pasted text(4).txt`; file ID `file_00000000177881fd8806584f4122e837`; collection locator `2026-09-16T08:31:21Z` | EC fields, complete THMM, query handlers, GFNS/GPFM/SPFM and keyboard helpers; Linux battery/sysfs capture | Full Project text extraction recovered; complete THMM and selected technical records published. Concatenated source ranges include discontinuities and repeated helpers, not a complete DSDT | Text-export SHA-256 `fd33e0fc94a69c558156ae82e124a213f891627daa788fc65d84e03c0d73cc16`, 33532 bytes, 938 logical text lines (937 LF terminators; no final newline); **not original raw File Library bytes** | [ACPI/WMI](acpi-wmi.md); [thermal interfaces](thermal-performance.md); [Linux](linux.md) |
| **S4** | `SRC-AML-A`; file ID `file_000000004f6081f5a04c617f93a5f64a`; collection locator `2026-09-16T03:18:50Z`<br>`SRC-AML-D`; file ID `file_00000000fd5881fd832a0b6f24276f92`; collection locator `2026-09-16T08:36:42Z`<br>`SRC-AML-D-DUP`; file ID `file_000000000bd081fd9564629597095188`; collection locator `2026-09-16T08:36:45Z` | WMAA dispatcher/package return and related method/search excerpts; duplicate upload is not independent | Selected GFNS/GVER and WMAA excerpts; original DSDT header/hash not included<br>WMAA beginning and return; grep locators<br>Duplicate of nearby capture; not an independent hardware test | `d527042e21d431f91fffeece6375e45a9740682a4121d32fb52f71c224afde65` (11859 bytes)<br>`d527042e21d431f91fffeece6375e45a9740682a4121d32fb52f71c224afde65` (11859 bytes)<br>not retained | [ACPI/WMI](acpi-wmi.md) |
| **S5** | `SRC-AML-B`; file ID `file_00000000619c82309ea7d3b9cb99ce73`; collection locator `2026-09-16T03:23:55Z` | Numbered GPFM/SPFM/GKBT/SKBT excerpts with the September ECMD(0x94)/ECMD(0x95) pair | GPFM/SPFM/GKBT/SKBT numbered source excerpts | `d527042e21d431f91fffeece6375e45a9740682a4121d32fb52f71c224afde65` (11859 bytes) | [Thermal interfaces](thermal-performance.md); [ACPI/WMI](acpi-wmi.md) |
| **S6** | `SRC-AML-OLD`; no exported file; collection locator `2026-04-22T03:13:44Z` | Earlier 0x91/0x92 lead without complete source identity | Earlier draft references SPFM 0x91/0x92. Complete original not included, firmware association unresolved | not retained | [Documentation status, P03](documentation-status.md#pending-evidence) |
| **S7** | `SRC-SREP`; file ID `file_000000003b188206acc9a48fac2fcb61`; collection locator `2026-09-13T20:57:09Z` | Retained candidate SREP configuration and historical runtime reveal report | Complete displayed text; original file-byte hash not verified; inert evidence only | `c115d16dd5d15f2e1ffe3e8c83c79c59e684d90261236df13021bd76e20c110e` (399 bytes) | [SREP runtime reveal](srep-runtime-reveal.md) |
| **S8** | `SRC-ROMARMOR` | One retained ROM Armor sysfs line; no complete flashrom transcript | One exact retained sysfs output line, no complete command/result bundle | `1cab283146026ae2ac7fded1e874f7c62772bb2b5c008f55b721682ec8a377f3` (55 bytes) | [Firmware access](firmware-access.md) |
| **S9** | `SRC-BINARIES`<br>`SRC-SFX` | Seven measured package uploads, selected members, SFX structure and H2OFFT metadata | Seven uploaded vendor packages sized and hashed; selected ZIP members measured; binaries not distributed<br>SFX signature, member listing/digests, earlier EC carve and H2OFFT numeric version independently checked without running vendor executables | `06d258dc27015a0ab260f04326fd373a8c29867d7d567276cbfe401f8e9113ab` (87468 bytes)<br>`13362ac89bd608372c1377cca55e5f3a0746fa0c46aa2a61ad20b6af1cac7ca4` (4511 bytes) | [Artifact registry](research-artifacts.md#uploaded-package-identities); [offline tooling](reproduction-tooling.md#offline-reproduction-examples) |
| **S10** | `SRC-LOGO`; file ID `file_000000008450821183b4e6828120d16f` | Retained logo-route analysis report and static P916F landmarks | Complete mounted bytes; secondary report, not original binary/disassembler capture | `adc1fd13a21bc16bcb1b84d320fd451a9f1b737ebb5c9bcf94301a6668ed9264` (4778 bytes) | [Boot-logo research](boot-logo-research.md) |
| **S11** | `SRC-DEVICE-AUDIO`; `Pasted text(71).txt`; file ID `file_000000007c24720bb23039daf2e6b088` | Linux PipeWire/V4L2, ALSA, PCI audio, loaded modules and kernel audio log | Full authorized raw-file export recovered; curated enumeration and message payloads published, personal shell/host metadata omitted. Platform string identifies P916F-STX; BIOS and kernel revision not established by this capture | Raw export SHA-256 `994d5a0b7df61bccec35d1ef6bb7a7818aa6e9ec33dbef430a5b6307ac1ee0a0`, 12706 bytes, 210 logical text lines (209 LF terminators; no final newline) | [Audio](audio.md#recovered-device-and-kernel-capture); [Linux inventory](linux.md#audio-and-peripheral-inventory) |

Collection timestamps in this table identify similarly named source records; they are not asserted experiment or acquisition times.
S2 and S11 digests identify authorized raw-file exports; S3 identifies Project-extracted text only. Full source recovery does not imply that an original private report is publicly distributed. S4/S5/S7–S9 digests identify excerpt or inventory records. Earlier S2 excerpt identity `2e0a8c1986d1aa4959c953aba89224f5997e35cd40c096fc58395e1aa6378e88` (5850 bytes) and combined AML excerpt identity `d527042e21d431f91fffeece6375e45a9740682a4121d32fb52f71c224afde65` (11859 bytes) remain historical records, not alternative hashes of the new full sources.

## Baseline access

The baseline snapshot is identified by commit `da1470aec6631c66b0a06b3ee26e740ec18477c2` and tree `265150dcaca51a957e3ebb0bb1b5951b04f23903`; the public reference is the [baseline repository tree](https://github.com/YMGPwcca/Mechrevo-Xingyao-14/tree/da1470aec6631c66b0a06b3ee26e740ec18477c2). The baseline technical pages remain the source for retained historical observations. Editorial consolidation does not constitute a hardware rerun.

## Artifact inspection record

`SRC-BINARIES` identifies `inventory/mounted_artifacts.json` (export SHA-256 `06d258dc27015a0ab260f04326fd373a8c29867d7d567276cbfe401f8e9113ab`, 87,468 bytes): seven uploaded vendor packages were sized and hashed, with selected ZIP members measured. `SRC-SFX` identifies `inventory/sfx_inspection.json` (export SHA-256 `13362ac89bd608372c1377cca55e5f3a0746fa0c46aa2a61ad20b6af1cac7ca4`, 4,511 bytes): the embedded 7-Zip signature, member listing/digests, earlier EC carve and H2OFFT numeric version were inspected without invoking vendor executables.

The measured updater EC extraction at `isflash.bin+0x268E30`, length `0x18000`, matches the retained historical digest. This establishes extraction identity; it does not establish equality with the separate `0x081000/0x20000` raw-ROM carve. No raw-ROM binary was available for a new full-dump hash or DXE comparison.

## External interface and licensing references

These official references were checked for the bounded statements in the table below. They define generic interface or licensing semantics; they do not establish support for a particular device, vendor package or firmware operation.

| ID | Primary reference | Intended bounded use | Current status |
|---|---|---|---|
| E1 | [Linux power-supply class](https://raw.githubusercontent.com/torvalds/linux/master/Documentation/power/power_supply_class.rst); [power-supply sysfs ABI](https://www.kernel.org/doc/Documentation/ABI/testing/sysfs-class-power) | Official references define capacity as a percentage and voltage/charge/energy/power units including µV, µAh, µWh and µW. The S1 baseline telemetry remains the source for this machine's values and attribute availability. | Bounded external semantics confirmed; not machine-support evidence |
| E2 | [CC BY 4.0 legal code](https://creativecommons.org/licenses/by/4.0/legalcode.en); [license deed](https://creativecommons.org/licenses/by/4.0/deed.en); [Creative Commons FAQ](https://creativecommons.org/faq/) | Legal code sections 1–8 define the CC BY 4.0 grant and conditions; the deed summarizes attribution/share/adapt/no-endorsement points; the FAQ distinguishes rights held by the licensor from third-party material. The legal code controls. | Bounded license scope confirmed; applies to original documentation only |

Vendor download URLs are not required provenance fields. File identity, parent relationships, source classification and availability are recorded independently of distribution links.

## Evidence recovery

The [documentation status register](documentation-status.md) tracks source-export, capture-recovery, identity, permission and publication gates. P01 is source-imported; P02a recovers complete THMM while P02b retains ACPI table identity. P15a recovers device enumeration while P15b/P15c retain exact environment and standard-profile gaps. P09, P10, P11, P16 and P17 remain evidence/identity gates. P18 is closed only for the bounded E1/E2 statements. Static source recovery does not validate a setter or turn a planning lead into measured data.

## Source coverage vocabulary

- **Full bytes/capture included:** the identified source bytes or complete capture are present in the public evidence set.
- **Selected excerpt included:** only the cited portions are available; claims are limited to those portions.
- **Retained report only:** the source report is preserved or summarized, but raw input/output is not available for independent inspection.
- **Source identified but not exported:** a concrete source locator exists and must be retrieved before full import.
- **Unverified lead:** a planning value or incomplete locator is retained only to guide future evidence recovery.

A historical live finding may remain `Live-confirmed` when its retained report supports the behavior; its source coverage remains separate and does not imply that this revision replayed it. Conversely, static help text does not establish that a command ran successfully.
