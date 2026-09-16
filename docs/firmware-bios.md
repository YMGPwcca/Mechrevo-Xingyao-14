# BIOS / firmware notes

## Version history observed on the researched machine

The machine was previously running BIOS **1.09** and was later updated to BIOS **1.15**.

After the update, the firmware UI reported:

```text
BIOS Version: 1.15
EC Version:   1.15
Build Date:   05/07/2026
```

The build-date string is recorded exactly as shown. It is likely `MM/DD/YYYY` in the usual Insyde/SMBIOS convention, but the repository keeps the raw value to avoid turning a formatting assumption into a fact.

The update visibly changed the boot branding: the BGRT/logo changed and a pre-boot animation appeared.

A photograph of the machine's BIOS screen appears to label the processor `AMD Ryzen AI 9 HX 365`. AMD's official retail processor name is `Ryzen AI 9 365`; this repository keeps the OEM BIOS label as a separate observation rather than treating `HX 365` as the canonical AMD model name.

## BIOS 1.15 package

The outer vendor archive was:

```text
STX_SKU2_1.15.zip
```

It contained **one updater executable**:

```text
STX_SKU2_1.15.exe
```

That EXE was a 7-Zip SFX. Extracting the EXE—not merely the outer ZIP—produced the Insyde H2O payload, including:

```text
isflash.bin              35,626,768 bytes
H2OFFT-Wx64.exe
platform.ini
BiosImageProcx64.dll
H2OFFT64.sys
...
```

This nesting matters for provenance: `isflash.bin` was **not directly stored in the outer ZIP**.

The extracted updater configuration contains:

```text
[FlashComplete]
Action=1,1
```

which corresponds to a shutdown action after a successful flash.

## Current raw ROM image

A raw 32 MiB image of the current machine was dumped and used for later EC/firmware analysis:

```text
filename: P916F-STX-current-ROM.bin
size:     0x2000000 bytes / 32 MiB
SHA-256:  77043505b6f42e4a482110a7ba0c7e12ba6b1db28fdaed2743c28578bbf76cd7
```

This image is the preferred provenance for machine-specific static claims in this repository.

## Boot animation resource

BIOS 1.15 contains an OEM animated resource rather than only a static BMP.

Static analysis found:

```text
format:     animated GIF
dimensions: 800 × 600
frames:     60
duration:   ~1.74 seconds
GUID:       931F77D1-10FE-48BF-AB72-773D389E3FAA
```

The resource is associated with `OemBadgingSupportDxe`.

Another relevant boot-graphics module identified during the ROM analysis is:

```text
BootGraphicsResourceTableDxe
GUID: B8E62775-BB0A-43F0-A843-5BE8B14F8CCD
```

### Live BGRT evidence

After the BIOS update Linux exposed BGRT metadata:

```text
status  = 0
type    = 0
version = 1
xoffset = 1040
yoffset = 387
```

The internal display was observed as 2880×1800. An 800-pixel-wide image centered horizontally on a 2880-pixel-wide panel has an offset of exactly 1040 pixels, so the live BGRT placement is consistent with the 800×600 firmware resource found statically.

This geometric consistency is useful corroboration, but BGRT itself describes the boot image presented to the OS; it is not a generic runtime API for replacing the firmware resource.

## Logo-only update mechanisms

A later audit followed the two obvious generic Insyde logo-update mechanisms into the exact P916F-STX BIOS 1.15 implementation.

### H2OFFT `-edt4f` / IHISI Type 0x54

The generic Insyde route maps a logo extra-data operation to IHISI Type `0x54`.

On this machine, the relevant `ChipsetSvcSmm` callback was identified at:

```text
protocol +0xA8
RVA 0x221C
```

The exact code handles type `0x50` and returns `EFI_UNSUPPORTED` for other values. Consequently the examined P916F path does **not** implement the project-specific Type-54 raw-logo writer.

### H2OFFT `-logoupdate` / Type 0x6D

The generic authenticated logo-update mechanism expects a target identified by:

```text
DACFAB69-F977-4784-8AD8-7724A6F4B440
```

Machine-specific checks found:

```text
Windows ESRT: no DACFAB69... entry
raw-ROM HFDM at 0x1D7C000: 47 entries, no DACFAB69... entry
```

Thus the expected dedicated Type-6D logo target is not provisioned in the tested BIOS image.

### Current logo-replacement conclusion

For BIOS 1.15:

```text
Type 0x54 raw logo update  -> exact P916F callback rejects it
Type 0x6D signed logo      -> required target region not provisioned
```

No enabled/provisioned logo-only update path has therefore been established on this machine. This does not mathematically rule out every conceivable third OEM-specific mechanism, but neither of the standard Insyde paths investigated is usable as a proven low-risk logo updater.

The complete evidence chain is in [`boot-logo-research.md`](boot-logo-research.md).

## Insyde SetupUtility

The exact SetupUtility FFS GUID identified in BIOS 1.15 is:

```text
FE3542FE-C1D3-4EF8-657C-8048606FF670
```

The Boot formset GUID identified during IFR work is:

```text
2D068309-12AC-45AB-9600-9187513CCDD8
```

These exact identifiers supersede the earlier shortened note `FE3542...`.

## Quiet Boot

Static IFR/SetupUtility analysis identified the **Quiet Boot** question.

Known details:

```text
QuestionId:         0x1064
VarStore:           SystemConfig
VarStore GUID:      A04A27F4-DF00-4D42-B552-39511302113D
VarStore offset:    0x6E
0x00:               Disabled
0x01:               Enabled
```

A live runtime read on the researched machine observed:

```text
Setup[0x6E] = 0x01
```

so Quiet Boot was enabled at that point.

### Stock suppression and SREP runtime reveal

The relevant Boot settings block was located under a `SuppressIf (TRUE)` condition in the static IFR.

A static binary-analysis landmark identified the suppression expression around SetupUtility PE offset:

```text
0x2636A0
```

with a candidate byte-level change discussed during analysis (`0x46 -> 0x47`). That permanent binary firmware modification was **not live-tested** and should remain only a reverse-engineering landmark.

Separately, **Smokeless Runtime EFI Patcher (SREP)** was actually booted on the machine. Its console reported a successful search/patch result, and the subsequently visible BIOS Boot page contained options including:

```text
Quick Boot
Quiet Boot
Network Stack
PXE Boot Capability
USB Boot
UEFI OS Fast Boot
```

Therefore the repository distinguishes clearly between:

- **static permanent firmware patch candidate** — not applied/tested;
- **runtime SREP reveal of the suppressed Boot form** — observed working on the real machine.

## Dynamic LID / AmdDynamicLid

Firmware forms/strings contain:

```text
Dynamic LID
AmdDynamicLid
```

Static setup analysis associated the option with:

```text
AMD_PBS_SETUP + 0xDF
default value = 0 (disabled)
```

ACPI separately exposes lid state through `_SB.LID._LID`, `LIDS` and EC-backed lid state.

What has **not** been proven is that this setup option means "open lid to power on". The name alone is insufficient to assign that behavior.

## Cross-flash warning

Sibling model names encountered during research include:

```text
P916F-HPT-R
P916F-ARL
```

Their firmware must not be treated as interchangeable with `P916F-STX`. Similar platform naming is not evidence of matching board power sequencing, EC firmware, GPIO routing, flash layout or setup defaults.
