# Validation report

This document records only the live tests required to establish the P916F-STX EC and battery-limit behavior.

## EC / PMC2 discovery

Live Super-I/O configuration-space probing produced:

```text
config port 0x2E -> no valid chip ID
config port 0x4E -> chip 0x5571, revision 0x07

LDN 0x12 (PMC2):
active = 0x01
I/O #0 = 0x0068
I/O #1 = 0x006C
I/O #2 = 0x0000
IRQ    = 0x00
```

Result:

```text
PMC2 DATA            = 0x68
PMC2 COMMAND/STATUS  = 0x6C
```

## Rejected dedicated-I2EC hypothesis

A read-only candidate I2EC test returned:

```text
EC[200D] = 0xFF
EC[0394] = 0xFF
EC[0D13] = 0xFF
EC[0D14] = 0xFF
```

while the known H2RAM/MMIO battery byte simultaneously returned a valid SOC value:

```text
0xFEEC2394 = 92
```

Result: the hypothesized stock dedicated-I2EC path at base `0x380` is rejected.

## Battery-limit baseline

Initial PMC2 reads returned:

```text
F1 12 -> state = 0
F1 13 -> T1    = 0
F1 14 -> T2    = 0
```

## Threshold write/readback

With the subsystem still disabled:

```text
F2 80  -> response 80
F3 100 -> response 100
```

Subsequent reads returned:

```text
state = 0
T1    = 80
T2    = 100
```

This validates the `F2` and `F3` setters and confirms that threshold writes do not implicitly enable the subsystem.

## Enable

After sending:

```text
F1 11
```

readback returned:

```text
state = 1
T1    = 80
T2    = 100
```

## Charging behavior above the configured threshold

With battery SOC well above 80% and AC connected, the machine settled at:

```text
status    = Not charging
power_now = 0
```

This validates that the enabled `80/100` configuration stops normal charging above the threshold region.

## Boundary behavior

After discharging below the configured region and reconnecting AC:

```text
displayed 77–78% -> charging observed
around displayed 79–80% -> charging stopped
```

Linux reports integer `capacity`, so this establishes the approximate transition region but not an exact hysteresis width or internal SOC rounding rule.

## Reboot persistence

After a normal reboot, without re-applying any command:

```text
state = 1
T1    = 80
T2    = 100
```

The configuration therefore persists across a normal reboot.

Persistence across complete EC power loss has not been tested.

## High-load power behavior

Before a five-minute all-CPU load:

```text
capacity   = 79%
status     = Not charging
power_now  = 0
energy_now = 63154000
ACAD       = online=1
```

After the load:

```text
capacity   = 78%
status     = Charging
power_now  = 28128000
energy_now = 62661000
```

Stored battery energy decreased by approximately:

```text
63.154 Wh - 62.661 Wh = 0.493 Wh
```

The battery therefore contributed net energy during the high-load interval before charging resumed below the threshold. The exact charger/power-path topology is not yet fully characterized.

## Validated configuration

```text
state = 1
T1    = 80
T2    = 100
```

This is the only T1/T2 pair currently behaviorally validated on the documented machine.
