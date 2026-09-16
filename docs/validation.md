# Live validation report

This report preserves the recorded P916F-STX EC and battery experiments from `SRC-BASELINE` (stable source crosswalk [S1](research-sources.md#project-sources)). No hardware tests were rerun during documentation consolidation. Capture dates, the complete original probe programs and some environment versions were not retained; this limits end-to-end reproduction, not the identity of the quoted observations. The report is the behavioral counterpart to the static analysis in [embedded-controller.md](embedded-controller.md) and [battery-charge-limit.md](battery-charge-limit.md).

Historical output is preserved verbatim. In labels such as `F2 80` and `F3 100`, the threshold arguments are decimal: the specified wire pairs are `0xF2 0x50` and `0xF3 0x64`. Other `F1` subcommand labels use hexadecimal notation in the protocol specification; raw labels below remain unchanged. Units in battery output are `capacity` in percent, `voltage_now` in µV, `power_now` in µW and `energy_now` in µWh. Blank output is not interpreted as zero.

## 1. Live ITE Super-I/O identity

Observed configuration-space result:

```text
CFG 0x2E: chip=0xFFFF rev=0xFF
CFG 0x4E: chip=0x5571 rev=0x07
```

The valid identity response was obtained at I/O `0x4E`.

### PMC2 logical device

Logical device `0x12` returned:

```text
PMC2 active = 0x01
I/O #0      = 0x0068
I/O #1      = 0x006C
I/O #2      = 0x0000
IRQ         = 0x00
```

The transport identified by that configuration was:

```text
PMC2 DATA            = 0x68
PMC2 COMMAND/STATUS  = 0x6C
```

The discovery was performed independently of the EC disassembly and subsequently correlated with the working command path.

## 2. Rejected candidate I2EC path

A candidate dedicated I2EC interface at base `0x380` was evaluated by querying state rather than writing the threshold fields. Address selection still required port transactions.

```text
I2EC control : EC[200D] = 0xFF
Battery %    : EC[0394] = 255
Threshold #1 : EC[0D13] = 255
Threshold #2 : EC[0D14] = 255
MMIO crosschk : FEEC2394 = 92
Cross-check   : MISMATCH
```

The known shared-memory SOC address returned:

```text
0xFEEC2394 = 92
```

The experiment's conclusion was:

```text
candidate dedicated I2EC base 0x380 -> rejected
```

The candidate interface did not return correlated EC state in the tested configuration.

## 3. PMC transaction behavior

The successful transactions used:

```text
status bit0 = OBF
status bit1 = IBF
```

The host waited for `IBF=0`, wrote the command byte to command/status I/O `0x6C`, waited again, and wrote the subcommand/data byte to data I/O `0x68`. When a response was expected, it waited for `OBF=1` and read `0x68`.

Commands were derived from static firmware analysis before the experiment. No brute-force command scan is part of this validation record. The trace does not establish bounded timeout values or concurrent-access safety.

## 4. Initial charge-limit state

Before any setter was exercised:

```text
F1 12 enabled/state : 0x00 (0)
F1 13 threshold #1 : 0x00 (0%)
F1 14 threshold #2 : 0x00 (0%)
```

Baseline state:

```text
state = 0
T1    = 0
T2    = 0
```

The getter sequence returned coherent disabled/default values before mutation.

## 5. Threshold write/readback while disabled

The recorded transaction labels were:

```text
F2 80
F3 100
```

These labels mean T1=80 decimal and T2=100 decimal; their wire data bytes are `0x50` and `0x64` respectively.

```text
Before: enabled=0
F2 80 response  : 80
F3 100 response : 100
After: enabled  = 0
0D13            = 80%
0D14            = 100%
```

The setters returned accepted values, getters read those values back, and the queried enable state remained zero. This separates threshold storage from subsystem activation.

## 6. Enable transition

Recorded operation:

```text
F1 11
```

Observed state transition:

```text
Before enable: state=0, T1=80%, T2=100%
Sending F1 11 (ENABLE)...
After enable : state=1, T1=80%, T2=100%
```

The specified operation is `0xF1 0x11`. The readback confirms activation while preserving the programmed threshold values; it does not specify the complete enable-command response framing.

## 7. Charge-stop behavior above the configured region

The battery was approximately 89%. The adapter was physically disconnected for the first part of the trace and connected during the sequence. Initial Discharging samples therefore do not test AC-connected limiting.

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

The repeated settled observation was:

```text
SOC well above 80%
AC connected
status = Not charging
power_now = 0
```

The trace contains intermediate Charging and Discharging samples. It establishes repeated inhibition after adapter connection, not an uninterrupted steady state throughout the entire interval.

## 8. Boundary behavior below the cap

After discharging below the configured region, the recorded observation with AC connected was:

```text
displayed 77–78% -> charging continued
around displayed 79% / slightly above -> charging stopped
```

Representative capped sample:

```text
79% - Not charging
```

This is an observed boundary description rather than a complete sampled charging curve. Integer-valued Linux `capacity` does not resolve an exact fractional SOC threshold, hysteresis width or rounding rule.

## 9. AC-online capped snapshot

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

The snapshot establishes:

```text
AC adapter present
battery not charging
reported battery power = 0
```

The empty `charge_now:` line is preserved as empty output. The snapshot does not measure wall-side adapter power or every instantaneous battery current.

## 10. Reboot persistence

After a normal reboot without reapplying the configuration:

```text
state = 1
T1    = 80
T2    = 100
```

Recorded conclusion:

```text
configuration persists across a normal reboot
```

The result does not establish persistence through complete EC power loss, battery disconnection, firmware update or another reset class.

## 11. High-load battery-energy test

The experiment applied approximately five minutes of all-CPU load while AC was connected.

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

The reported energy values were:

```text
63.154 Wh
```

and:

```text
62.661 Wh
```

The decrease was:

```text
0.493 Wh
```

Calculation: `(63154000 - 62661000) µWh = 493000 µWh = 0.493 Wh`. The two timestamped snapshots span 313 seconds; the stress tool reports a 300-second (five-minute) run. These durations are not interchangeable for a precise average-power calculation.

Reported stored energy decreased over the interval and charging was reported at the later sample. This supports net battery-energy contribution and is consistent with battery-assist operation. Adapter rating, instantaneous adapter draw, a continuous current trace and the complete power-path topology were not established by this test.

## 12. Current validated configuration

The retained known-good state was:

```text
state = 1
T1    = 80
T2    = 100
```

This is the only threshold pair behaviorally validated in this report.

## 13. Validation boundaries

The following statements were not established:

```text
T1=N, T2=100 always creates an N% cap
T2 is definitely a recharge threshold
T2 is definitely an upper/lower hysteresis boundary
configuration survives complete EC power loss
power_now=0 proves the adapter supplies every instantaneous system watt
```

The disable/reset path was not exercised. Thermal AML findings, including `THMM` ordering and the `_Q40`/`_Q81`/`_QA0`/`_QA1` query paths, are static source evidence documented separately in [thermal/performance interfaces](thermal-performance.md); they do not count as additional battery experiments. Package hashing and other source-recovery work likewise do not change this validation record.
