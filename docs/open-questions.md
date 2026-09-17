# Open technical questions

This page records unresolved hardware, firmware and behavior questions. It does not turn a source-recovery task into a technical conclusion. Source and publication work are tracked in [`documentation-status.md`](documentation-status.md#pending-evidence); technical questions remain here.

## Battery charge limit

The `T1=80%`, `T2=100%` pair and its normal-reboot readback are behaviorally recorded. A later live observation after battery depletion caused complete system power loss found `Enabled=0`, `T1=0`, `T2=0` on the next powered session, so persistence across that recorded event is no longer unresolved. The following remain unresolved:

- Exact user-facing meaning of `T2` / `XRAM[0x0D14]`.
- Behavior of threshold pairs other than the validated `T1=80%`, `T2=100%` configuration.
- Exact internal SOC resolution and rounding around the stop/restart boundary.
- Exact hysteresis width between charging stop and restart.
- The exact transition that cleared the battery-limit fields during the recorded battery-depletion full-power-loss event: depletion/brownout, an EC reset, firmware initialization on the next power-on, or another event-associated transition.
- Whether other power-loss classes such as battery disconnect, CMOS/RTC-power removal, explicit EC reset, firmware update or other G3 entries produce the same clearing behavior.
- Storage mechanism responsible for persistence across a normal reboot and loss across the recorded depletion event.
- Whether the statically identified `0xF1 0x10` reset/disable path has the expected user-facing rollback behavior; its clearing logic is not a live rollback result.

The setter range check establishes an inclusive stored range of decimal `0..100`; it does not establish an arbitrary percentage cap or a complete transport contract. See the [battery-depletion power-loss observation](battery-limit-power-loss-observation.md) for the new live capture and its boundaries.

## Charger and power path

- Exact semantics of EC working words around `XRAM[0x0D54..0x0D68]`.
- End-to-end confirmation of the charger transactions associated with command numbers `0x14` and `0x15`.
- Electrical conditions under which the battery contributes energy while AC is online.
- Whether the observed AC-online `Not charging` / `power_now=0` state is selected by charger policy, system load, battery state or another condition.
- Complete charger topology and wall-side adapter power behavior.

The retained energy decrease during CPU load is an electrical observation at the battery telemetry boundary, not a complete power-path model.

## PMC2 and ACPI/WMI

- Complete PMC2 command map beyond the validated battery-control `0xF1`/`0xF2`/`0xF3` family.
- Presence of a firmware/interface version query or capability query.
- Response and error semantics beyond the single-byte responses observed in validated transactions.
- Whether another factory/debug I2EC transport exists but is disabled.
- IT5571-specific differences from available IT5570 documentation.
- Whether stock firmware exposes any safe generic full-XRAM read mechanism beyond known command handlers.
- Decoded BMOF class/member definitions and the exact schema of the extended WMI methods; visible GUID registration and an AML wrapper do not establish the proposed 64/256/4128-byte contracts.
- Raw `dsdt.dat` bytes and digest for the recovered September ACPI extraction. The extraction commands, table length/header and ACPICA version are recovered, but the raw table object is not.

The hypothesized dedicated I2EC interface at base `0x380` is rejected for the stock configuration because its retained read-only cross-check returned `0xFF` while the H2RAM/MMIO path returned a plausible SOC. That rejection does not answer whether a different disabled debug transport exists.

## Firmware setup and runtime visibility

- Exact runtime behavior controlled by `Dynamic LID` / `AMD_PBS_SETUP + 0xDF`.
- Whether `Dynamic LID` controls open-lid power-on, another lid policy, or only a setup/UI concept.
- Side effects and correctness of a permanent SetupUtility suppression patch around extracted-PE file offset `0x2636A0`; only runtime SREP exposure has been recorded.
- Which suppressed child forms become visible under each SREP operation and whether the revealed controls correspond to supported hardware.
- Behavior of the other proposed setup overlays (AC Loss, Auto Wake S5, Charger BYPASS, Wake On Voice, Human Presence, GbE Power, WLAN S3/S4 and LOM D3Cold) on the documented unit.
- Whether a saved setup-variable change is required to alter behavior after a runtime form reveal.
- Exact SREP executable build/digest and which recovered candidate configuration produced the photographed expanded Boot page. The photograph itself is recovered; session-to-config association remains unresolved.

IFR defaults, static form presence, runtime visibility and live variable values remain separate questions. A default or a visible form is not a behavioral result.

## Thermal and performance control

The recovered AML establishes field layouts and dispatch relationships. A recovered live capture additionally correlates GFNS results with the two changing H2RAM FNS fields. It still does not establish a complete fan controller or a safe write interface.

- Exact units, calibration and update behavior of `FNS0` and `FNS1`; the live correlation establishes the query/field relationship but not that the returned words are calibrated RPM.
- The runtime values and transition conditions of `FTVL` for Balance, Performance and LID states.
- Complete live mapping of profile selection, including the user-visible Fn+X path, `_Q16`, `_Q40` and `_Q81` notifications, and the relationship between `GPFM` and `SPFM`.
- Recovery of the original live Fn+X/GPFM transition capture previously reported as `0x02 -> 0x01`; it is not reconstructed from memory.
- The side effects and runtime success conditions of the September `SPFM` branches using `ECMD(0x94)` and `ECMD(0x95)`.
- Why the recovered April source uses `ECMD(0x91)` and `ECMD(0x92)`, and the exact firmware/table identity of that source. The source bytes/text are recovered, but the command maps must not be merged until association is established.
- The exact `THMM` / `ALIB` parameter contract, including units and platform conditions.
- Whether the equal, fully recovered Balance/LID ALIB parameter sequences have any established relationship to EC fan-table rows; sequence equality alone does not establish table equality.
- Tachometer registers, duty/mode fields, PWM controller semantics, target-RPM arrays, temperature/index axis and profile persistence.
- A safe, model- and revision-specific fan setter, if one exists.

Register and table interpretations require the exact EC artifact, bank/address context, source bytes or disassembly, and the decoding rationale before they can establish a machine-specific fan-control contract.

## Boot-logo update behavior

The standard Insyde type-`0x54` and type-`0x6D` logo-update paths are unavailable on the tested BIOS 1.15 image: the examined type-`0x54` callback rejects the request, while the required type-`0x6D` target was not found in the retained ESRT/HFDM scope.

- Whether P916F-STX implements a separate OEM-specific logo-update mechanism outside those two paths.
- Whether any such mechanism has a provisioned target and authentication contract.
- Whether the embedded animated resource can be changed without a complete firmware update; no launchable logo-only procedure is established.

## Linux integration

- Whether the EC charge limiter can be represented through a future or nonstandard Linux interface without conflating it with the generic power-supply threshold ABI.
- Whether `platform_profile` or another standard Linux thermal/profile interface is exposed on a later or different software environment; no retained capture establishes current support or absence.
- Which kernel, ALSA and installed WirePlumber package versions affect the observed interfaces; current conclusions are scoped to the retained environment.
- Panel model/EDID/refresh and AC-adapter identity/rating. The installed SSD model `YMTC PC41Q-1TB-B` is recovered for the investigated unit, so storage model is no longer part of this missing-inventory set.

## Audio DSP and physical topology

The recovered S15 `NahimicExport.zip` closes the earlier blanket gap for application-level Nahimic preset data. It provides selected per-family preset names, ten-band EQ JSON tables and Windows endpoint/APO integration evidence for the Realtek ALC256-class endpoint. The following deeper questions remain:

- Which Nahimic profile family and dynamic processing state were active during each historical Windows/Linux listening comparison.
- Complete runtime Nahimic/A-Volute DSP graph beyond the recovered application-level EQ preset files.
- Dynamic-range compression/limiting, bass enhancement, spatial processing and other algorithm parameters not represented by the recovered preset JSON.
- Channel-specific gain/delay/crossover and amplifier-specific parameters, if any.
- The complete Windows endpoint APO/property set and whether an OEM driver or service supplies additional amplifier tuning outside the recovered standard APO/application state.
- Whether the reported four physical speaker drivers are confirmed by an OEM specification or physical inspection; logical Linux `FL`/`FR` channels do not prove physical driver count.
- How the physical drivers, amplifiers and any crossover/DSP are grouped behind the stereo host endpoint.

ALC256 enumeration, working playback, the logical stereo endpoint, Nahimic/A-Volute endpoint integration and application-level EQ tables are now recorded evidence. The thinner Linux sound and its reproduction in an Ubuntu live environment support a bounded processing/tuning interpretation, not sole-cause proof or a complete recovered OEM signal chain.

## Firmware-version portability and platform inventory

- Compatibility of the documented internal addresses and PMC2 protocol with future BIOS/EC revisions.
- Whether the firmware-reported `EC 1.15` and the internal `IT557x V1.09` image string describe components that change independently in later packages.
- Exact kernel, ALSA and installed WirePlumber package versions, panel/EDID and adapter inventory, and standard `platform_profile` exposure. S11 recovers camera/microphone interfaces, loaded audio modules and PipeWire server `1.6.7`; another exact-machine capture now recovers the installed SSD model.
- Whether any result from sibling names such as `P916F-HPT-R` or `P916F-ARL` transfers to `P916F-STX`; no such transfer is assumed.

## Artifact and provenance gaps

- Rehash of the exact private 32 MiB raw-ROM object and 128 KiB EC carve when raw-byte export is available. The source objects are located; this is now a re-verification gap, not a source-discovery gap.
- SHA-256 identity of the located `P916F-charge-reverse.tar.gz` parent. Its exact private Library object and size are known, but raw bytes were not exportable in the current audit.
- Original outer `STX_SKU2_1.15.zip` size and SHA-256. The nested EXE identity cannot substitute for its parent archive.
