# ACPI and WMI findings

## ACPI EC region

The DSDT exposes the embedded-controller shared-memory window as:

```text
OperationRegion (ERAM, SystemMemory, 0xFEEC2300, 0x100)
```

This is a **SystemMemory** region backed by the EC's H2RAM mapping, not the classic byte-oriented ACPI EC I/O mechanism itself.

Static EC firmware analysis establishes the corresponding EC-side window as:

```text
host physical 0xFEEC2300 + N  <->  EC XRAM 0x0300 + N
```

for `N = 0x00..0xFF`.

The EC-side configuration sequence is documented in [`embedded-controller.md`](embedded-controller.md).

## ACPI EC device

The ACPI EC device uses the normal embedded-controller identity:

```text
_HID = PNP0C09
GPE  = 0x0B
```

The machine also has the conventional ACPI EC host ports around `0x62/0x66` for standard ACPI EC transactions.

That path is separate from the additional ITE **PMC2** logical device later found live at `0x68/0x6C`.

## Lid device

ACPI exposes a standard lid object:

```text
_HID = PNP0C0D
```

Its `_LID` method reads the EC-backed `LIDS` bit. In the decompiled AML:

```text
LIDS == 0 -> _LID returns 0
otherwise -> _LID returns 1
```

An EC query handler `_Q81` calls `Notify (LID, 0x80)` to report a status change.

This is the firmware-level lid-status path and is separate from Linux `systemd-logind` policy about what the OS should do when the lid closes.

## Battery object: LCBT

The ACPI battery object is `LCBT`, and Linux exposes it as:

```text
/sys/class/power_supply/LCBT
```

Relevant AML methods include:

```text
_BIX   extended battery information
_BST   dynamic battery status
_BTP   battery trip-point programming
```

### `_BTP` is not charge limiting

`_BTP` writes EC fields `BTPL/BTPH` at approximately ACPI-region offset `0x90`.

That mechanism is the standard ACPI battery trip point used for notification/event behavior. It is **not** the firmware charge-cap implementation at `0x0D13/0x0D14`.

This distinction is important because the presence of an ACPI `_BTP` method can otherwise look superficially like a configurable charge threshold.

## DSDT EC field map

The retained DSDT mapping includes:

```text
Offset 0x30:
  ACIN  bit6
  ACLW  bit7

Offset 0x31:
  PBST  bit0
  LIDS  bit1
  PBS2  bit6

Offset 0x32:
  TPST  bit0
  KBFN  bit1

Offset 0x33:
  KBBL  8 bits

Offset 0x34:
  WINS  bit0
  CPLS  bit1

Offset 0x35:
  KBST  16 bits

Offset 0x80:
  BATI  bit0
  BAII  bit1
  BACG  bit2
  BAIC  bit3

Battery telemetry region:
  0x81  BFCL/BFCH
  0x83  BRML/BRMH
  0x85  BDCL/BDCH
  0x87  BMNF
  0x88  BVLL/BVLH
  0x8A  BCRL/BCRH
  0x8C  BDVL/BDVH
  0x8E  BACL/BACH

Other fields:
  0x90  BTPL/BTPH
  0x92  SRNM (16 bits)
  0x94  RSOL/RSOH
  0xA0  BDVN (128-bit/model-string region)
  0xCF  STAS
  0xFE  TEMP
```

The exact symbolic names above come from the AML field declarations. Their higher-level semantics should be established from their AML consumers rather than guessed solely from abbreviations.

## OEM ACPI control methods

The DSDT contains several small host-callable methods that expose EC-backed laptop controls. These methods return a 256-byte buffer whose first byte is used as a status field.

### Fn-lock state

```text
GFLS -> reads EC field KBFN
SFLS -> accepts only 0 or 1 and writes EC field KBFN
```

`SFLS` contains explicit firmware debug strings identifying the two states as:

```text
Fn lock
Fn unlock
```

so the semantic mapping is unusually clear here.

### Windows-key lock state

```text
GWLS -> reads EC field WINS
SWLS -> accepts only 0 or 1 and writes EC field WINS
```

`SWLS` contains explicit debug strings:

```text
WMI Request Win lock
WMI Request Win unlock
```

### Copilot-key lock state

```text
GCLS -> reads EC field CPLS
SCLS -> accepts only 0 or 1 and writes EC field CPLS
```

`SCLS` explicitly logs:

```text
WMI Request Copilot lock
WMI Request Copilot unlock
```

### TPST state/control pair

The methods:

```text
GTPS -> reads EC field TPST
STPS -> accepts only 0 or 1
```

are paired with EC commands:

```text
requested state 1 -> ECMD(0x90)
requested state 0 -> ECMD(0x8F)
```

The field/method naming and event flow identify this as a two-state platform input-device control path. The exact end-user polarity/behavior should still be considered unverified until exercised live rather than inferred only from names.

### Interface version method

`GVER` returns:

```text
0x00020004
```

as its data DWORD. This appears to be the OEM ACPI/WMI interface version advertised by this firmware path; no broader semantic interpretation has been required for the battery work.

## EC query -> WMI event mapping

Several EC query handlers convert hardware-state changes into WMI notifications through `WMI1.WMEN` followed by `Notify (WMI1, 0xA0)`.

Known examples from the AML include:

```text
_Q0C:
  TPST == 0 -> WMEN = 0x30
  TPST == 1 -> WMEN = 0x31

_Q11:
  KBBL == 0 -> WMEN = 0x20
  KBBL == 1 -> WMEN = 0x21
  KBBL == 2 -> WMEN = 0x22

_Q17:
  KBFN == 0 -> WMEN = 0x50
  KBFN == 1 -> WMEN = 0x51

_Q10 -> WMEN = 0xA1
_Q12 -> WMEN = 0xA0
_Q18 -> WMEN = 0xA2
```

The first three groups can be correlated directly with named EC state fields. The exact user-facing meanings of the standalone `0xA0/0xA1/0xA2` events were not established in this investigation and are therefore left unnamed.

## Live battery model / Linux identity

The retained battery model string is:

```text
588974-3S-G-A0
```

Linux exposes status/telemetry successfully through the ACPI battery driver, including `capacity`, `status`, `voltage_now`, `power_now` and `energy_now` on the researched installation.

It does **not** expose generic charge-limit files such as:

```text
charge_control_start_threshold
charge_control_end_threshold
charge_behaviour
```

## Huawei-compatible WMI

The firmware includes Huawei-compatible WMI plumbing and a Binary MOF block.

Known BMOF GUID:

```text
05901221-D566-11D1-B2F0-00A0C9062910
```

The DSDT contains the corresponding `WQBA` buffer.

An extracted `WQBA.bmof` was observed as:

```text
size:          1092 bytes / 0x444
first bytes:   46 4F 4D 42 01 00 00 00 34 04 00 00 5C 10 00 00
ASCII prefix:  FOMB
```

Decoding showed generic OEM WMI classes; no dedicated battery-charge-rationing class was established for this machine.

## `huawei-wmi` Linux result

A `huawei-wmi` platform device was present, but the expected battery-charge control attributes were absent.

The existence of a Huawei-compatible WMI device therefore does **not** imply the standard Huawei charge-threshold API is implemented by this firmware.

## WMAA dispatch inspection

The ACPI WMI method `WMAA` was inspected statically.

The observed dispatcher recognizes a finite set of known MFID/SFID combinations in the lower command range; the investigation did not find the expected battery-threshold `0x10/0x11` family implemented as a usable path.

This static result matches the later live failure of the standard Huawei threshold GET.

## Standard Huawei threshold API test

A read-only Huawei-style battery-threshold GET using:

```text
method/function ID 0x1103
```

returned a failure/unsupported status (`byte0 = 1` in the returned result used during the investigation).

Because the corresponding GET was unsupported, the write-side method:

```text
0x1003
```

was deliberately **not** tested.

That negative result is important: the working P916F charge-limit feature should not be described as a Huawei-WMI battery threshold feature merely because Huawei-compatible WMI plumbing exists elsewhere in the firmware.

## Classic ACPI EC versus ITE PMC2

There are two different host-facing interfaces relevant to the investigation:

```text
ACPI EC:  data/status path around 0x62/0x66
ITE PMC2: data 0x68, command/status 0x6C
```

An earlier ACPI-only inspection did not reveal a useful charge-limit path and initially led to the assumption that `0x68/0x6C` was irrelevant.

Live ITE Super-I/O configuration-space probing later showed:

```text
LDN 0x12 (PMC2)
active = 1
I/O #0 = 0x0068
I/O #1 = 0x006C
```

and the battery command family then worked through those ports.

Therefore the later live PMC2 result **supersedes** the earlier ACPI-only negative assumption.

## Conclusion

The machine's battery charge cap is:

- not a generic Linux power-supply threshold interface,
- not the ACPI `_BTP` trip-point facility,
- not the tested Huawei `0x1103/0x1003` threshold API,
- not the generic Uniwill `ECRR/ECRW` path found in some Windows packages.

The proven path is the P916F's own ITE PMC2 command family documented in [`battery-charge-limit.md`](battery-charge-limit.md).
