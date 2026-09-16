# MECHREVO Control Center findings

This document records what was learned from Windows MECHREVO Control Center packages while researching the Xingyao 14 / `P916F-STX`.

## Package families inspected

Two relevant package sets were examined during research:

- `ControlCenter_5.56.1.13_Mechrevo_GX.zip`
- `jxgm_21911.zip`, identified as **机械革命电竞服务中心 3.0.2.4**

The latter reported that the machine was not supported when launched, so its behavior cannot be treated as proof of P916F compatibility.

The `5.56.1.13` GX package contained a more useful service/backend stack.

## Important binaries

The extracted GX package contained, among other files:

```text
AiStoneService/GCUBridge.exe
AiStoneService/MyControlCenter/ACPIDriverDll.dll
AiStoneService/MyControlCenter/GCUService.exe
AiStoneService/MyControlCenter/GCUServicePlugin.dll
AiStoneService/MyControlCenter/GCUUtil.exe
```

These binaries were useful primarily as reverse-engineering clues and cross-checks against the machine's own firmware.

## Battery-protection UI

The UWP/control-center frontend contains battery-protection concepts including:

```text
BatteryProtection_Command
Protection_Status
HighCapacityMode
BalancedMode
HealthyMode
IsBatteryHealthSupport
BatteryProtection/Control
```

Observed frontend selection values:

```text
High-capacity / performance mode -> 0
Balanced mode                    -> 1
Healthy mode                     -> 2
```

These values are **UI/profile identifiers**, not proven direct percentage values.

## Service-side classes and enums

The exact GX `GCUService.exe` contains types/classes including:

```text
GCUService.MySystem.BatteryProtection2
MyECIO.MyEcCtrl
MyECIO.IOdriverEC
MyControlCenter.WMIEC
```

Battery-related enums include concepts such as:

```text
BatteryHealthProtection_Status:
  PERFORMANCE
  BALANCED
  HEALTHY

Battery_Commands:
  GET
  CHARGING_UP_LIMIT
  CHARGING_DOWN_LIMIT
  RECOVERY
  TYPE_C
  ...
```

Methods/names associated with upper/lower charging limits were also present.

Some method bodies were obfuscated/anti-tampered or resolved to runtime stubs when decompiled, so method names alone were not sufficient to recover the exact P916F protocol.

## Generic offsets found in the Windows software

Metadata in the GX service included constants such as:

```text
ADDR_BATTERY_CHARGE_LIMIT_UP   = 0x07B9
ADDR_BATTERY_CHARGE_LIMIT_DOWN = 0x07D0
```

These match known generic Uniwill/Tongfang-style EC register maps.

They **must not be interpreted as P916F-STX proof**.

The machine's exact firmware was later shown to have a different charge-limit subsystem based on:

```text
XRAM 0x0D13
XRAM 0x0D14
```

with a host command protocol over ITE PMC2.

Therefore the Control Center package appears to support multiple machine families, and generic metadata cannot be copied blindly to this laptop.

## ACPIDriverDll.dll

An exact extracted native DLL used during reverse engineering had SHA-256:

```text
97d7115943600c2a09951440859f9bd75fd0d8bff9db49c296c868b49df8c8c6
```

It referenced a Windows device path:

```text
\\.\ACPIDriver
```

and IOCTL values including:

```text
ReadEC  = 0x9C40A488
WriteEC = 0x9C40A48C
```

That DLL belongs to a generic driver family that expects ACPI methods such as `ECRR/ECRW` on an `INOU0000` device.

The P916F-STX ACPI tables were checked and **do not expose**:

```text
INOU0000
INOU
ECRR
ECRW
```

So this standard Uniwill ACPIDriver path is not the native P916F route for battery control.

## GCUService.exe hash

One exact service binary extracted for analysis had SHA-256:

```text
01225ef470420d50e51bc541d63dd5ed321835d40c4209106da9908c8f277a9c
```

## Research conclusion

The Windows software was useful for proving that MECHREVO ships battery-protection concepts and for exposing generic service architecture, but the decisive evidence came from the **P916F's own BIOS/EC firmware plus live hardware probing**.

For charge limiting, the exact, live-validated mechanism is documented in [`battery-charge-limit.md`](battery-charge-limit.md).
