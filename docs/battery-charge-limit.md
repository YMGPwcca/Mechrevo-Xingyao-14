# Battery charge limit

This is the most thoroughly live-validated low-level feature currently documented for the `P916F-STX`.

## Executive summary

The laptop has a firmware-level battery charge-limit subsystem implemented in the ITE EC.

It is reachable from Linux through **PMC2** using:

```text
DATA           = 0x68
COMMAND/STATUS = 0x6C
```

The charge-limit configuration is not exposed by ordinary Linux battery sysfs attributes on this machine, but the EC protocol works directly.

A live-tested configuration of:

```text
enabled = 1
T1      = 80
T2      = 100
```

produced the expected behavior:

- charging stopped around the configured 80% cap,
- charging resumed below the cap,
- the configuration survived a normal reboot,
- no Windows Control Center process was required for it to remain active after reboot.

## Battery device in Linux

The ACPI battery appears as:

```text
/sys/class/power_supply/LCBT
```

The AC adapter appears as:

```text
/sys/class/power_supply/ACAD
```

Example live state while capped:

```text
capacity:   79%
status:     Not charging
power_now:  0
energy_now: 63154000
ACAD: online=1
```

## EC-side state

Static reverse engineering of the exact P916F IT5571 firmware identified a dedicated charge-limit subsystem using:

| EC XRAM | Meaning |
|---|---|
| `0x0D01` bit 4 | subsystem enable flag |
| `0x0D13` | threshold value #1 |
| `0x0D14` | threshold value #2 |
| `0x0394` | live battery percentage used by the decision logic |

The firmware contains validation setters for both thresholds. Their 8051 `SUBB` logic accepts values from **0 through 100 inclusive**.

## Host protocol

The command dispatcher in the exact P916F firmware maps the following PMC2 transactions:

| Command | Data byte | Meaning |
|---|---:|---|
| `F1` | `10` | disable/reset battery-limit subsystem |
| `F1` | `11` | enable subsystem |
| `F1` | `12` | read enabled state |
| `F1` | `13` | read threshold #1 (`0x0D13`) |
| `F1` | `14` | read threshold #2 (`0x0D14`) |
| `F2` | `NN` | set threshold #1 to `NN` |
| `F3` | `NN` | set threshold #2 to `NN` |

The GET path returns its result through the PMC2 data-out register.

## Validation chronology

### 1. Stock state

The first read returned:

```text
state = 0
T1    = 0
T2    = 0
```

This established that the machine had the subsystem disabled and unconfigured at that moment.

### 2. Setter/readback test while still disabled

The thresholds were written as:

```text
T1 = 80
T2 = 100
```

Readback returned exactly:

```text
Before: enabled=0
After:  enabled=0
0D13 = 80%
0D14 = 100%
```

This proved that the setters and getters were correctly identified before enabling the subsystem.

### 3. Enable test

After sending the enable command:

```text
Before enable: state=0, T1=80%, T2=100%
After enable : state=1, T1=80%, T2=100%
```

At approximately 89% battery, the adapter was connected. The battery state transitioned through a short transient and then settled repeatedly at:

```text
Not charging
power_now=0
```

This is direct live evidence that the EC stopped charging above the cap.

### 4. Re-charge threshold behavior

The battery was deliberately discharged below the cap.

Observed behavior:

- at 77–78%, charging still occurred,
- around 79–80%, charging stopped again.

This establishes practical hysteresis/cap behavior around the configured threshold. The Linux `capacity` value is integer-rounded, while the EC may be acting on a finer internal SOC value.

### 5. Persistence across reboot

After a normal reboot, without re-applying anything, GET returned:

```text
state = 1
T1    = 80
T2    = 100
```

Therefore the configuration **persists across a normal reboot**.

This does not yet prove persistence across a complete EC power loss / battery disconnect / true EC reset.

## Threshold semantics

Static firmware flow strongly shows threshold #1 (`0x0D13`) as the principal charging cutoff threshold: the charging-control routine compares live SOC at `0x0394` against it and switches to the zero-charge-current path once the threshold condition is met.

Threshold #2 (`0x0D14`) participates in a secondary branch/guard condition. A configuration of `T2=100` was chosen during validation to keep that secondary condition effectively out of the normal 80% cap path.

Therefore the **experimentally established practical configuration** is:

```text
T1 = desired cap
T2 = 100
```

Only the `80/100` configuration has been live-validated in this repository so far.

## Power-path observations at the cap

With AC connected and the battery capped, the machine showed:

```text
ACAD online=1
status=Not charging
power_now=0
```

Under a five-minute full-CPU stress run, battery energy dropped from approximately:

```text
63.154 Wh
```

to:

```text
62.661 Wh
```

and the battery later returned to `Charging` at 78% with about 28.1 W reported charging power.

The most reasonable interpretation is that the machine normally runs from the adapter while the battery is idle at the cap, but under sufficiently heavy load the platform may use **battery assist / hybrid power** temporarily and then recharge back toward the cap.

This is an observation of behavior, not a complete electrical characterization of the charger topology.

## What did not work

### Standard Linux threshold sysfs

The battery does not expose ordinary attributes such as:

```text
charge_control_start_threshold
charge_control_end_threshold
charge_behaviour
```

### Huawei-compatible WMI threshold API

The machine has Huawei-compatible WMI pieces, but the standard threshold GET using method ID `0x1103` returned failure/unsupported.

The corresponding SET path was intentionally **not** tested after GET failed.

### Generic Uniwill offsets

Generic MECHREVO/Tongfang/Uniwill software contains old-style offsets such as:

```text
0x07B9
0x07D0
```

Those are not the proven P916F host API and should not be copied onto this machine.

## Safety / rollback

The firmware-provided disable/reset operation is:

```text
F1 10
```

Static analysis shows that path clearing the enable bit and zeroing both thresholds.

Do not replace this protocol with arbitrary XRAM writes unless there is a very specific research reason to do so.

## Current known-good state of the researched machine

At the end of validation:

```text
state = 1
T1    = 80
T2    = 100
```

This state was confirmed after reboot.
