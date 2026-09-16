# Linux platform integration

This document describes Linux-visible interfaces and platform behavior for the MECHREVO Xingyao 14 / `P916F-STX`. It focuses on hardware/firmware integration rather than desktop-environment configuration.

## 1. Platform identity

Reference platform:

```text
MECHREVO XINGYAO Series-P916F-STX
AMD Ryzen AI 9 365
AMD Radeon 880M
32 GiB memory on the documented unit
2880 × 1800 internal display
```

The primary environment used for the live reverse-engineering work was CachyOS / Arch-family Linux, but several observations were cross-checked outside that installation where useful.

## 2. Power-supply devices

Linux exposes the internal battery as:

```text
/sys/class/power_supply/LCBT
```

and the AC adapter as:

```text
/sys/class/power_supply/ACAD
```

### Battery model

Observed sysfs model string:

```text
/sys/class/power_supply/LCBT/model_name
588974-3S-G-A0
```

### Battery telemetry

Attributes observed and used during validation include:

```text
capacity
status
voltage_now
power_now
energy_now
model_name
```

`current_now` was not present in the observed `LCBT` device.

Representative states recorded during the charge-limit work include:

```text
89% - Discharging
```

and, with AC online near the configured cap:

```text
capacity   = 79%
status     = Not charging
power_now  = 0
energy_now = 63154000
ACAD       = online=1
```

The complete measurements are retained in [`validation.md`](validation.md).

## 3. Missing generic Linux charge-threshold ABI

The battery device did **not** expose the standard-style charge-control files commonly seen on platforms with upstream threshold support:

```text
charge_control_start_threshold
charge_control_end_threshold
charge_behaviour
```

Therefore the P916F firmware feature is not currently surfaced by the generic Linux power-supply ABI on the documented machine.

This absence is one reason the EC implementation was investigated directly.

## 4. Huawei-compatible WMI exposure

A `huawei-wmi` platform device was present, but it did not expose usable battery charge-control attributes.

A Huawei-style threshold GET using method/function `0x1103` returned unsupported/failure during direct testing. The corresponding write-side `0x1003` path was intentionally not attempted after the read path failed.

Thus:

```text
presence of huawei-wmi != working Huawei battery-threshold API
```

The proven P916F charge-limit interface is the IT5571 PMC2 path documented in [`battery-charge-limit.md`](battery-charge-limit.md).

## 5. Three distinct EC-facing mechanisms

Linux/firmware analysis revealed three different concepts that must not be conflated.

### 5.1 Standard ACPI EC interface

The ACPI embedded-controller device uses:

```text
_HID = PNP0C09
GPE  = 0x0B
```

and the conventional host EC I/O path around:

```text
0x62 / 0x66
```

This interface serves normal AML EC field accesses.

### 5.2 SystemMemory H2RAM aperture

The DSDT also declares:

```text
OperationRegion (ERAM, SystemMemory, 0xFEEC2300, 0x100)
```

Static analysis of the exact IT5571 firmware maps this host-visible 256-byte aperture to:

```text
EC XRAM 0x0300 .. 0x03FF
```

The host/EC relationship is:

```text
host 0xFEEC2300 + N  <->  EC XRAM 0x0300 + N
```

One important field is:

```text
EC XRAM 0x0394
host    0xFEEC2394
```

which is used as an SOC/battery-percentage value by the charge-control logic.

### 5.3 ITE PMC2

Live Super-I/O probing found ITE logical device `0x12` active at:

```text
DATA            = 0x68
COMMAND/STATUS  = 0x6C
```

This is the working host command transport used by the reverse-engineered charge-limit protocol.

The three mechanisms therefore have different roles:

```text
0x62/0x66             standard ACPI EC traffic
0xFEEC2300..23FF      H2RAM-backed shared-memory aperture
0x68/0x6C             ITE PMC2 command transport
```

## 6. Battery charge-limit behavior visible from Linux

With the validated EC state:

```text
state = 1
T1    = 80
T2    = 100
```

Linux showed:

- charging below the configured region;
- transition to `Not charging` around displayed 79–80%;
- `power_now=0` in stable capped samples;
- preserved EC state after a normal reboot.

During a five-minute high-CPU-load experiment with AC connected, stored battery energy still decreased by approximately `0.493 Wh`, after which the battery resumed charging at a lower displayed SOC. This demonstrates that the battery can contribute net energy under sufficient system load even while the charge cap is active.

The exact electrical power-path topology remains unresolved; Linux battery telemetry alone cannot measure wall-side adapter power.

## 7. ACPI battery methods

The battery object is `LCBT`.

Important methods include:

```text
_BIX
_BST
_BTP
```

`_BTP` writes the ACPI battery trip-point fields and is **not** the charge-limit feature.

The distinction matters because `_BTP` can superficially look like a battery threshold API while serving a different ACPI purpose.

See [`acpi-wmi.md`](acpi-wmi.md) for the full field/method mapping.

## 8. Lid interface

ACPI exposes lid state through the firmware lid object and EC-backed state. Firmware also contains the `Dynamic LID` / `AmdDynamicLid` setup item.

The setup option's exact end-user behavior remains unverified. Its presence should not be interpreted automatically as “open lid to power on.”

For OS policy, systemd-logind can independently choose whether a lid-close event suspends the system; that policy does not change the firmware lid signal itself.

## 9. BGRT / boot graphics exposure

After BIOS 1.15, Linux exposed ACPI BGRT metadata:

```text
status  = 0
type    = 0
version = 1
xoffset = 1040
yoffset = 387
```

The firmware contains an 800×600 boot-animation resource. On the documented 2880-pixel-wide panel:

```text
1040 + 800 + 1040 = 2880
```

so the BGRT horizontal placement is consistent with the extracted boot-graphics geometry.

This is corroborating evidence for the firmware resource analysis, not a logo-update mechanism by itself.

## 10. Graphics

The integrated Radeon 880M graphics stack operates through the normal AMD Linux graphics stack. Wayland operation was successful on the documented machine.

No P916F-specific graphics firmware workaround emerged from the reverse-engineering work recorded in this repository.

## 11. Audio exposure

Linux detects the internal analog path as:

```text
Realtek ALC256 Analog
```

PipeWire/WirePlumber exposes the internal speaker path as stereo:

```text
output_FL
output_FR
```

The chassis has four physical speaker drivers, but Linux does not expose a separate 4.0 or LFE endpoint. See [`audio.md`](audio.md) for the detailed distinction between physical speaker count, logical channels and OEM Windows DSP processing.

## 12. Current kernel-integration gap

The firmware charge-limit protocol is understood sufficiently to document:

```text
PMC2 ports
command framing
enable state
threshold getters/setters
validated 80/100 behavior
```

but Linux currently does not expose it through the generic power-supply threshold attributes on this machine.

A future kernel/platform integration could potentially translate the P916F EC protocol into a standard charge-control ABI, but this repository intentionally documents the protocol rather than shipping a driver implementation.
