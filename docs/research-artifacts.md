# Research artifacts and provenance

This repository intentionally stores **documentation only**. Binary artifacts used during reverse engineering are listed here for provenance but are not committed.

Exact hashes matter because the investigation touched several package revisions, generic OEM components and more than one representation of the EC firmware.

## Current raw 32 MiB ROM

```text
filename: P916F-STX-current-ROM.bin
size:     0x2000000 bytes / 32 MiB
SHA-256:  77043505b6f42e4a482110a7ba0c7e12ba6b1db28fdaed2743c28578bbf76cd7
```

This was a dump of the current machine and is the preferred source for machine-specific static claims.

The later EC carve at ROM offset `0x081000` came from this file.

## Final current-ROM IT5571 carve

```text
filename:   P916F-IT5571-EC-1.09.bin
source:     P916F-STX-current-ROM.bin
ROM offset: 0x081000
length:     0x20000 bytes / 128 KiB
SHA-256:    42c117f00c130c5e533be93ee1657401ac4d687255ed1b2250f74d3cc79397ea
```

This full 128 KiB carve is the EC image used for the detailed battery-limit reverse engineering documented in this repository.

The first 64 KiB bank is dense; the second bank is sparse but contains real data/code and should not be mistaken for a separate unrelated EC image.

## Earlier BIOS-package EC extraction

Before the raw current-ROM dump was available, an EC-like blob was extracted from the BIOS 1.15 update package's embedded `isflash.bin`:

```text
source:     BIOS 1.15 package / embedded isflash.bin
offset:     0x268E30
length:     0x18000 bytes / 98,304 bytes
SHA-256:    030ec5da8b5f027d2461af98b92416eab4a526734bee4b3032e5d9042d016023
```

It contained strings including:

```text
ITE EC-V14.6
IT557x V1.09 E00 - 20230831
AMD Motherboard
MECHREVO
VER:01.0F.00
```

This artifact was useful early in the investigation, but its package-layout offset/length should **not** be confused with the later raw-ROM carve at `0x081000/0x20000`.

The raw-ROM carve is the preferred reference for exact code addresses in the current documentation.

## MECHREVO Control Center GX package

```text
filename: ControlCenter_5.56.1.13_Mechrevo_GX.zip
SHA-256: d081d2b338068ca6fd1099be2f6762d522c1223796f20a800d47034842423449
```

Useful backend files included:

```text
AiStoneService/GCUBridge.exe
AiStoneService/MyControlCenter/ACPIDriverDll.dll
AiStoneService/MyControlCenter/GCUService.exe
AiStoneService/MyControlCenter/GCUServicePlugin.dll
AiStoneService/MyControlCenter/GCUUtil.exe
```

## Compact charge-reverse bundle

A compact bundle used to isolate native/service components was named:

```text
P916F-charge-reverse.tar.gz
```

Extracted material included:

```text
ACPIDriverDll.dll
GCUService.exe
service.ini
AirplaneDriver-related files
```

No `ACPIDriver.sys` was present in that compact extraction.

A bundle-level SHA-256 was not retained in the current documentation, so the individual binary hashes below are the stronger provenance anchors.

## Native ACPIDriverDll.dll

```text
SHA-256: 97d7115943600c2a09951440859f9bd75fd0d8bff9db49c296c868b49df8c8c6
```

Observed characteristics:

```text
Windows device path: \\.\ACPIDriver
ReadEC IOCTL:         0x9C40A488
WriteEC IOCTL:        0x9C40A48C
```

## GCUService.exe

```text
SHA-256: 01225ef470420d50e51bc541d63dd5ed321835d40c4209106da9908c8f277a9c
```

This exact binary contained battery-protection-related type and enum names but important method bodies were not directly recoverable as normal unobfuscated logic.

## BIOS updater package

The vendor BIOS 1.15 package was referred to during research as:

```text
STX_SKU2_1.15.zip
```

and contained Insyde H2O flash tooling such as:

```text
H2OFFT-Wx64.exe
isflash.bin
platform.ini
```

The raw 32 MiB current-ROM dump and the update-package layout are distinct artifacts; offsets must never be transferred between them without establishing the mapping.

## Boot animation resource

BIOS 1.15 contains an OEM animation with:

```text
GUID:       931F77D1-10FE-48BF-AB72-773D389E3FAA
format:     GIF
dimensions: 800 × 600
frames:     60
duration:   ~1.74 s
association: OemBadgingSupportDxe
```

A related boot-graphics module identified during analysis is:

```text
BootGraphicsResourceTableDxe
GUID: B8E62775-BB0A-43F0-A843-5BE8B14F8CCD
```

## SetupUtility identifiers

```text
SetupUtility FFS GUID:
FE3542FE-C1D3-4EF8-657C-8048606FF670

Boot formset GUID:
2D068309-12AC-45AB-9600-9187513CCDD8

SystemConfig VarStore GUID used by Quiet Boot:
A04A27F4-DF00-4D42-B552-39511302113D
```

## WMI Binary MOF

An extracted DSDT BMOF buffer was saved during research as `WQBA.bmof`.

Observed metadata:

```text
size:         1092 bytes / 0x444
first bytes:  46 4F 4D 42 01 00 00 00 34 04 00 00 5C 10 00 00
ASCII prefix: FOMB
BMOF GUID:    05901221-D566-11D1-B2F0-00A0C9062910
```

## Provenance rules for future findings

When adding a new low-level claim, record at minimum:

- exact product/board name;
- BIOS/EC version shown by the machine;
- source file name;
- source SHA-256 when available;
- whether offsets refer to update-package layout or raw flash layout;
- whether the result is Live-confirmed, Static-confirmed, Inferred, Comparative only, or Rejected/Superseded.

This prevents an address or conclusion from one firmware container or machine family from silently becoming a false "P916F-STX fact."
