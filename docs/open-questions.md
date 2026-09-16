# Open questions

This file separates **known facts** from things that are still only partially understood.

## Battery charge limit

### Persistence across a true EC power loss

A normal reboot preserved:

```text
state = 1
T1    = 80
T2    = 100
```

What is not yet proven is persistence across a complete EC power loss, for example a battery disconnect or whatever sequence actually clears the controller's retained state.

### Exact semantic role of T2

`T1` is strongly established as the practical charge cutoff threshold.

`T2` is consumed by a secondary branch/guard in the charging logic, but its complete user-facing meaning has not been experimentally mapped. `T2=100` was used during the successful 80% cap validation to keep that secondary condition out of the normal operating range.

### Other cap values

The firmware setter validates values in the 0..100 range, but only the `80/100` configuration has been live-validated in this research so far.

## Battery / charger topology

At the cap, AC-online + `Not charging` + `power_now=0` strongly indicates the adapter is powering the system while the battery is idle.

Under heavy CPU load, battery energy decreased temporarily and then the battery recharged below the cap. This is consistent with hybrid/battery-assist behavior, but the exact charger power-path implementation and adapter telemetry have not been fully decoded.

## Firmware setup

### Dynamic LID

The BIOS contains `Dynamic LID` / `AmdDynamicLid` references, but their exact practical effect on this machine has not been established.

### Boot logo replacement

The OEM boot animation resource and its firmware container were found, but there is no proven low-risk method to replace branding without rewriting firmware storage.

## WMI

The Huawei-compatible WMI path exists, but the commonly expected battery-threshold API was unsupported in testing. It remains possible that other OEM WMI functions have useful undocumented meanings, but they should be reverse-engineered from this firmware rather than guessed.

## Linux integration

The battery cap works at firmware level but is not exposed through the standard Linux power-supply threshold attributes.

A future upstream-quality integration would need to decide whether the best abstraction is:

- a platform driver exposing standard charge-control sysfs, or
- a dedicated MECHREVO/ITE platform interface.

This repository remains documentation-only and does not contain such code.

## Audio

Linux speaker playback works, but the exact OEM Nahimic EQ/DSP curve used by Windows has not been recovered. Recreating that tuning remains the main unresolved audio-quality task.

## Version portability

All low-level EC protocol findings should be assumed specific to the tested P916F-STX firmware family until revalidated on another BIOS/EC revision.

A future BIOS update could move command handlers, change defaults or alter semantics even if the external protocol remains similar.
