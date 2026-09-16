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

## BIOS 1.15 package

A vendor package named `STX_SKU2_1.15.zip` / corresponding packaged updater content was inspected. It uses the Insyde H2O flash stack and contains at least:

```text
H2OFFT-Wx64.exe
isflash.bin
platform.ini
```

The updater configuration contains:

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
format:    animated GIF
dimensions: 800 × 600
frames:    60
duration:  ~1.74 seconds
GUID:      931F77D1-10FE-48BF-AB72-773D389E3FAA
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

## Logo replacement status

No safe runtime-only path has been proven for changing just the MECHREVO boot image/animation.

The branding asset lives inside UEFI firmware structures. Replacing it would require preserving the relevant FFS/FV structure, compression, alignment/checksums and flash layout. A valid-looking image file alone is not sufficient.

Because the machine is already running a known-good BIOS, this repository treats boot-logo replacement as **static reverse-engineering knowledge, not a proven safe modification procedure**.

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

Static IFR/SetupUtility analysis identified the hidden **Quiet Boot** question.

Known details:

```text
QuestionId:         0x1064
VarStore:           SystemConfig
VarStore GUID:      A04A27F4-DF00-4D42-B552-39511302113D
VarStore offset:    0x6E
0x00:               Disabled
0x01:               Enabled
```

The relevant Boot settings block was under a `SuppressIf (TRUE)` condition. A static binary-analysis note identified a candidate suppression opcode location around SetupUtility PE offset:

```text
0x2636A0
```

with a candidate byte-level change discussed during analysis (`0x46 -> 0x47`) to alter the suppression expression.

That candidate was **not live-tested** and should be treated only as a reverse-engineering landmark. The suppression block also covers more than one Boot setting, so changing it would not be a narrowly scoped "show Quiet Boot only" operation.

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
