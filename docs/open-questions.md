# Open technical questions

## Battery charge limit

- Exact user-facing meaning of `T2` / `XRAM[0x0D14]`.
- Behavior of threshold pairs other than the validated `T1=80, T2=100` configuration.
- Exact internal SOC resolution and rounding around the stop/restart boundary.
- Persistence behavior across a true EC power loss or battery-controller power reset.
- Storage mechanism responsible for persistence across a normal reboot.

## Charger and power path

- Exact semantics of EC working words around `0x0D54..0x0D68`.
- End-to-end confirmation of the charger transactions associated with command numbers `0x14` and `0x15`.
- Electrical conditions under which the battery contributes energy while AC is online.

## PMC2

- Complete command map beyond the validated battery-control `F1/F2/F3` family.
- Presence of a firmware/interface version query or capability query.
- Response/error semantics beyond the single-byte responses observed in the validated transactions.

## IT5571 host access

The hypothesized dedicated I2EC interface at base `0x380` is rejected for the stock configuration. Remaining unknowns are:

- whether another factory/debug I2EC transport exists but is disabled;
- IT5571-specific differences from available IT5570 documentation;
- whether stock firmware exposes any safe generic full-XRAM read mechanism beyond known command handlers.

## Firmware setup

- Exact runtime behavior controlled by `Dynamic LID` / `AMD_PBS_SETUP + 0xDF`.
- Side effects and correctness of a permanent SetupUtility suppression patch around PE offset `0x2636A0`; only runtime SREP exposure has been validated.

## Boot logo update

The standard Insyde type-`0x54` and type-`0x6D` logo-update paths are unavailable on the tested BIOS 1.15 image.

Unresolved: whether P916F-STX implements a separate OEM-specific logo-update mechanism outside those two paths.

## Linux integration

The EC charge limiter is not exposed through the generic Linux power-supply threshold ABI. A native Linux integration has not been identified.

## Audio DSP

The exact Nahimic/A-Volute DSP/EQ configuration used by the Windows OEM stack has not been recovered.

## Firmware-version portability

The documented internal code addresses and protocol behavior are validated against the tested BIOS/EC revision only. Compatibility with future BIOS/EC revisions has not been established.
