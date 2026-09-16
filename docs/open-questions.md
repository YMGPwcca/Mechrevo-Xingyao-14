# Open questions

This file deliberately separates unresolved questions from established facts so future work does not accidentally promote an inference into a platform truth.

## Battery charge-limit subsystem

### Persistence across a true EC power loss

A normal reboot preserved:

```text
state = 1
T1    = 80
T2    = 100
```

What is **not** yet proven is persistence across a real EC power loss/reset condition—for example a battery/controller power-domain reset or battery disconnect.

A normal OS reboot is not equivalent to removing power from the EC.

### Where the persistent state is stored

Because the values survived reboot, some persistence mechanism exists from the host's perspective, but the exact storage mechanism is not yet identified.

Possibilities include retained EC RAM under continuously powered EC rails or explicit nonvolatile storage/reload logic. The current evidence does not justify choosing one yet.

### Exact semantic role of T2 / `0x0D14`

T2 is unquestionably:

- a 0..100 validated field,
- read/write through the same battery-limit command family,
- consumed by the charge-control decision routine.

What remains unresolved is its exact user-facing meaning.

The successful validation used:

```text
T1 = 80
T2 = 100
```

and produced an approximately 80% cap, which proves T1 has direct practical influence in that configuration. It does **not** prove that T2 is simply an "upper threshold," "lower threshold," recharge threshold or hysteresis value.

### Other threshold pairs

The firmware accepts numeric values 0 through 100 for both fields, but only `80/100` has been behaviorally validated.

Questions still open:

- Does `T1=70, T2=100` produce a ~70% cap?
- Are there ordering constraints beyond what the setter itself validates?
- Does T2 alter restart/stop behavior under some SOC range?
- Are some combinations accepted but nonsensical?

Until tested, documentation should not advertise a generic formula for arbitrary values.

### Exact internal SOC resolution

Linux reports integer `capacity`, while the EC compares `XRAM[0x0394]`.

The observed transition occurred around displayed 79–80%, but this is insufficient to measure a precise hysteresis band or rounding rule.

A better future experiment would correlate repeated direct host-visible `0xFEEC2394` reads with sysfs `capacity` around the transition.

## Battery / charger power path

At the cap, a stable state showed:

```text
ACAD online=1
status=Not charging
power_now=0
```

Under a five-minute full-CPU load, stored battery energy still decreased before charging resumed at 78%.

This is consistent with battery assist / hybrid power under load, but several details remain unknown:

- the adapter's actual wall/input power during the event;
- whether the battery assists only above a platform-power threshold;
- whether the charger intentionally uses battery assist or the adapter simply reaches a limit;
- exact meanings of the EC working words around `0x0D54..0x0D68`;
- full end-to-end confirmation that staged command numbers `0x14/0x15` are charger current/voltage transactions on this board.

## PMC2 command family

The battery-related `F1/F2/F3` commands are understood well enough for the documented validation, but the larger PMC2 protocol is not fully mapped.

Open questions include:

- what other top-level command bytes are implemented;
- whether there is a version/capability query;
- whether the battery command family has additional undocumented subcommands;
- whether response framing/error codes exist beyond the single-byte responses observed.

No brute-force command probing should be used to answer these questions.

## Dedicated I2EC

The hypothesized stock dedicated I2EC window at `0x380` failed a live read-only cross-check and is rejected as a usable path.

Still unresolved:

- whether another I2EC transport is deliberately disabled but could exist for factory/debug use;
- the exact IT5571-D differences from available IT5570 documentation/reference work;
- whether any safe stock host path can read arbitrary full 16-bit XRAM besides known command handlers.

These questions are research-only; the working battery feature does not require solving them.

## Firmware setup

### Dynamic LID

Static firmware analysis found:

```text
Dynamic LID / AmdDynamicLid
AMD_PBS_SETUP + 0xDF
default 0
```

Its exact practical behavior on this machine remains unverified. The name alone does not prove "open lid to power on."

### Hidden Boot settings

The SetupUtility suppression block and Quiet Boot question are understood statically, but the candidate binary suppression change has not been live-tested.

### Boot-logo replacement

The embedded GIF resource, GUID and associated modules are known, but no low-risk runtime method has been proven that changes only the branding without a firmware rewrite.

## WMI

Huawei-compatible WMI plumbing exists, but the expected battery-threshold GET `0x1103` returned unsupported/failure and SET `0x1003` was intentionally not attempted.

Other WMI functions may still have undocumented meanings, but they should be decoded from this exact firmware rather than inferred from a different Huawei/MECHREVO implementation.

## Linux integration

The charge cap works at firmware level but is not exposed through standard Linux power-supply threshold files.

A future Linux integration would need to decide whether to expose the feature through:

- the generic power-supply charge-control ABI, or
- a dedicated platform/EC interface.

This repository intentionally remains documentation-only.

## Audio

Linux detects ALC256 and drives the speakers, but the exact OEM Nahimic/A-Volute DSP/EQ configuration has not been recovered.

The main unresolved audio task is reproducing the Windows tuning, not basic codec enumeration.

## Version portability

All EC command/address findings should be assumed specific to the tested P916F-STX firmware family until revalidated after a firmware update or on another unit.

A future BIOS/EC revision may:

- preserve the same external PMC2 protocol,
- change internal code addresses while keeping behavior,
- alter threshold semantics/defaults,
- or remove/replace the command family entirely.

External protocol bytes are more likely to remain stable than internal `CODE:` addresses, but even that is not guaranteed without re-testing.
