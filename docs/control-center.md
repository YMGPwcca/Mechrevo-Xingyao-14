# MECHREVO Control Center findings

This document records what was learned from Windows MECHREVO Control Center packages while researching the Xingyao 14 / `P916F-STX`.

The key lesson is that OEM software contains useful clues, but **generic package metadata is not machine-specific proof**. The decisive P916F results came from the laptop's own BIOS/EC image plus live hardware probing.

## Package families inspected

### GX package

```text
ControlCenter_5.56.1.13_Mechrevo_GX.zip
SHA-256: d081d2b338068ca6fd1099be2f6762d522c1223796f20a800d47034842423449
```

This package contained the most useful service/backend stack for static inspection.

### 机械革命电竞服务中心 3.0.2.4 package

A second package:

```text
jxgm_21911.zip
```

was identified as:

```text
机械革命电竞服务中心 3.0.2.4
```

When launched on the researched machine it reported that the system was **not supported**. Consequently that package's runtime behavior is not evidence of P916F-STX compatibility.

## Important GX backend files

The extracted GX package contained, among others:

```text
AiStoneService/GCUBridge.exe
AiStoneService/MyControlCenter/ACPIDriverDll.dll
AiStoneService/MyControlCenter/GCUService.exe
AiStoneService/MyControlCenter/GCUServicePlugin.dll
AiStoneService/MyControlCenter/GCUUtil.exe
```

These names show a layered design: UI/frontend, service logic, EC/ACPI abstraction and a native driver bridge.

## Battery-protection UI concepts

The frontend contains battery-protection identifiers including:

```text
BatteryProtection_Command
Protection_Status
HighCapacityMode
BalancedMode
HealthyMode
IsBatteryHealthSupport
BatteryProtection/Control
```

Observed profile-selection values:

```text
High-capacity / performance mode -> 0
Balanced mode                    -> 1
Healthy mode                     -> 2
```

These numbers are **profile IDs**, not proven direct charge percentages.

No documentation in the inspected frontend established a reliable mapping such as "profile 2 = exactly 80%" for this P916F machine.

## Service-side battery types

The exact GX `GCUService.exe` contains types/classes such as:

```text
GCUService.MySystem.BatteryProtection2
MyECIO.MyEcCtrl
MyECIO.IOdriverEC
MyControlCenter.WMIEC
```

Battery-related enum names include:

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

The naming clearly shows that the broader Control Center codebase knows about battery-protection states and upper/lower charging limits.

However, method bodies relevant to this functionality were anti-tampered/obfuscated or decompiled as runtime/empty stubs in the inspected managed binary, so the names alone did not reveal the exact P916F transaction sequence.

## Generic old-style EC offsets found in GX metadata

The service metadata contains constants:

```text
ADDR_BATTERY_CHARGE_LIMIT_UP   = 1977 = 0x07B9
ADDR_BATTERY_CHARGE_LIMIT_DOWN = 2000 = 0x07D0
```

These addresses also appear in generic Uniwill/Tongfang-oriented EC work and are therefore believable as real offsets for **some** supported machines.

They are not the proven path on this P916F-STX.

The exact P916F firmware later revealed a separate charge-limit subsystem using:

```text
XRAM[0x0D13]
XRAM[0x0D14]
XRAM[0x0D01].bit4
```

and the feature was successfully controlled through PMC2 commands `F1/F2/F3`.

Therefore `0x07B9/0x07D0` should remain labelled **Comparative only** for this repository.

## Native ACPIDriverDll.dll

One exact native DLL extracted for analysis has:

```text
SHA-256: 97d7115943600c2a09951440859f9bd75fd0d8bff9db49c296c868b49df8c8c6
```

It references the Windows device path:

```text
\\.\ACPIDriver
```

and IOCTL values:

```text
ReadEC  = 0x9C40A488
WriteEC = 0x9C40A48C
```

The generic driver family associated with this DLL expects ACPI methods/interfaces such as:

```text
INOU0000
INOU
ECRR
ECRW
```

The P916F-STX ACPI tables were explicitly searched and these interfaces were **not present**.

Therefore the generic `ACPIDriver -> ECRR/ECRW -> old EC offset` route is not the native P916F battery-limit path.

## GCUService.exe artifact

One exact service binary used in the investigation has:

```text
SHA-256: 01225ef470420d50e51bc541d63dd5ed321835d40c4209106da9908c8f277a9c
```

The compact reverse-engineering bundle also contained `service.ini` and unrelated AirplaneDriver material, but no `ACPIDriver.sys` corresponding to the generic DLL path was present in that compact bundle.

## How Control Center evidence should be interpreted

The OEM package established several useful facts:

- MECHREVO software has explicit battery-protection UI concepts.
- The broader backend supports multiple EC/control mechanisms.
- Some supported machines use upper/lower threshold concepts and old-style offsets such as `0x07B9/0x07D0`.
- The package is multi-model and cannot be assumed to use one EC map everywhere.

What it did **not** establish by itself:

- that `0x07B9/0x07D0` belong to P916F-STX;
- that UI profile IDs 0/1/2 are direct percentages;
- that the generic `ACPIDriver` ACPI interface exists on this laptop;
- the exact PMC2 command bytes used by P916F.

Those P916F-specific facts were recovered from the exact machine firmware and validated live. See [`battery-charge-limit.md`](battery-charge-limit.md).
