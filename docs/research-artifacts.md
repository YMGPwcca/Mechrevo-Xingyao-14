# Research artifacts and provenance

This repository intentionally stores **documentation only**. The binary artifacts listed here were used during reverse engineering but are not committed.

## Current full ROM

```text
filename: P916F-STX-current-ROM.bin
size:     32 MiB
SHA-256:  77043505b6f42e4a482110a7ba0c7e12ba6b1db28fdaed2743c28578bbf76cd7
```

## Carved IT5571 EC firmware

```text
filename:  P916F-IT5571-EC-1.09.bin
ROM offset: 0x081000
length:     0x20000 bytes
SHA-256:    42c117f00c130c5e533be93ee1657401ac4d687255ed1b2250f74d3cc79397ea
```

## MECHREVO Control Center GX package

```text
filename: ControlCenter_5.56.1.13_Mechrevo_GX.zip
SHA-256: d081d2b338068ca6fd1099be2f6762d522c1223796f20a800d47034842423449
```

## Native ACPIDriverDll.dll

```text
SHA-256: 97d7115943600c2a09951440859f9bd75fd0d8bff9db49c296c868b49df8c8c6
```

## GCUService.exe

```text
SHA-256: 01225ef470420d50e51bc541d63dd5ed321835d40c4209106da9908c8f277a9c
```

## BIOS updater package

A vendor BIOS 1.15 package named:

```text
STX_SKU2_1.15.zip
```

was inspected. Important contents include Insyde H2O flash tooling (`H2OFFT-Wx64.exe`, `isflash.bin`) and the platform configuration used by the updater.

## Boot animation

The BIOS 1.15 image contains an OEM animation associated with GUID:

```text
931f77d1-10fe-48bf-ab72-773d389e3faa
```

Observed properties:

```text
format:    GIF
size:      800x600
frames:    60
duration:  ~1.74 s
consumer:  OemBadgingSupportDxe
```

## WMI Binary MOF

An extracted DSDT BMOF buffer was saved during research as `WQBA.bmof`.

Observed metadata:

```text
size:         1092 bytes (0x444)
first bytes:  46 4F 4D 42 01 00 00 00 34 04 00 00 5C 10 00 00
BMOF GUID:    05901221-D566-11D1-B2F0-00A0C9062910
```

## Why hashes are recorded

The machine has been investigated using multiple vendor packages, sibling-platform files and generic MECHREVO/Uniwill components. Exact hashes prevent conclusions from one binary being silently attributed to another revision.

When adding future findings, record at minimum:

- exact product/board name,
- BIOS/EC version,
- file name,
- SHA-256,
- whether the finding is static-only or live-confirmed.
