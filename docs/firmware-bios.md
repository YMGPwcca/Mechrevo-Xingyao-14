# BIOS / firmware notes

## BIOS 1.15 package

The researched Xingyao 14 / `P916F-STX` has been tested on BIOS **1.15** with EC version **1.15**, build date **2026-05-07**.

A vendor package named `STX_SKU2_1.15.zip` was inspected. It uses the Insyde H2O flashing stack and contains at least:

- `H2OFFT-Wx64.exe`
- `isflash.bin`
- Insyde `platform.ini`

The updater configuration includes `[FlashComplete] Action=1,1`, which corresponds to shutting the machine down after a successful flash.

## Full ROM image

A 32 MiB dump of the current BIOS/flash image was used for static analysis.

Known file in the research workspace:

- `P916F-STX-current-ROM.bin`
- size: **32 MiB**
- SHA-256: `77043505b6f42e4a482110a7ba0c7e12ba6b1db28fdaed2743c28578bbf76cd7`

The image contains the embedded ITE EC firmware discussed in [`embedded-controller.md`](embedded-controller.md).

## Boot logo / animation

The BIOS contains an OEM boot animation resource rather than only a single static BMP.

Static analysis found:

- an **800×600 animated GIF**,
- **60 frames**,
- duration approximately **1.74 seconds**,
- firmware object GUID: **`931f77d1-10fe-48bf-ab72-773d389e3faa`**,
- referenced by / associated with **`OemBadgingSupportDxe`**.

After BIOS 1.15, the machine was observed to show a new BGRT/logo and pre-boot animation.

Changing the boot branding is therefore not simply a matter of replacing an ordinary filesystem image. The asset is embedded in the UEFI firmware volume and must preserve the firmware object's structure, compression/alignment/checksums and flash layout.

### Current safety position

No runtime method has been proven that changes only the logo without rewriting firmware storage. Because a failed firmware write can brick the machine, this repository does **not** treat logo replacement as a safe live customization path.

## Quiet Boot

Static analysis located a hidden Insyde setup variable for **Quiet Boot**.

Known details:

- setup/VarStore GUID: **`A04A27F4-DF00-4D42-B552-39511302113D`**
- offset: **`0x6E`**
- observed semantics:
  - `1` = enabled
  - `0` = disabled

This is a firmware setup finding, not a recommendation to edit NVRAM blindly.

## Setup utility

The firmware contains an Insyde SetupUtility module. Its FFS GUID begins with **`FE3542...`**; earlier reverse-engineering work identified it while inspecting hidden setup forms and variables.

Where exact full GUID or IFR offsets are not retained in this repository, they should be re-derived from the 1.15 image rather than guessed from another machine.

## Dynamic LID

Firmware strings/forms contain **`Dynamic LID` / `AmdDynamicLid`**. The option appeared disabled by default in static setup analysis.

ACPI separately exposes lid state through `_LID`/`LIDS`, but no confirmed firmware option for "open lid to power on" has been established from this finding alone.

## Cross-flash warning

Sibling model names such as `P916F-HPT-R` and `P916F-ARL` have appeared during research. Their firmware should not be treated as interchangeable with `P916F-STX`.

Board-name similarity is not enough evidence for a safe flash.
