# Live validation report

This document preserves the live tests used to establish the P916F-STX EC transport and battery charge-limit behavior. It includes raw observations where they materially support a conclusion.

The purpose is reproducibility: the static firmware analysis in [`embedded-controller.md`](embedded-controller.md) and [`battery-charge-limit.md`](battery-charge-limit.md) can be checked against the actual machine behavior recorded here.

## 1. Live ITE Super-I/O identity

ITE configuration space was checked at the common candidate ports.

Observed result:

```text
CFG 0x2E: chip=0xFFFF rev=0xFF
CFG 0x4E: chip=0x5571 rev=0x07
```

The valid interface is therefore `0x4E`.

### PMC2 logical device

Logical device `0x12` returned:

```text
PMC2 active = 0x01
I/O #0      = 0x0068
I/O #1      = 0x006C
I/O #2      = 0x0000
IRQ         = 0x00
```

This establishes the live host transport:

```text
PMC2 DATA            = 0x68
PMC2 COMMAND/STATUS  = 0x6C
```

This result was obtained independently of the EC disassembly and then matched the firmware command path.

## 2. Rejected candidate I2EC path

Before PMC2 was identified, static initialization code suggested a possible dedicated I2EC interface. A candidate base around `0x380` was tested **read-only**.

Observed output:

```text
I2EC control : EC[200D] = 0xFF
Battery %    : EC[0394] = 255
Threshold #1 : EC[0D13] = 255
Threshold #2 : EC[0D14] = 255
MMIO crosschk : FEEC2394 = 92
Cross-check   : MISMATCH
```

The same battery SOC field that returned `0xFF` through the candidate I2EC route simultaneously returned a plausible value through the known H2RAM/MMIO mapping:

```text
0xFEEC2394 = 92
```

Conclusion:

```text
candidate dedicated I2EC base 0x380 -> rejected
```

The path did not return real EC XRAM on the stock machine.

## 3. PMC transaction behavior

The successful transactions used:

```text
status bit0 = OBF
status bit1 = IBF
```

Working sequence:

1. wait for `IBF=0`;
2. write command byte to `0x6C`;
3. wait for `IBF=0`;
4. write subcommand/data byte to `0x68`;
5. if a response is expected, wait for `OBF=1`;
6. read response byte from `0x68`.

No brute-force command scanning was used. Commands were derived from the EC firmware first and exercised afterward.

## 4. Initial charge-limit state

Before any setter was exercised, the read commands returned:

```text
F1 12 enabled/state : 0x00 (0)
F1 13 threshold #1 : 0x00 (0%)
F1 14 threshold #2 : 0x00 (0%)
```

Baseline:

```text
state = 0
T1    = 0
T2    = 0
```

This is important because the protocol produced coherent default state before any mutation.

## 5. Threshold write/readback while disabled

The subsystem was intentionally kept disabled while testing the two setters.

Transactions:

```text
F2 80
F3 100
```

Observed output:

```text
Before: enabled=0
F2 80 response  : 80
F3 100 response : 100
After: enabled  = 0
0D13            = 80%
0D14            = 100%
```

Conclusions:

- `F2` writes threshold field `0x0D13`;
- `F3` writes threshold field `0x0D14`;
- accepted values are returned in the observed SET response;
- the GET path reads the same values back;
- threshold writes do not implicitly enable the subsystem.

## 6. Enable transition

After confirming the two threshold values, the enable command was issued:

```text
F1 11
```

Observed state:

```text
Before enable: state=0, T1=80%, T2=100%
Sending F1 11 (ENABLE)...
After enable : state=1, T1=80%, T2=100%
```

This validates `F1 11` as the enable operation and shows that enabling preserves the previously programmed thresholds.

## 7. Charge-stop behavior above the configured region

The battery was approximately 89% when AC was connected.

The first part of the trace was captured before the adapter was physically connected; the adapter was connected during the sequence.

Observed trace:

```text
00s  status=Discharging  cap= 89% power_now=3906000 voltage_now=13003000
02s  status=Discharging  cap= 89% power_now=4609000 voltage_now=12984000
04s  status=Discharging  cap= 89% power_now=3648000 voltage_now=13002000
06s  status=Discharging  cap= 89% power_now=3612000 voltage_now=13003000
08s  status=Discharging  cap= 89% power_now=3953000 voltage_now=13000000
10s  status=Discharging  cap= 89% power_now=3601000 voltage_now=13004000
12s  status=Discharging  cap= 89% power_now=4140000 voltage_now=12999000
14s  status=Discharging  cap= 89% power_now=4140000 voltage_now=12999000
16s  status=Charging     cap= 89% power_now=2263000 voltage_now=13052000
18s  status=Discharging  cap= 89% power_now=246000  voltage_now=13053000
20s  status=Not charging cap= 89% power_now=0       voltage_now=13054000
22s  status=Not charging cap= 89% power_now=0       voltage_now=13054000
24s  status=Discharging  cap= 89% power_now=903000  voltage_now=13042000
26s  status=Not charging cap= 89% power_now=0       voltage_now=13054000
28s  status=Not charging cap= 89% power_now=0       voltage_now=13054000
```

The significant steady-state observation is:

```text
SOC well above 80%
AC connected
status = Not charging
power_now = 0
```

The intermediate `Charging` / low-power `Discharging` samples show transition behavior while the EC/charger changed state; they do not invalidate the later stable capped condition.

## 8. Boundary behavior below the cap

The battery was then intentionally discharged below the configured region and observed while AC was connected.

Observed behavior:

```text
displayed 77–78% -> charging continued
around displayed 79% / slightly above -> charging stopped
```

Representative capped sample:

```text
79% - Not charging
```

Linux `capacity` is integer-valued. Therefore this establishes a practical transition near the configured 80 value but does **not** establish an exact internal hysteresis width, fractional SOC threshold or rounding rule.

## 9. AC-online capped snapshot

A stable snapshot at the boundary was:

```text
=== BATTERY ===
capacity:   79%
status:     Not charging
power_now:  0
energy_now: 63154000
charge_now:
voltage:    12764000

=== AC ===
ACAD: online=1
```

This establishes that, at that instant:

```text
AC adapter present
battery not charging
reported battery power = 0
```

The snapshot alone does not measure wall-side adapter power, so it is not treated as a complete power-path characterization.

## 10. Reboot persistence

After a normal reboot, without re-applying thresholds or enable, the same GET sequence returned:

```text
state = 1
T1    = 80
T2    = 100
```

Conclusion:

```text
configuration persists across a normal reboot
```

This does not establish persistence across a complete EC power loss, battery disconnect or other condition that removes the EC's retained power/state.

## 11. High-load battery-energy test

A five-minute all-CPU load was used to determine whether the battery remained electrically idle under a heavier system load while AC was connected.

### Before load

```text
20:47:14  cap=79%  status=Not charging  power=0  energy=63154000
```

### CPU stress execution

```text
stress-ng: info:  [21163] setting to a 5 mins run per stressor
stress-ng: info:  [21163] dispatching hogs: 20 cpu
stress-ng: info:  [21163] skipped: 0
stress-ng: info:  [21163] passed: 20: cpu (20)
stress-ng: info:  [21163] failed: 0
stress-ng: info:  [21163] metrics untrustworthy: 0
stress-ng: info:  [21163] successful run completed in 5 mins
```

### After load

```text
20:52:27  cap=78%  status=Charging  power=28128000  energy=62661000
```

Stored battery energy changed from:

```text
63.154 Wh
```

to:

```text
62.661 Wh
```

Delta:

```text
0.493 Wh
```

over roughly five minutes.

The battery therefore supplied net stored energy during the load interval before charging resumed below the threshold region.

The measured result supports a battery-assist / hybrid-power interpretation under load, but it does not by itself identify the exact charger topology, adapter limit or control policy.

## 12. Current validated configuration

The known-good state retained by the documented machine is:

```text
state = 1
T1    = 80
T2    = 100
```

This is the only threshold pair currently validated behaviorally.

## 13. Validation boundaries

The following statements are **not** established by the tests above:

```text
T1=N, T2=100 always creates an N% cap
T2 is definitely a recharge threshold
T2 is definitely an upper/lower hysteresis boundary
configuration survives complete EC power loss
power_now=0 proves the adapter supplies every instantaneous system watt
```

Those remain open technical questions even though the `80/100` configuration itself is validated.
