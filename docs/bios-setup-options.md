# BIOS setup options and reachability

## Scope and source boundary

This is a partial reference for the P916F-STX BIOS 1.15 HII audit, `P916F-STX_BIOS_1.15_full_option_audit.md` (File Library ID `file_00000000179c8206a3dc01afcd82478c`, S2 in the [source register](research-sources.md#project-sources)). Only selected excerpts are available; the full audit remains identified but not exported.

The source reports **204 named non-reference AMD PBS controls** and **416 named non-reference AMD CBS controls**. Those are source-reported section totals, not importer counts and not the number of rows reproduced here. This page therefore does not claim a 620-control import.

### Counts in this page

The recovered excerpt contains:

- **8 option rows** in the selected source-row table;
- **10 reachability rows** in the selected reachability table;
- **8 formset rows** in the source formset table.

These are separate row categories and are not added to form a control total. The remaining option, reachability and reference/action rows require the exact full source export. Pending gate **P01** remains `NEEDS_SOURCE_EXPORT`; see [documentation status](documentation-status.md#pending-evidence).

## Interpretation rules

The source legend uses these static categories:

| Source label | Meaning in this reference |
|---|---|
| `VISIBLE/UNSUPPRESSED` | Static IFR does not hide the item. A top-level tab can still depend on Insyde FrontPage/FormBrowser policy. |
| `HARD-HIDDEN` | A constant `SuppressIf TRUE`, or a parent menu reference hidden by that condition, statically hides the item. |
| `CONDITIONAL` | Visibility depends on another setting or hardware condition. |
| `ORPHAN/DYNAMIC` | A form exists, but no static root-form reference path was found; runtime injection remains possible. |
| `AMD PBS/CBS external formset` | A real HII formset that is not statically linked into the OEM SetupUtility tree. It must not be described as an ordinary hidden child of Advanced. |

Static visibility is not a live screenshot result. An IFR default is not a current variable value, and an option's presence does not confirm the associated hardware or behavior. The `Risk` column reproduces the source analyst's annotation; it is not a measured safety or reliability result.

## SetupUtility formsets

| Formset | GUID | Source note |
|---|---|---|
| Power | `A6712873-925F-46C6-90B4-A40F86A0917B` | Insyde SetupUtility formset |
| Advanced | `C6D4769E-7F48-4D2A-98E9-87ADCCF35CCC` | Insyde SetupUtility formset |
| Main | `C1E0B01A-607E-4B75-B8BB-0631ECFAACF2` | Insyde SetupUtility formset |
| Boot | `2D068309-12AC-45AB-9600-9187513CCDD8` | Insyde SetupUtility formset |
| Security | `5204F764-DF25-48A2-B337-9EC122B85E0D` | Insyde SetupUtility formset |
| Exit | `B6936426-FB04-4A7B-AA51-FD49397CDC01` | Insyde SetupUtility formset |
| AMD PBS | `B863B959-0EC6-4033-99C1-8FD89F040222` | Separate `AmdPbsSetupDxe` HII formset |
| AMD CBS | `B04535E3-3004-4946-9EB7-149428983053` | Separate `CbsSetupDxeSTX` HII formset |

The exact SetupUtility FFS GUID is `FE3542FE-C1D3-4EF8-657C-8048606FF670`. The Boot formset is listed above; it is not evidence that its controls are currently visible in the stock UI.

## Recovered option rows

The following table preserves every option row present in the selected excerpt. Choice spelling, source values, duplicate values and default markings are retained.

| Form | Option | Type | Internal suppression | Var/offset | Choices/default | Risk (source annotation) |
|---|---|---|---|---|---|---|
| AMD PBS Option | Dynamic LID | OneOf | `UNSUPPRESSED` inside formset | `AMD_PBS_SETUP+0xDF` | `Disabled` = `0` (default)<br>`Enabled ` = `1` | Medium/High |
| AMD PBS Option | Dynamic P3T limit | OneOf | `UNSUPPRESSED` inside formset | `AMD_PBS_SETUP+0xA9` | `Disabled` = `0`<br>`Enable for DC-only case (include fake DC)` = `1` (default)<br>`Enabled ` = `2` | Medium/High |
| AMD PBS Option | APIC Software Enable | OneOf | `UNSUPPRESSED` inside formset | `AMD_PBS_SETUP+0x85` | `Disabled` = `0`<br>`Enabled` = `1` (default) | High |
| AMD PBS Option | ACPI Power Button Method Support | OneOf | `UNSUPPRESSED` inside formset | `AMD_PBS_SETUP+0xED` | `Generic Button Device` = `0`<br>`ACPI Control Method` = `1` (default) | Medium/High |
| AMD PBS Option | Power Button Override | OneOf | `CONDITIONAL` inside formset | `AMD_PBS_SETUP+0x101` | `4 Seconds` = `0` (default)<br>`10 Seconds` = `1` | Medium/High |
| CPU Common Options | Core Performance Boost | OneOf | `UNSUPPRESSED` inside formset | `AmdSetup+0x24` | `Disabled` = `0`<br>`Auto` = `1` (default) | Medium/High |
| CPU Common Options | Global C-state Control | OneOf | `UNSUPPRESSED` inside formset | `AmdSetup+0x25` | `Disabled` = `0`<br>`Enabled` = `1`<br>`Auto` = `3` (default) | Medium/High |
| AIM-T Options | Wireless KVM Mouse Protocol | OneOf | `CONDITIONAL` inside formset | `AmdSetup+0x26D` | `Absolute` = `0`<br>`Simple` = `1`<br>`Auto` = `0` (default) | Medium/High |

The repeated value `0` for `Absolute` and `Auto` is retained as a source anomaly. It is not corrected to an assumed distinct enum value. No live values are supplied in this table: the only retained setup overlay in the baseline is Quiet Boot at `SystemConfig+0x6E` (see below and [firmware structure](firmware-bios.md#quiet-boot)).

## Recovered reachability rows

These rows are the complete reachability subset present in the excerpt, not a complete map of every form or child reference.

| Tab | Form | Static status |
|---|---|---|
| Advanced | Advanced | `VISIBLE` |
| Advanced | PCI Express Configurations | `HARD-HIDDEN-PARENT` |
| Advanced | Boot Configuration | `HARD-HIDDEN-PARENT` |
| Advanced | Peripheral Configuration | `HARD-HIDDEN-PARENT` |
| Advanced | SATA Configuration | `HARD-HIDDEN-PARENT` |
| Advanced | Video Configuration | `HARD-HIDDEN-PARENT` |
| Advanced | USB Configuration | `VISIBLE` |
| Advanced | Chipset Configuration | `HARD-HIDDEN-PARENT` |
| Advanced | ACPI Table/Features Control | `HARD-HIDDEN-PARENT` |
| Advanced | CPU Related setting | `HARD-HIDDEN-PARENT` |

The source also records a bounded negative search: `SetupUtility` contains `User Access Level`, but searches of `SetupUtility` and `H2OFormBrowserDxe` found no `Setup Menu Insyde Full Show`, `Hide Item Control`, `Developer Mode`, `Advanced Mode`, `Full Show`, or equivalent global show-all variable. This does not prove that every possible visibility mechanism is absent, and generic keyboard shortcuts are not verified for this build.

## Defaults, live values and behavior are separate

### IFR/source defaults

The selected excerpt lists `Dynamic LID` with `Disabled = 0` marked as the source default and `Enabled = 1` as the alternative. This IFR default is static metadata, not a runtime read, and does not establish that opening the lid powers on the machine. ACPI lid-state methods and the setup option are separate layers.

### Retained runtime read

The baseline recorded one live value:

```text
QuestionId:      0x1064
VarStore:        SystemConfig
VarStore GUID:   A04A27F4-DF00-4D42-B552-39511302113D
VarStore offset: 0x6E
0x00:            Disabled
0x01:            Enabled
Runtime read:    Setup[0x6E] = 0x01
```

This confirms the reported Quiet Boot value at that observation only. It does not convert any IFR default into a live value, and it does not establish the result of saving a changed setting. Proposed live values for Dynamic LID, AC Loss, Auto Wake S5, Charger BYPASS and other settings remain unconfirmed until an identified variable capture is recovered (pending **P12**).

### Behavioral evidence

A setting's option/default and a runtime read do not establish what the firmware does after a change. The retained SREP record concerns runtime form visibility; it is not a behavioral test of each control. Any future overlay must identify the variable, GUID/VarStore, offset, width, initial value, operation and observed result separately.
## Evidence coverage

| Claim or record | Evidence class | Source coverage | Source |
|---|---|---|---|
| Eight formset records and eight selected option rows | Static-confirmed | Selected excerpt included; full audit identified but not exported | S2 · [project source register](research-sources.md#project-sources) |
| Ten selected reachability rows and bounded search result | Static-confirmed | Selected excerpt included; remaining rows not exported | S2 |
| Source-reported PBS 204 and CBS 416 totals | Static-confirmed | Selected section headers only; this is a report statement, not an independent recount | S2 |
| Quiet Boot `Setup[0x6E] = 0x01` | Live-confirmed | Retained baseline report; original variable export not included | S1 |
| Dynamic LID choices `Disabled = 0` / `Enabled = 1`, with `Disabled = 0` marked default | Static-confirmed | Selected option row included; default is an IFR/source value, not a live read | S2 |

The evidence class describes what the cited record supports; source coverage separately describes how much of that record is available in this repository. The full option inventory remains gated by P01.


## Completion gate

Coverage is partial: the excerpted formsets, option rows and reachability rows are mapped above, while the remaining audit sections and rows await the exact full-source export. **P01 = NEEDS_SOURCE_EXPORT**; source-reported PBS/CBS totals must remain separate from importer-counted rows.
