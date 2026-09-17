# Battery charge-limit protocol

## Scope and validated behavior

This page documents the firmware-level battery charge-limit subsystem identified in the `P916F-STX` IT5571 EC and exercised through the live ITE PMC2 host interface. The machine-specific historical findings are retained from `SRC-BASELINE` (stable source crosswalk [S1](research-sources.md#project-sources)); the raw validation record is in [validation](validation.md). A later owner-supplied live capture is registered as [S18](research-sources.md#project-sources) and records the post-depletion power-loss state. The exact EC carve and its address-space limits are listed in the [artifact registry](research-artifacts.md#current-raw-32-mib-rom).

The recorded live configuration was:

```text
state=1
T1=80%
T2=100%
```

With that exact pair, charging stopped near the displayed 79–80% boundary, resumed below that region and retained the configuration across a normal reboot. The recorded reboot result did not require Windows Control Center to keep the state. A later live observation after battery depletion caused complete system power loss found `Enabled=0`, `T1=0`, `T2=0` on the next powered session. This establishes that the earlier configured state did not survive that recorded event; it does not establish the exact clearing mechanism or behavior for every G3, battery-disconnect or EC-reset class. These results apply to the investigated P916F-STX unit and their stated capture scope, not to arbitrary threshold pairs or other models. The stock battery device in the recorded Linux environment did not expose `charge_control_start_threshold`, `charge_control_end_threshold` or `charge_behaviour`; the firmware command path nevertheless worked.

The firmware range check accepts numeric values from 0 through 100 inclusive. Only the `T1=80%`, `T2=100%` pair has been behaviorally validated. The range check must not be turned into an unqualified claim that every arbitrary pair has known charging semantics.

## Linux power-supply devices and units

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

Useful live attributes include `capacity`, `status`, `voltage_now`, `power_now` and `energy_now`. In the observed `LCBT` sysfs tree, `current_now` was not present. In the recorded output, `capacity` is a percent; `voltage_now` is in µV; `power_now` is in µW; and `energy_now` is in µWh. Blank output is not interpreted as zero.

A representative capped state with AC connected was:

```text
capacity:   79%
status:     Not charging
power_now:  0
energy_now: 63154000
ACAD:       online=1
```

## EC-side state and firmware landmarks

Static reverse engineering of the exact P916F IT5571 firmware identified:

| EC XRAM | Role | Evidence |
|---|---|---|
| `XRAM[0x0D01].bit4` | Enable/state bit | Static-confirmed; state also observed through host GET |
| `XRAM[0x0D13]` | Threshold value 1 | Static-confirmed plus live setter/getter readback |
| `XRAM[0x0D14]` | Threshold value 2 | Static-confirmed plus live setter/getter readback |
| `XRAM[0x0394]` | SOC/battery percentage used by control logic | Static-confirmed plus host-MMIO corroboration |

These are EC XRAM addresses, not host physical addresses. Important locations in the carved EC image are:

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
CODE:0xF4D8      recorded PMC2 parser landmark
CODE:0xE65D      recorded PMC2 data-out helper landmark
```

The EC is banked. A `CODE:` address identifies a logical code location and does not automatically identify a byte offset in either firmware container; the complete bank-selection model was not retained. The [embedded-controller page](embedded-controller.md#ec-firmware-image-and-address-spaces) documents the preferred raw-ROM carve and related address spaces.

## Inclusive numeric range

Both threshold setters contain the same 8051 range-check pattern. In simplified form:

```asm
MOV  A,value
CLR  C
SUBB A,#0
JC   invalid

MOV  A,value
SETB C
SUBB A,#0x64
JNC invalid
```

The second compare intentionally sets carry before `SUBB`. On the 8051:

```text
SUBB A,src  =>  A = A - src - C
```

For `A=100`, the operation is effectively `100 - 100 - 1`, which borrows; `JNC invalid` is therefore not taken and 100 remains valid. For `A=101`, the result does not borrow and the invalid branch is taken. The accepted encoded range is consequently 0 through 100 inclusive (`0x00` through `0x64` as a numeric byte).

This strongly establishes percentage-like fields. It does not define the user-facing meaning of every possible `T1`/`T2` pair.

## PMC2 host protocol

The exact command family exercised on live hardware is carried by the ITE PMC2 logical device:

| Operation | Command byte written to I/O `0x6C` | Argument/data byte written to I/O `0x68` | Recorded meaning | Validation |
|---|---:|---:|---|---|
| Disable/reset | `0xF1` | `0x10` | Clears enable state and both threshold fields in the static path | Static-confirmed only; not live-tested |
| Enable | `0xF1` | `0x11` | Enables the programmed subsystem | Live-confirmed |
| Read state | `0xF1` | `0x12` | Returns the enable state | Live-confirmed |
| Read T1 | `0xF1` | `0x13` | Returns threshold field 1 / `XRAM[0x0D13]` | Live-confirmed |
| Read T2 | `0xF1` | `0x14` | Returns threshold field 2 / `XRAM[0x0D14]` | Live-confirmed |
| Set T1 to 80% | `0xF2` | `0x50` | `0x50` is 80 decimal | Live-confirmed at 80% |
| Set T2 to 100% | `0xF3` | `0x64` | `0x64` is 100 decimal | Live-confirmed at 100% |

The historical validation log labels the last two transactions `F2 80` and `F3 100`; those labels are preserved verbatim in the raw blocks below. Outside raw evidence, the wire pairs are `0xF2` with data `0x50` and `0xF3` with data `0x64`. The encoded byte is not ASCII text or packed decimal.

The successful host transaction sequence was:

1. Wait until PMC2 `IBF` (status bit 1) is clear.
2. Write the command byte (`0xF1`, `0xF2` or `0xF3`) to command/status I/O `0x6C`.
3. Wait for `IBF` to clear again.
4. Write the command's argument or subcommand byte to data I/O `0x68`.
5. For a returning command, wait for `OBF` (status bit 0) and read the response from data I/O `0x68`.

The threshold setters returned accepted values of 80 and 100 in the recorded experiment, and the state and threshold getters returned coherent one-byte values. The complete reply/error contract for enable, reset, invalid arguments and timeouts was not retained; operations are not assumed to have identical response lengths. Timing bounds, stale-output handling, transaction ownership and concurrent access by other platform software are not established. This is a recovered protocol description, not a complete production transport implementation.

## Validation chronology

### 1. Initial live state

The first successful PMC2 GET produced:

```text
F1 12 enabled/state : 0x00 (0)
F1 13 threshold #1 : 0x00 (0%)
F1 14 threshold #2 : 0x00 (0%)
```

This established coherent zero/default state before any setter was exercised.

### 2. Setter/readback while still disabled

The subsystem was deliberately left disabled while writing the thresholds. The recorded output was:

```text
Before: enabled=0
F2 80 response  : 80
F3 100 response : 100
After: enabled  = 0
0D13            = 80%
0D14            = 100%
```

This established that:

- `0xF2` reaches threshold field `XRAM[0x0D13]`;
- `0xF3` reaches threshold field `XRAM[0x0D14]`;
- the values read back through `0xF1 0x13` and `0xF1 0x14` match what was written;
- writing thresholds does not implicitly enable the subsystem;
- the SET transactions returned response bytes corresponding to the accepted values in this test.

### 3. Enable test

Only after the thresholds had been verified was the enable command sent:

```text
Before enable: state=0, T1=80%, T2=100%
Sending F1 11 (ENABLE)...
After enable : state=1, T1=80%, T2=100%
```

This is live confirmation that `0xF1 0x11` changes the enabled state without destroying the programmed threshold values.

### 4. Stop-charging behavior above the configured boundary

At the beginning of the test the battery was around 89%. The first roughly ten seconds were captured with the adapter physically disconnected, so those initial `Discharging` samples are not evidence about the cap. After AC was connected, the observed transition included:

```text
16s  status=Charging      cap=89%  power_now=2263000
18s  status=Discharging   cap=89%  power_now=246000
20s  status=Not charging  cap=89%  power_now=0
22s  status=Not charging  cap=89%  power_now=0
24s  status=Discharging   cap=89%  power_now=903000
26s  status=Not charging  cap=89%  power_now=0
28s  status=Not charging  cap=89%  power_now=0
```

The short transition states occurred while the EC/charger changed operating state. The important repeated settled observation was:

```text
AC connected
SOC well above 80%
status = Not charging
power_now = 0
```

This is direct behavioral evidence that the enabled firmware feature prevented continued charging above the configured 80 value in the tested configuration.

### 5. Boundary behavior while discharging and recharging

The battery was intentionally discharged below the boundary and then observed with AC connected:

```text
77–78% displayed: still charging
around 79% / slightly above: charging stopped
```

A representative capped sample was:

```text
79% - Not charging
```

This establishes a practical transition around the configured value within the coarse integer resolution of Linux `capacity`. It does not establish an exact internal hysteresis width, fractional SOC threshold or rounding rule. The EC may compare a differently rounded or higher-resolution SOC value.

### 6. Persistence across normal reboot

After rebooting Linux without re-applying anything, the same GET sequence returned:

```text
state = 1
T1    = 80
T2    = 100
```

The configuration therefore survived a normal reboot.

### 7. State after battery-depletion full power loss

A later observation was recorded after battery depletion caused complete system power loss. On the next powered session, the battery-limit query returned:

```text
Enabled : 0 (OFF)
T1      : 0% (0x00)
T2      : 0% (0x00)
```

At the same time, AC was online, SOC was 89%, and the battery reported `Charging` with `Power=21137000`, `Energy=64350000` and `Voltage=13431000` in the same capture. The complete raw text is preserved in [validation](validation.md#battery-depletion-full-power-loss-observation) and the standalone [power-loss observation](battery-limit-power-loss-observation.md).

This is **Live-confirmed** evidence that the earlier `1 / 80 / 100` state did not survive this recorded battery-depletion full-power-loss event. It does not prove when or why the values were cleared, whether the static `0xF1 0x10` handler ran, whether the fields are stored only in volatile memory, or whether every G3, battery-disconnect, CMOS/RTC-power removal, firmware-update or EC-reset condition behaves identically.

The practical boundary is direct: after battery depletion fully powers the machine off, the charge-limit state should be read back before relying on it.

## Threshold semantics

### T1

Static code around `CODE:0xC063` compares the live SOC byte at `XRAM[0x0394]` against `XRAM[0x0D13]` before entering different charger-control branches. In the tested configuration, `T1=80%` correlated with charging stopping around 80%. T1 is therefore strongly established as a threshold that directly controls the practical cap behavior in that configuration.

### T2

`XRAM[0x0D14]` is a second range-checked 0–100 threshold consumed by the same control routine. It participates in additional ordering/guard comparisons after the `XRAM[0x0D13]` check, but its exact end-user meaning has not been fully mapped.

The test used:

```text
T1 = 80
T2 = 100
```

100 was a conservative high value for the second field while isolating the behavior associated with the 80 setting. The evidence does not justify a generic rule such as:

```text
T1 = any desired cap
T2 = 100
```

What is proven is only that `T1=80%`, `T2=100%` works as an approximately 80% cap on this machine. T2 is not identified here as a recharge threshold or as a precisely measured hysteresis boundary.

## Charger-control working values

The decision logic derives working words around:

XRAM[0x0D65]/XRAM[0x0D66]
XRAM[0x0D67]/XRAM[0x0D68]

from source/default words around:

```text
XRAM[0x0D54..0x0D57]
```

Helpers around `CODE:0xC249` and `CODE:0xC27A` clear or copy these words depending on the selected charge-control branch. A later firmware worker stages transactions with command numbers `0x14` and `0x15`. Those numbers are consistent with standard Smart Battery charger `ChargingCurrent` and `ChargingVoltage` conventions, but that semantic naming remains an inference until the complete bus transaction path is independently decoded.

## AC and battery power observations

With AC connected and the battery capped, one stable snapshot was:

```text
capacity:   79%
status:     Not charging
power_now:  0
energy_now: 63154000
ACAD:       online=1
```

This shows that the battery was neither reported as charging nor reporting battery power flow at that instant. Battery sysfs does not directly measure wall-side adapter draw, so the snapshot does not prove that the adapter supplied every instantaneous system watt.

During a recorded five-minute `stress-ng --cpu 0` run:

```text
start:
79%  Not charging  power=0         energy=63154000

end:
78%  Charging      power=28128000  energy=62661000
```

The retained energy values were:

```text
63.154 Wh -> 62.661 Wh
Delta     = 0.493 Wh
```

The same values are `63154000 µWh` and `62661000 µWh`; their difference is `493000 µWh = 0.493 Wh`. The two timestamped snapshots in the detailed validation record are 313 seconds apart, whereas the stress tool reports a 300-second (five-minute) run. Those durations are not interchangeable for a precise average-power calculation.

The battery therefore contributed net stored energy during that high-load interval before charging resumed below the cap. Battery-assist or hybrid-power operation is a plausible interpretation under load, but the test did not measure adapter input power, resolve instantaneous current paths or establish the complete charger/power-path topology.

## Paths tested and rejected

| Alternative | Recorded result and boundary |
|---|---|
| Generic Linux threshold sysfs | `charge_control_start_threshold`, `charge_control_end_threshold` and `charge_behaviour` were absent from the observed battery device |
| Huawei-compatible WMI threshold API | Read-only GET `0x1103` returned failure/unsupported; corresponding SET `0x1003` was not attempted |
| Generic Uniwill/Tongfang offsets | `0x07B9` and `0x07D0` are comparative software constants, not established P916F control addresses |
| `INOU0000` / `ECRR` / `ECRW` | Not found in the examined ACPI tables |
| Dedicated I2EC base `0x380` | Rejected by the recorded `0xFF` candidate-read / valid-MMIO cross-check; see [embedded-controller](embedded-controller.md#rejected-dedicated-i2ec-candidate) |
| ACPI `_BTP` | Battery trip-point notification, not the charge-cap subsystem |

No arbitrary EC writes or cross-model offsets are required by the documented result.

## Disable/reset path

Static analysis maps the following pair to the reset/disable handler around `CODE:0xF508`:

```text
F1 10
```

Outside that raw historical label, the command is `0xF1` with data `0x10` written through the PMC2 command/status and data ports described above. The handler statically clears:

```text
XRAM[0x0D01].bit4
XRAM[0x0D13]
XRAM[0x0D14]
```

This path is **static-confirmed only**. It was not exercised live in the documented validation sequence because the known-good `80%/100%` state was intentionally left enabled. The post-depletion `0 / 0 / 0` observation does not prove that this handler executed. The path must not be described as a tested rollback or as a verified restoration of every factory charging parameter.

## Current known-good state and persistence boundary

The configured state that was behaviorally validated and that survived a normal reboot was:

```text
state = 1
T1    = 80
T2    = 100
```

This exact state remains the only threshold pair behaviorally validated in the retained record and the strongest known-good configured reference point for future work. Persistence is now separately bounded: a normal reboot retained `1 / 80 / 100`, while the later battery-depletion full-power-loss event was followed by `0 / 0 / 0`.

Remaining technical questions are tracked in [open technical questions](open-questions.md#battery-charge-limit), including T2 semantics, internal SOC resolution, the exact clearing mechanism and behavior under other reset/power-loss classes. No manual EC writer or production transport recipe is published here.
