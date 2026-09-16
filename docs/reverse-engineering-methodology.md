# Reverse-engineering methodology

This document describes how the P916F-STX findings in this repository were established. It is intentionally detailed so another engineer can reproduce the reasoning, distinguish direct evidence from interpretation, and continue the investigation without restarting from generic assumptions.

## 1. Evidence hierarchy

The investigation uses the following order of trust:

1. **Live behavior on the exact P916F-STX machine**
2. **The machine's own raw 32 MiB ROM dump**
3. **The exact ACPI tables exported by that machine**
4. **The vendor BIOS 1.15 updater for P916F-STX**
5. **Exact MECHREVO software packages inspected during the investigation**
6. **Generic Insyde / ITE / Uniwill / Tongfang material used only for architectural context**

A generic OEM register or method is never promoted to a P916F fact without either machine-specific static evidence or a live cross-check.

This rule is the reason offsets such as `0x07B9/0x07D0` remain comparative evidence while the `0x0D13/0x0D14` subsystem is treated as machine-specific.

## 2. Artifact provenance

The primary raw firmware image is:

```text
P916F-STX-current-ROM.bin
size:    0x2000000 bytes / 32 MiB
SHA-256: 77043505b6f42e4a482110a7ba0c7e12ba6b1db28fdaed2743c28578bbf76cd7
```

The preferred embedded-controller carve is:

```text
P916F-IT5571-EC-1.09.bin
source offset: 0x081000
length:        0x20000 bytes
SHA-256:       42c117f00c130c5e533be93ee1657401ac4d687255ed1b2250f74d3cc79397ea
```

The BIOS 1.15 vendor package has a nested structure:

```text
STX_SKU2_1.15.zip
  -> STX_SKU2_1.15.exe   (7-Zip self-extracting archive)
      -> isflash.bin
      -> H2OFFT-Wx64.exe
      -> platform.ini
      -> H2OFFT64.sys
      -> BiosImageProcx64.dll
      -> ...
```

Offsets quoted inside `isflash.bin` and offsets quoted inside the raw 32 MiB flash dump are therefore separate address spaces and are never mixed without an explicit mapping.

See [`research-artifacts.md`](research-artifacts.md) for exact hashes and artifact relationships.

## 3. Firmware-image analysis

### 3.1 UEFI image structure

The BIOS image was inspected at firmware-volume / FFS-module level to identify components relevant to:

- SetupUtility and IFR forms;
- OEM badging / boot graphics;
- BGRT support;
- Insyde H2OFFT/IHISI services;
- chipset SMM callbacks;
- firmware data-map regions;
- the embedded ITE EC image.

Exact GUIDs are retained in the detailed documents rather than replaced with descriptive names alone.

Examples include:

```text
SetupUtility:
FE3542FE-C1D3-4EF8-657C-8048606FF670

Boot formset:
2D068309-12AC-45AB-9600-9187513CCDD8

Boot animation resource:
931F77D1-10FE-48BF-AB72-773D389E3FAA

BootGraphicsResourceTableDxe:
B8E62775-BB0A-43F0-A843-5BE8B14F8CCD
```

### 3.2 IFR / setup analysis

Setup questions were recovered from the exact SetupUtility image rather than inferred from screenshots.

For example, Quiet Boot was identified as:

```text
QuestionId:      0x1064
VarStore:        SystemConfig
VarStore GUID:   A04A27F4-DF00-4D42-B552-39511302113D
VarStore offset: 0x6E
0x00: Disabled
0x01: Enabled
```

The form containing that setting was statically found under a suppression condition. Separately, SREP was used live to expose the hidden form at runtime. These two facts are documented separately because static firmware structure and runtime behavior are different kinds of evidence.

## 4. ACPI analysis

The machine's exact DSDT was inspected for:

- EC device definition;
- `OperationRegion` declarations;
- battery object methods;
- WMI methods and BMOF buffers;
- lid methods;
- keyboard/Fn/Windows/Copilot state fields;
- EC query events;
- OEM control methods.

One critical DSDT declaration is:

```text
OperationRegion (ERAM, SystemMemory, 0xFEEC2300, 0x100)
```

That observation alone shows only that ACPI accesses a 256-byte system-memory region. The EC-side meaning was established separately from the EC firmware's own H2RAM initialization code.

This cross-layer approach prevents assigning semantics to ACPI addresses based only on field names.

## 5. EC firmware reverse engineering

The carved IT557x image is MCS-51 / 8051-family firmware.

A lightweight 8051 disassembly workflow was used to follow:

- reset/vector flow;
- XRAM accesses through `MOVX`;
- constant DPTR loads;
- command dispatchers;
- battery-state reads;
- charger-control working values;
- setters/getters for threshold fields;
- PMC data-in/data-out paths.

Particular attention was paid to **cross-references to XRAM addresses**, because those provide more reliable semantic anchors than isolated strings.

### 5.1 H2RAM mapping

The DSDT exposes host physical memory at `0xFEEC2300`.

The exact EC image contains an initialization sequence around `CODE:0xDD50` that programs the H2RAM base/size. The resulting mapping is:

```text
host physical 0xFEEC2300 + N
        <->
EC XRAM       0x0300 + N
```

for the 256-byte exposed window.

The battery SOC field used by charge logic is at:

```text
EC XRAM 0x0394
```

which maps to:

```text
host 0xFEEC2394
```

Live MMIO values were then used as a cross-check against the static mapping.

### 5.2 Charge-limit subsystem

Static analysis identified:

```text
XRAM[0x0D01].bit4  enable/state
XRAM[0x0D13]       threshold #1
XRAM[0x0D14]       threshold #2
XRAM[0x0394]       SOC used by control logic
```

Relevant code landmarks include:

```text
0xED7A  set enable bit
0xED8E  test enable bit
0xEDBA  validate/write 0x0D13
0xEDDF  validate/write 0x0D14
0xF508  reset/disable path
0xF526  read 0x0D13
0xF621  read 0x0D14
0xC063  main SOC/threshold decision path
```

The range-check code was interpreted instruction-by-instruction rather than merely inferred from later behavior. The use of `SETB C` before `SUBB A,#0x64` is what establishes `100` as still valid and `101` as invalid.

## 6. Live I/O discovery

### 6.1 Super-I/O configuration

ITE configuration space was checked at the common candidate ports.

The machine returned no valid chip identity at `0x2E`, while `0x4E` exposed:

```text
chip     = 0x5571
revision = 0x07
```

Logical device `0x12` showed:

```text
active = 0x01
I/O #0 = 0x0068
I/O #1 = 0x006C
I/O #2 = 0x0000
IRQ    = 0x00
```

This established the actual live PMC2 transport independently of generic ITE documentation.

### 6.2 PMC transaction model

The successful battery commands use:

```text
DATA            = 0x68
COMMAND/STATUS  = 0x6C
```

with the standard status semantics exercised in testing:

```text
OBF = bit0
IBF = bit1
```

A transaction waits for `IBF=0`, sends the command to `0x6C`, waits again, writes the following data/subcommand byte to `0x68`, and—when a reply is expected—waits for `OBF=1` before reading `0x68`.

The host protocol was not guessed by brute force. It was recovered statically first and then exercised with the smallest read-only / state-preserving tests possible.

## 7. Progressive validation strategy

The charge-limit feature was validated in increasing-risk order:

1. **Read state only**
   - `F1 12`
   - `F1 13`
   - `F1 14`
2. **Write thresholds while the subsystem remained disabled**
   - `F2 80`
   - `F3 100`
3. **Read back thresholds**
4. **Enable only after readback matched**
   - `F1 11`
5. **Observe charging behavior above the configured threshold**
6. **Discharge below the boundary and observe charging resume**
7. **Reboot and verify persistence**
8. **Apply CPU load and observe battery-energy behavior**

This sequence was chosen to separate protocol correctness from charger behavior and reduce the chance of an unknown write producing immediate power-state changes.

The raw results are retained in [`validation.md`](validation.md).

## 8. Rejected hypotheses are part of the technical record

A failed hypothesis is kept when the failure rules out a plausible implementation path.

### 8.1 Dedicated I2EC base `0x380`

A candidate I2EC interpretation was tested read-only. Candidate reads returned `0xFF` for all targets while the known H2RAM/MMIO path returned a real SOC value.

That result rejects the proposed stock `0x380` access path.

### 8.2 Huawei threshold API

The Huawei-compatible WMI path exists, but the expected battery-threshold GET returned unsupported/failure. The corresponding SET was not attempted.

This prevents the mere presence of `huawei-wmi` from being mistaken for the actual charge-limit implementation.

### 8.3 Generic Uniwill offsets

MECHREVO software contains familiar generic battery offsets such as:

```text
0x07B9
0x07D0
```

The exact P916F firmware instead exposes the `0x0D13/0x0D14` subsystem through PMC2. Generic offsets are therefore retained as comparative OEM-software evidence, not as P916F register definitions.

### 8.4 Insyde logo-update routes

Two standard Insyde mechanisms were traced into the exact P916F firmware:

- Type `0x54` / `-edt4f`
- Type `0x6D` / `-logoupdate`

The first is rejected by the exact chipset callback; the second lacks the expected provisioned target region. This is technically useful negative evidence and is retained in detail in [`boot-logo-research.md`](boot-logo-research.md).

## 9. Static fact versus behavioral semantics

The repository intentionally distinguishes cases such as:

```text
static fact:
  0x0D14 is a validated 0..100 field used by the charge-control routine

not yet proven:
  the exact end-user meaning of 0x0D14
```

Likewise:

```text
static fact:
  both threshold setters accept 0..100

live fact:
  T1=80 / T2=100 produces an approximately 80% cap

not yet proven:
  every T1=N / T2=100 pair produces an N% cap
```

This separation is essential for publishing reverse-engineering results responsibly: implementation facts should remain useful even when user-facing semantics are incomplete.

## 10. Safety principles used during the investigation

The investigation followed several practical constraints:

- no brute-force EC command scanning;
- no arbitrary writes to unknown H2RAM/MMIO bytes;
- no enabling generic EC write support merely to experiment with offsets;
- no Huawei threshold SET after the corresponding GET proved unsupported;
- no direct adoption of sibling-model EC offsets;
- write tests only after static analysis established a specific command and expected state transition.

These constraints are part of the methodology because they explain why some areas remain unresolved even though the firmware contains additional unexplored command space.
