# Live validation log

This document preserves the most important **raw observed outputs** from the P916F-STX battery/EC investigation. It exists so later summaries can be checked against actual machine behavior rather than memory.

The snippets below are evidence records, not instructions.

## 1. Failed dedicated-I2EC cross-check

A read-only candidate I2EC test at base `0x380` returned:

```text
I2EC control : EC[200D] = 0xFF
Battery %    : EC[0394] = 255
Threshold #1 : EC[0D13] = 255
Threshold #2 : EC[0D14] = 255
MMIO crosschk : FEEC2394 = 92
Cross-check   : MISMATCH
```

Interpretation:

- candidate ports `0x381..0x383` were not returning real EC XRAM;
- the known ACPI/H2RAM MMIO path simultaneously returned a plausible SOC value (`92`);
- therefore the `0x380` dedicated-I2EC hypothesis was rejected.

## 2. Live ITE Super-I/O / PMC2 configuration

Configuration-space probing produced:

```text
CFG 0x2E: chip=0xFFFF rev=0xFF
CFG 0x4E: chip=0x5571 rev=0x07
  PMC2 active = 0x01
  I/O #0      = 0x0068
  I/O #1      = 0x006C
  I/O #2      = 0x0000
  IRQ         = 0x00
```

This is the key live hardware proof for:

```text
ITE config port = 0x4E
EC chip ID       = 0x5571
revision         = 0x07
PMC2 DATA        = 0x68
PMC2 CMD/STATUS  = 0x6C
```

## 3. Initial battery-limit GET state

Before any successful setter operation, PMC2 GETs returned:

```text
F1 12 enabled/state : 0x00 (0)
F1 13 threshold #1 : 0x00 (0%)
F1 14 threshold #2 : 0x00 (0%)
```

This established a coherent baseline of disabled / zero thresholds.

## 4. Threshold setter/readback with subsystem still disabled

After setting threshold #1 to 80 and threshold #2 to 100, but before enabling:

```text
Before: enabled=0
F2 80 response  : 80
F3 100 response : 100
After: enabled  = 0
0D13            = 80%
0D14            = 100%
```

This proves:

- `F2` changed T1;
- `F3` changed T2;
- the values round-tripped through `F1 13/14`;
- threshold writes alone did not enable the feature.

## 5. Enable transition

The next live step produced:

```text
Before enable: state=0, T1=80%, T2=100%
Sending F1 11 (ENABLE)...
After enable : state=1, T1=80%, T2=100%
```

This is the direct live evidence for `F1 11` as enable.

## 6. Adapter connection while SOC was above the cap

The first approximately ten seconds below were with the adapter unplugged. The adapter was connected afterward.

```text
00s  status=Discharging  cap= 89% current_now=         - power_now=   3906000 voltage_now=13003000
02s  status=Discharging  cap= 89% current_now=         - power_now=   4609000 voltage_now=12984000
04s  status=Discharging  cap= 89% current_now=         - power_now=   3648000 voltage_now=13002000
06s  status=Discharging  cap= 89% current_now=         - power_now=   3612000 voltage_now=13003000
08s  status=Discharging  cap= 89% current_now=         - power_now=   3953000 voltage_now=13000000
10s  status=Discharging  cap= 89% current_now=         - power_now=   3601000 voltage_now=13004000
12s  status=Discharging  cap= 89% current_now=         - power_now=   4140000 voltage_now=12999000
14s  status=Discharging  cap= 89% current_now=         - power_now=   4140000 voltage_now=12999000
16s  status=Charging     cap= 89% current_now=         - power_now=   2263000 voltage_now=13052000
18s  status=Discharging  cap= 89% current_now=         - power_now=    246000 voltage_now=13053000
20s  status=Not charging cap= 89% current_now=         - power_now=         0 voltage_now=13054000
22s  status=Not charging cap= 89% current_now=         - power_now=         0 voltage_now=13054000
24s  status=Discharging  cap= 89% current_now=         - power_now=    903000 voltage_now=13042000
26s  status=Not charging cap= 89% current_now=         - power_now=         0 voltage_now=13054000
28s  status=Not charging cap= 89% current_now=         - power_now=         0 voltage_now=13054000
```

The important steady-state evidence is the repeated:

```text
89% / AC connected / Not charging / power_now=0
```

condition after the transition.

## 7. State persisted across normal reboot

After reboot, without re-applying thresholds or enable:

```text
state = 1
T1    = 80
T2    = 100
```

This proves persistence across a normal reboot, but not necessarily across complete EC power loss.

## 8. Headless battery status near the cap

With the machine headless, Linux reported:

```text
LCBT: 89% - Discharging
```

Later, after intentionally discharging the battery and reconnecting AC:

```text
79% - Not charging
```

The user observed that displayed 77–78% still charged, while around 79% / slightly above the machine stopped charging.

Because Linux `capacity` is integer-valued, this observation is evidence of a transition near the configured value, not a precise measurement of internal hysteresis.

## 9. AC-online capped snapshot

A later live snapshot was:

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

This proves that AC was online while the battery itself was reporting zero power and `Not charging` at that instant.

## 10. Five-minute full-CPU load test

Before the stress interval:

```text
20:47:14  cap=79%  status=Not charging  power=0  energy=63154000
```

The machine then ran a five-minute all-CPU `stress-ng` load. The stressor completed successfully:

```text
stress-ng: info:  [21163] setting to a 5 mins run per stressor
stress-ng: info:  [21163] dispatching hogs: 20 cpu
stress-ng: info:  [21163] skipped: 0
stress-ng: info:  [21163] passed: 20: cpu (20)
stress-ng: info:  [21163] failed: 0
stress-ng: info:  [21163] metrics untrustworthy: 0
stress-ng: info:  [21163] successful run completed in 5 mins
```

Afterward:

```text
20:52:27  cap=78%  status=Charging  power=28128000  energy=62661000
```

Battery stored energy therefore changed from:

```text
63.154 Wh -> 62.661 Wh
```

for a delta of approximately:

```text
0.493 Wh
```

across the interval.

This proves the battery contributed net stored energy during the high-load period before the system resumed charging below the cap. Describing the electrical mechanism as "hybrid power" or "battery assist" is a reasonable inference, but not yet a complete charger-topology proof.

## 11. Current known-good battery-limit state

The final known-good configuration retained by the machine is:

```text
state = 1
T1    = 80
T2    = 100
```

This is the reference state against which future firmware changes or experiments should be compared.
