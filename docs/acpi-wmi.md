# ACPI and WMI findings

## ACPI EC region

The DSDT exposes the embedded-controller shared-memory window as:

```text
OperationRegion (ERAM, SystemMemory, 0xFEEC2300, 0x100)
```

This is not the classic ACPI EC I/O window itself; it is a system-memory aperture backed by the EC's H2RAM mapping.

Static EC analysis established:

```text
host 0xFEEC2300 + N  <->  EC XRAM 0x0300 + N
```

See [`embedded-controller.md`](embedded-controller.md).

## ACPI battery object

The battery device is `LCBT`.

Relevant methods/fields include:

- `_BIX` — extended static battery information.
- `_BST` — battery status/current/remain/voltage information.
- `_BTP` — ACPI battery trip-point programming.

`_BTP` writes EC fields `BTPL/BTPH` around ACPI-region offset `0x90`.

This is **not** the firmware charge-limit control. `_BTP` belongs to the ACPI battery notification/trip-point mechanism.

## Selected EC field map from DSDT

```text
0x30: ACIN bit6, ACLW bit7
0x80: BATI bit0, BAII bit1, BACG bit2, BAIC bit3
0x81+: battery telemetry fields
0x90: BTPL/BTPH
0x92: SRNM
0x94: RSOL/RSOH
0xA0: BDVN, 128-bit model-string region
0xCF: STAS
0xFE: TEMP
```

The exact semantic names come from the AML field definitions; some are used directly by `_BST`/battery logic.

## Huawei-compatible WMI

The firmware exposes Huawei-compatible WMI plumbing and a WMI Binary MOF block.

Known Binary MOF GUID:

```text
05901221-D566-11D1-B2F0-00A0C9062910
```

The DSDT contains the corresponding `WQBA` buffer.

An extracted `WQBA.bmof` was observed as:

- size: `1092` bytes (`0x444`)
- leading bytes: `46 4F 4D 42`, ASCII `FOMB`

The decoded classes were generic OEM WMI classes; no dedicated `BatteryChargeRationing`-style class was established for this machine.

## WMAA method dispatch

Inspection of the WMI dispatch path showed a finite set of recognized MFID/SFID combinations. No direct charge-limit path corresponding to the common Huawei battery-threshold IDs was established in that dispatcher.

## Standard Huawei threshold test

A read-only test of the Huawei-style battery-threshold GET path using method `0x1103` returned a failure/unsupported status.

Because GET was not supported, the corresponding SET method `0x1003` was deliberately **not** tested.

Conclusion: the P916F-STX's actual charge-limit implementation should not be accessed through that Huawei API. The working path is the EC PMC2 protocol described in [`battery-charge-limit.md`](battery-charge-limit.md).

## Classic ACPI EC ports

The machine also has the normal ACPI EC interface around ports `0x62`/`0x66` for standard ACPI EC traffic.

That interface is distinct from the later-discovered ITE **PMC2** host channel at `0x68`/`0x6C`.

An earlier research phase incorrectly concluded that the platform had no useful `0x68/0x6C` path because ACPI AML did not expose an obvious one. Live ITE Super-I/O configuration-space probing later proved PMC2 logical device `0x12` is active at exactly those ports. The newer live result supersedes the earlier negative assumption.

## Linux sysfs result

The ACPI battery driver exposes ordinary status/telemetry but not Linux charge-limit controls such as:

```text
charge_control_start_threshold
charge_control_end_threshold
charge_behaviour
```

Therefore Linux does not currently surface the EC charge-limit feature through the generic power-supply interface.
