# Battery charge limit

This is currently the most thoroughly live-validated low-level feature documented for the `P916F-STX`.

## Executive summary

The laptop has a real firmware-level battery charge-limit subsystem implemented in the ITE IT5571 EC.

It is reachable from Linux through the live ITE **PMC2** host interface:

```text
DATA            = 0x68
COMMAND/STATUS  = 0x6C
```

The stock Linux battery driver does not expose generic charge-threshold sysfs attributes for this machine, but the EC's own command protocol is functional.

The one configuration fully validated on live hardware is:

```text
state = enabled
T1    = 80
T2    = 100
```

With that exact pair the machine:

- stopped charging around the displayed 79–80% boundary,
- charged again when the battery had fallen below that boundary,
- retained `state=1, T1=80, T2=100` across a normal reboot,
- did not require the Windows Control Center to keep the state after reboot.

The firmware setter accepts numeric values from 0 through 100, but only the `80/100` pair has been behaviorally validated. Do **not** turn that into an unqualified claim that every arbitrary `T1/T2` pair has known semantics.

## Linux power-supply devices

The battery appears as:

```text
/sys/class/power_supply/LCBT
```

The AC adapter appears as:

```text
/sys/class/power_supply/ACAD
```

The retained battery model string is:

```text
588974-3S-G-A0
```

Useful live attributes include:

```text
capacity
status
voltage_now
power_now
energy_now
```

`current_now` was not present in the observed `LCBT` sysfs tree.

A representative capped state with AC connected was:

```text
capacity:   79%
status:     Not charging
power_now:  0
energy_now: 63154000
ACAD:       online=1
```

## EC-side state discovered statically

Static reverse engineering of the exact P916F IT5571 firmware identified:

| EC XRAM | Role | Evidence |
|---|---|---|
| `0x0D01` bit 4 | enable/state bit | Static-confirmed; state also observed through host GET |
| `0x0D13` | threshold value #1 | Static-confirmed + live set/readback |
| `0x0D14` | threshold value #2 | Static-confirmed + live set/readback |
| `0x0394` | SOC/battery percentage used by the control logic | Static-confirmed + host-MMIO corroboration |

Important routines in the carved EC image:

```text
CODE:0xED60...   enable/status handler family
CODE:0xED7A      sets XRAM[0x0D01].bit4
CODE:0xED8E      checks XRAM[0x0D01].bit4
CODE:0xEDBA      validates and writes XRAM[0x0D13]
CODE:0xEDDF      validates and writes XRAM[0x0D14]
CODE:0xF508      disable/reset path
CODE:0xF526      reads XRAM[0x0D13]
CODE:0xF621      reads XRAM[0x0D14]
CODE:0xC063      SOC/threshold decision logic
```

The host command dispatcher that reaches this family was traced around the later `0xF4xx` region; a PMC2 command parser was identified around `CODE:0xF4D8`.

## Numeric range validation: 0..100 inclusive

Both threshold setters contain the same 8051 range-check pattern. In simplified form:

```asm
MOV  A,value
CLR  C
SUBB A,#0
JC   invalid

MOV  A,value
SETB C
SUBB A,#0x64
JNC  invalid
```

The second compare intentionally sets carry before `SUBB`. On 8051:

```text
SUBB A,src  =>  A = A - src - C
```

For `A=100`, the operation is effectively `100 - 100 - 1`, which borrows; `JNC invalid` is therefore **not** taken and 100 remains valid.

For `A=101`, the result does not borrow; `JNC invalid` is taken.

Therefore the accepted numeric range is:

```text
0 <= value <= 100
```

This strongly establishes percentage-like fields. It does not fully define the user-facing meaning of every possible pair.

## Host protocol

The exact command family exercised on live hardware is:

| Command byte | Following data byte | Observed/static meaning | Validation |
|---|---:|---|---|
| `F1` | `10` | disable/reset subsystem; static path clears enable bit and both thresholds | Static-confirmed only |
| `F1` | `11` | enable subsystem | Live-confirmed |
| `F1` | `12` | GET enabled state | Live-confirmed |
| `F1` | `13` | GET threshold #1 / `0x0D13` | Live-confirmed |
| `F1` | `14` | GET threshold #2 / `0x0D14` | Live-confirmed |
| `F2` | `NN` | SET threshold #1 to `NN` | Live-confirmed at 80 |
| `F3` | `NN` | SET threshold #2 to `NN` | Live-confirmed at 100 |

The successful host transaction sequence was:

1. wait until PMC2 `IBF` (status bit 1) is clear;
2. write the command byte (`F1`, `F2` or `F3`) to `0x6C`;
3. wait for `IBF` to clear again;
4. write the command's argument/subcommand byte to `0x68`;
5. for a returning command, wait for `OBF` (status bit 0) and read the response from `0x68`.

The EC data-out helper used by the relevant firmware path writes its response to the PMC2 data-out register; one identified helper sits around `CODE:0xE65D`.

## Validation chronology

### 1. Initial live state

The first successful PMC2 GET produced:

```text
F1 12 enabled/state : 0x00 (0)
F1 13 threshold #1 : 0x00 (0%)
F1 14 threshold #2 : 0x00 (0%)
```

This was important because it proved the protocol could return coherent zero/default state before any setters were exercised.

### 2. Setter/readback while still disabled

The subsystem was deliberately left disabled while writing the thresholds.

Live output:

```text
Before: enabled=0
F2 80 response  : 80
F3 100 response : 100
After: enabled  = 0
0D13            = 80%
0D14            = 100%
```

This proved several things at once:

- `F2` reaches threshold #1;
- `F3` reaches threshold #2;
- the values read back through `F1 13/14` exactly match what was written;
- writing the thresholds does not implicitly enable the subsystem.

The returned `80` and `100` also show that these SET transactions produced response bytes corresponding to the accepted values in this test.

### 3. Enable test

After the thresholds had been verified, only then was the enable command sent.

```text
Before enable: state=0, T1=80%, T2=100%
Sending F1 11 (ENABLE)...
After enable : state=1, T1=80%, T2=100%
```

This is live confirmation that `F1 11` changes the enabled state without destroying the programmed threshold values.

### 4. Stop-charging behavior above the configured boundary

At the beginning of the test the battery was around 89%. The first roughly ten seconds were with the adapter physically disconnected, so those initial `Discharging` samples are not evidence about the cap.

After AC was connected, the observed transition included:

```text
16s  status=Charging      cap=89%  power_now=2263000
18s  status=Discharging   cap=89%  power_now=246000
20s  status=Not charging  cap=89%  power_now=0
22s  status=Not charging  cap=89%  power_now=0
24s  status=Discharging   cap=89%  power_now=903000
26s  status=Not charging  cap=89%  power_now=0
28s  status=Not charging  cap=89%  power_now=0
```

The short transition states are expected while the EC/charger changes operating state. The important observation is that the machine repeatedly settled at:

```text
AC connected
SOC well above 80%
status = Not charging
power_now = 0
```

That is direct behavioral evidence that the enabled firmware feature prevented continued charging above the configured 80 value.

### 5. Boundary behavior while discharging and recharging

The battery was intentionally discharged below the boundary and then observed with AC connected.

The user observed that:

```text
77–78% displayed: still charging
around 79% / slightly above: charging stopped
```

A representative capped sample was:

```text
79% - Not charging
```

This proves a practical transition around the configured value, within the coarse integer resolution of Linux's `capacity` property.

It does **not** yet prove the exact internal hysteresis width. Linux reports integer percent, while the EC may compare a differently rounded or higher-resolution SOC value internally. Accordingly this repository no longer labels the 77/78/79 observation as a precisely measured hysteresis band.

### 6. Persistence across normal reboot

After rebooting Linux without re-applying anything, the same GET sequence returned:

```text
state = 1
T1    = 80
T2    = 100
```

Therefore the configuration survives a normal reboot.

What remains unknown is whether it survives a **true EC power loss/reset**, such as whatever hardware condition actually clears EC retained state or disconnects the battery/controller power domain.

## What can be said about T1 and T2

### T1

Static code around `CODE:0xC063` compares live SOC at `0x0394` against `0x0D13` before entering different charger-control branches.

Live behavior with `T1=80` then stopped charging around 80.

Therefore T1 is strongly established as a threshold that directly controls the practical cap behavior in the tested configuration.

### T2

`0x0D14` is definitely a second validated 0..100 threshold consumed by the same control routine. It participates in additional ordering/guard comparisons after the `0x0D13` check.

Its exact end-user meaning has **not** yet been fully mapped.

The test used:

```text
T1 = 80
T2 = 100
```

because 100 was a conservative high value for the second field while isolating the behavior associated with the 80 setting.

It is therefore too strong to document a generic rule such as:

```text
T1 = any desired cap
T2 = 100
```

as proven behavior. What is proven is only that **80/100 works as an approximately 80% cap on this machine**.

## Charger-control internals related to the thresholds

The decision logic derives working words around:

```text
0x0D65/0x0D66
0x0D67/0x0D68
```

from source/default words around:

```text
0x0D54..0x0D57
```

Helpers around `CODE:0xC249` and `CODE:0xC27A` clear or copy these words depending on the selected charge-control branch.

A later firmware worker stages transactions with command numbers `0x14` and `0x15`. Those numbers are consistent with standard smart-charger `ChargingCurrent` and `ChargingVoltage` commands, but that semantic naming remains an **inference** until the complete bus transaction path is independently decoded.

## AC-power / battery-power observation at the cap

With AC connected and the battery capped, one stable snapshot was:

```text
capacity:   79%
status:     Not charging
power_now:  0
energy_now: 63154000
ACAD:       online=1
```

This shows that the battery was neither being reported as charging nor reporting battery power flow at that instant.

It is tempting to say this alone proves the whole laptop was powered directly from the adapter, but battery sysfs does not directly measure wall-side adapter draw. The stronger test was to watch stored battery energy under load.

During a five-minute `stress-ng --cpu 0` run:

```text
start:
79%  Not charging  power=0         energy=63154000

end:
78%  Charging      power=28128000  energy=62661000
```

The energy change was:

```text
63.154 Wh -> 62.661 Wh
Delta     = 0.493 Wh
```

across roughly five minutes.

This demonstrates that the battery did contribute energy during that high-load interval before charging resumed below the cap. The most plausible interpretation is a platform **hybrid/battery-assist** behavior under sufficient load, followed by recharge toward the cap.

That interpretation is useful, but it is not a complete electrical characterization of the charger/power-path topology.

## Paths that were tested and rejected

### Generic Linux threshold sysfs

The battery did not expose:

```text
charge_control_start_threshold
charge_control_end_threshold
charge_behaviour
```

### Huawei-compatible WMI threshold API

A Huawei-style threshold GET using method ID `0x1103` returned failure/unsupported.

The corresponding SET method `0x1003` was deliberately **not** tested after GET failed.

### Generic Uniwill/Tongfang charge-limit offsets

Windows OEM software contained generic constants:

```text
0x07B9
0x07D0
```

The P916F's exact firmware does not require those offsets for the working host protocol. They are comparative evidence only and should not be written directly on this machine.

### Dedicated I2EC at `0x380`

A read-only cross-check returned `0xFF` for all candidate I2EC reads while host MMIO returned a real battery value. That path is rejected; see [`embedded-controller.md`](embedded-controller.md).

## Disable/reset path

Static analysis maps:

```text
F1 10
```

to the reset/disable handler around `CODE:0xF508`, which clears:

```text
XRAM[0x0D01].bit4
XRAM[0x0D13]
XRAM[0x0D14]
```

This rollback command has **not** been exercised live in the documented validation sequence, because the known-good 80/100 state was intentionally left enabled.

## Current known-good state

At the end of the experiment and again after reboot:

```text
state = 1
T1    = 80
T2    = 100
```

That exact state is the strongest known-good reference point for future work on this machine.
