# Runtime visibility of setup forms

## Scope

The firmware investigation distinguishes static form presence, runtime visibility changes and persistent firmware modification. Sources are the baseline report S1, setup audit S2 and retained candidate configuration S7 in the [project source register](research-sources.md#project-sources). This page deliberately does not publish a runnable bypass configuration.

## Static form structure

The setup audit identifies OEM SetupUtility formsets and separate AMD PBS/CBS HII formsets. Some menu references or child blocks are suppressed by constant conditions; other controls depend on settings or hardware conditions. A form without a static reference path can still be installed or reached dynamically.

These categories are not interchangeable. Exposing a top-level formset does not necessarily remove child suppression, and removing suppression can reveal controls for absent or unsupported hardware. The [option inventory](bios-setup-options.md) retains the distinct visibility classes and the source's selected row counts.

## Retained candidate configuration

The project source collection identifies `P916F_STX_115_SREP_full_reveal.cfg` (S7). Its complete displayed text is retained privately as an inert source excerpt, but its directives are not reproduced here because they would form a reusable visibility-bypass configuration. The source names H2OFormBrowserDxe visibility records for three identified formsets:

| Formset | GUID |
|---|---|
| AMD PBS | `B863B959-0EC6-4033-99C1-8FD89F040222` |
| AMD CBS | `B04535E3-3004-4946-9EB7-149428983053` |
| Power | `A6712873-925F-46C6-90B4-A40F86A0917B` |

It also names `SuppressIfPatcher` and a subsequent load/execute operation for `SetupUtilityApp`. This establishes the configuration's intended operations, not their successful execution on the investigated machine. The exact configuration is not published as a runnable known-good patch because its association with the successful session is unconfirmed.

A separately named formset-only configuration was referenced during planning, but its complete contents and result were not recovered. No failure/success chronology is invented for those candidate files.

## Recorded runtime observation

The baseline report states that SREP reported a successful search/patch and that a subsequent BIOS photograph showed the Boot page with:

```text
Quick Boot
Quiet Boot
Network Stack
PXE Boot Capability
USB Boot
UEFI OS Fast Boot
```

This remains a recorded live observation from the investigation. The original photograph, patcher binary version/digest and exact successful configuration are not yet linked in the recovered source set. The evidence therefore supports reporting that the page was revealed, but not publishing a fully reproducible configuration as validated.

The reported page reveal and the candidate configuration are separate records. The former is a retained result on the machine; the latter documents intended target formsets and operations. No claim is made that the candidate was the configuration used for the reported reveal.

## Quiet Boot value and behavior

The baseline records QuestionId `0x1064`, `SystemConfig` offset `0x6E`, Disabled=`0x00`, Enabled=`0x01`, and the runtime read `Setup[0x6E]=0x01`. These observations do not establish a completed save/change experiment or the resulting boot presentation.

A runtime form-visibility change and a subsequent setup-variable save are different operations. The form patch being applied in memory does not establish that later variable writes had no persistent effects. Conversely, the retained `Quiet Boot` read does not establish a successful change or the behavior of every boot stage.

## Permanent SetupUtility candidate

The baseline analysis records a suppression expression near **extracted SetupUtility PE file offset** `0x2636A0` and a proposed `0x46` to `0x47` byte change. This is an image-specific, extracted-PE file-offset landmark, not a raw-ROM offset or a cross-version patch specification. The modification was not flashed or live-tested and is not equivalent to the reported runtime reveal.

The candidate byte change is retained as a reverse-engineering landmark only. No patch recipe, write instruction or permanent firmware modification is supplied.

## Reproduction and pending gates

A reproducible account would require the exact patcher build, the final configuration digest, the firmware/module identity, the console result and the associated photograph. Until that set is recovered, the runtime observation and candidate file remain separate evidence records. Pending gate **P13** is `NEEDS_CONFIRMATION`: recover the successful session evidence and final configuration digest before calling any candidate known-good. The exact permanent landmark remains untested regardless of the SREP result.

Unknown hardware behavior and missing source material are tracked in [open questions](open-questions.md) and [documentation status](documentation-status.md#pending-evidence).
