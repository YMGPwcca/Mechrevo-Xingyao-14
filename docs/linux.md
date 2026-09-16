# Linux platform interfaces

This page documents Linux-visible interfaces that are relevant to the P916F-STX hardware and firmware. General desktop configuration and installation history are intentionally excluded.

## Platform identity

```text
MECHREVO XINGYAO Series-P916F-STX
AMD Ryzen AI 9 365
Radeon 880M
```

## Power-supply devices

Battery:

```text
/sys/class/power_supply/LCBT
```

AC adapter:

```text
/sys/class/power_supply/ACAD
```

Battery model:

```text
588974-3S-G-A0
```

Observed battery attributes include:

```text
capacity
status
voltage_now
power_now
energy_now
```

`current_now` was not present on the documented system.

## Generic charge-control ABI

The battery device does not expose:

```text
charge_control_start_threshold
charge_control_end_threshold
charge_behaviour
```

A `huawei-wmi` platform device is present, but it does not expose a working battery charge-threshold interface for this machine.

The validated charge-limit implementation is therefore not available through the generic Linux power-supply ABI. It is implemented by the EC and reached through ITE PMC2. See [`battery-charge-limit.md`](battery-charge-limit.md).

## ACPI EC and H2RAM

The DSDT declares a 256-byte EC-backed SystemMemory region:

```text
OperationRegion (ERAM, SystemMemory, 0xFEEC2300, 0x100)
```

Static EC analysis maps this region to:

```text
host 0xFEEC2300..0xFEEC23FF
  <->
EC XRAM 0x0300..0x03FF
```

This H2RAM window is distinct from the conventional ACPI EC I/O interface and from PMC2.

## ITE PMC2

The active PMC2 interface is:

```text
DATA            = 0x68
COMMAND/STATUS  = 0x6C
```

It was confirmed through live Super-I/O configuration-space reads and subsequently used for the battery-limit command family.

## Lid reporting

ACPI exposes a standard lid device with `_HID = PNP0C0D`. `_LID` reads the EC-backed `LIDS` state, and EC query `_Q81` issues `Notify (LID, 0x80)` on lid-state changes.

OS policy for suspend-on-lid-close is separate from this firmware reporting path.

## Audio exposure

Linux detects the internal analog codec as:

```text
Realtek ALC256 Analog
```

The speaker endpoint is exposed as stereo FL/FR. No separate LFE or four-channel logical endpoint has been observed.

See [`audio.md`](audio.md).
