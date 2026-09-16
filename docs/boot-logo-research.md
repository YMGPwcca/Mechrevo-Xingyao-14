# Boot-logo / OEM-badging research

This document records the detailed investigation into whether the MECHREVO boot animation/logo on `P916F-STX` BIOS 1.15 can be changed **without rebuilding and reflashing the firmware image**.

The short result is:

> **No enabled/provisioned logo-only update mechanism was found in this BIOS 1.15 build.**

Two generic Insyde mechanisms were investigated. The exact P916F firmware rejects the raw Type-54 route, while the authenticated Type-6D route lacks its required target region on this machine.

This conclusion is specific to the researched P916F-STX BIOS 1.15 image.

## Embedded OEM animation

The current BIOS contains an OEM animated GIF:

```text
format:      GIF
dimensions:  800 × 600
frames:      60
duration:    ~1.74 s
GUID:        931F77D1-10FE-48BF-AB72-773D389E3FAA
association: OemBadgingSupportDxe
```

Linux BGRT metadata after the BIOS update reported:

```text
status  = 0
type    = 0
version = 1
xoffset = 1040
yoffset = 387
```

On the observed 2880×1800 panel, `xoffset=1040` is geometrically consistent with an 800-pixel-wide image centered horizontally:

```text
1040 + 800 + 1040 = 2880
```

That is useful corroboration that the firmware-extracted 800×600 resource is closely related to the boot branding presented on the real machine.

## Insyde route #1: H2OFFT `-edt4f` / IHISI Type 0x54

Insyde H2OFFT has a generic extra-data mechanism whose documented logo-style usage takes a form such as:

```text
H2OFFT-Wx64.exe -edt4f:<image>
```

The relevant generic flow identified during the investigation is:

```text
-edt4f
  -> Extra Data Type 4
  -> IHISI OEM extra-data type 0x54
  -> LogoUpdate path
```

The fact that H2OFFT understands this option does **not** mean the target BIOS implements the corresponding OEM writer.

### Exact P916F `ChipsetSvcSmm` result

The P916F-STX BIOS 1.15 `ChipsetSvcSmm` module was inspected directly.

The callback used by the relevant IHISI extra-data service was located at:

```text
ChipsetSvcSmm
protocol +0xA8
-> RVA 0x221C
```

The decisive code is equivalent to:

```asm
221C  cmp cl, 50h
221F  je  222C
2221  mov rax, EFI_UNSUPPORTED
222B  ret
```

Semantically:

```text
if type == 0x50:
    use the implemented OA3-oriented path
else:
    return EFI_UNSUPPORTED
```

For the types relevant to this research:

```text
0x50  -> handled by this callback
0x54  -> EFI_UNSUPPORTED
0x6D  -> EFI_UNSUPPORTED in this callback
```

The generic Insyde source/reference behavior examined during the investigation is consistent with this: a project-specific OEM implementation is required for the Type-54 logo writer, while the default hook may simply return `EFI_UNSUPPORTED`.

### Type-54 conclusion

No P916F-specific raw-logo writer was found behind the examined Type-54 path.

Therefore the existence of the H2OFFT command itself must **not** be interpreted as evidence that running `-edt4f` can safely update the logo on this machine.

For BIOS 1.15 on this P916F-STX, the Type-54 route is treated as:

```text
REJECTED / NOT IMPLEMENTED FOR THIS BUILD
```

## Insyde route #2: `-logoupdate` / authenticated Type 0x6D

A second generic Insyde logo-update mechanism follows a different design:

```text
-logoupdate
  -> type 0x6D
  -> authenticated/signed image
  -> PKCS#7 verification
  -> dedicated target region identified by GUID
```

The target GUID identified for that generic logo region is:

```text
DACFAB69-F977-4784-8AD8-7724A6F4B440
```

For this mechanism to be useful, the machine needs the corresponding firmware-update/target-region plumbing to be provisioned.

### Windows ESRT result

The researched machine's Windows ESRT was inspected and did **not** expose an entry for:

```text
DACFAB69-F977-4784-8AD8-7724A6F4B440
```

This is live evidence against the machine exposing that logo component as a firmware-resource update target.

### Raw-ROM FDM result

The raw 32 MiB ROM was also scanned at the firmware device map area:

```text
HFDM / FDM raw-ROM offset: 0x1D7C000
entries observed:          47
```

None of those 47 entries used:

```text
DACFAB69-F977-4784-8AD8-7724A6F4B440
```

This is machine-specific static evidence that the expected Type-6D logo target region is not provisioned in the current raw flash image.

### Type-6D conclusion

The generic mechanism exists in the broader Insyde ecosystem, but the required target region is absent from the two machine-specific places checked:

```text
Windows ESRT  -> no DACFAB69... entry
raw-ROM FDM   -> no DACFAB69... entry among 47 entries
```

The Type-6D route is therefore treated as:

```text
NOT PROVISIONED ON THE TESTED P916F-STX BIOS 1.15
```

## Combined conclusion for logo-only update paths

The two obvious generic Insyde routes resolve as follows:

```text
P916F-STX BIOS 1.15

H2OFFT -edt4f
  -> IHISI Type 0x54
  -> raw/project-specific logo writer
  -> exact P916F callback rejects type 0x54
  -> NOT IMPLEMENTED

H2OFFT -logoupdate
  -> Type 0x6D
  -> authenticated PKCS#7 logo image
  -> requires DACFAB69-F977-4784-8AD8-7724A6F4B440 target
  -> absent from Windows ESRT
  -> absent from 47-entry raw-ROM FDM
  -> NOT PROVISIONED
```

Accordingly:

> **No safe, enabled logo-only updater has been established for this BIOS build.**

This is stronger than merely saying "we did not find the right command": both known generic Insyde paths were followed far enough into this machine's own firmware/provisioning to identify concrete blockers.

It still does **not** prove that no third, completely different OEM-specific mechanism could exist. None has been found so far.

## Why the embedded GIF is not treated as a simple replaceable file

The animation is contained inside UEFI firmware structures. Replacing the image persistently would require preserving the relevant firmware-volume/file structure and any associated compression, alignment, integrity data and flash layout.

Therefore:

```text
finding the GIF != having a safe logo replacement method
```

A firmware rebuild/reflash path remains fundamentally different from a dedicated logo-only OEM update service.

## Quiet Boot: a separate, safer lever

The investigation also identified the BIOS `Quiet Boot` setting:

```text
QuestionId:      0x1064
VarStore:        SystemConfig
VarStore GUID:   A04A27F4-DF00-4D42-B552-39511302113D
VarStore offset: 0x6E
0x00:            Disabled
0x01:            Enabled
```

A runtime read on the researched machine observed:

```text
Setup[0x6E] = 0x01
```

so Quiet Boot was enabled at that time.

### Stock IFR visibility versus runtime reveal

Static IFR analysis found the relevant Boot settings inside a suppressed block. Separately, the machine was booted with **Smokeless Runtime EFI Patcher (SREP)**; the patcher reported a successful search/patch operation, and a subsequent BIOS photograph showed the Boot page containing options including:

```text
Quick Boot
Quiet Boot
Network Stack
PXE Boot Capability
USB Boot
UEFI OS Fast Boot
```

This establishes an important distinction:

- the settings are present in the firmware forms;
- stock presentation may suppress the block;
- the block can be exposed at runtime without first rebuilding the SPI firmware image.

A candidate permanent SetupUtility suppression-byte modification discussed during static analysis is documented elsewhere, but it was **not** written to the firmware and should not be confused with the successful runtime SREP reveal.

## Safety conclusion

The logo research did **not** justify trying speculative firmware writes.

In particular:

- do not run the generic `-edt4f` command merely because H2OFFT supports the syntax;
- do not assume `-logoupdate` has a target when the expected region is absent;
- do not treat a sibling P916F firmware's logo layout as interchangeable;
- do not infer that finding an embedded GIF makes direct replacement safe.

For changing boot presentation without rewriting firmware regions, a firmware-owned setup option such as Quiet Boot is conceptually much safer than inventing an unsupported logo write path.
