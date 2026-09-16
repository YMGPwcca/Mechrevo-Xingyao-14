# Hardware and platform reference

This reference describes the investigated MECHREVO Xingyao 14 / `P916F-STX` unit and the hardware context needed to interpret the firmware and Linux findings. A value recorded on this unit is not a specification for every Xingyao 14 or every product using the P916F name. The principal machine records are retained in [S1 / `SRC-BASELINE`](research-sources.md#project-sources); the source register separates validation class from source coverage.

## Platform identity

| Component | Recorded value | Evidence class | Source coverage and boundary |
|---|---|---|---|
| Product | MECHREVO Xingyao 14 / 机械革命 星耀14 | Live-confirmed | Retained machine-identification report |
| Board/platform | `P916F-STX` | Live-confirmed | Retained machine-identification report |
| Full platform string | `MECHREVO XINGYAO Series-P916F-STX` | Live-confirmed | Recorded firmware/OS identity |
| Processor | AMD Ryzen AI 9 365 | Comparative only | Canonical model attribution retained from the baseline platform specification; not independently established by the stress-worker count |
| Processor family | AMD Strix Point / Ryzen AI 300 | Comparative only | Model-family attribution; not a substitute for the exact machine identity |
| Processor topology | 10 cores / 20 threads | Comparative only | Model specification; a 20-worker stress invocation is not independent topology evidence |
| Integrated graphics | Radeon 880M | Live-confirmed | Platform and Linux observation |
| Installed memory | 32 GiB | Live-confirmed | Observed configuration of the documented unit |
| Internal display | 2880 × 1800 | Live-confirmed | Display observation; refresh rate and EDID are not retained at equivalent coverage |
| System firmware | UEFI / Insyde H2O | Static-confirmed | Firmware analysis and recorded environment |
| System BIOS | `1.15` | Live-confirmed | Firmware UI observation |
| EC version reported by firmware | `1.15` | Live-confirmed | Firmware UI observation |
| BIOS build-date string | `05/07/2026` | Live-confirmed | Raw display string; date convention is not assumed |
| EC silicon | ITE `0x5571`, revision `0x07` | Live-confirmed | Super-I/O configuration read at `0x4E` |
| Internal analog codec | Realtek ALC256 | Live-confirmed | ALSA enumeration |
| Battery object | `LCBT` | Live-confirmed | Linux power-supply enumeration |
| AC adapter object | `ACAD` | Live-confirmed | Linux power-supply enumeration |
| Battery model string | `588974-3S-G-A0` | Live-confirmed | Recorded Linux model string |
| Physical speaker layout | Four drivers, two per side | Not established | Reported hardware description; no retained product specification or physical inspection |

The BIOS date remains `05/07/2026`; no date-format conversion is asserted.

The processor identity used throughout this repository is **AMD Ryzen AI 9 365**. The baseline stress output records 20 CPU workers because that was the selected workload, not because the worker count independently measured 10 physical cores and 20 logical CPUs.

## Firmware version identifiers

The investigation encountered several independent version identifiers:

```text
System BIOS:             1.15
Firmware-reported EC:    1.15
Internal EC build:       IT557x V1.09 E00 - 20230831
Additional EC strings:   ITE EC-V14.6
                         VER:01.0F.00
```

These strings belong to different build or reporting namespaces. `IT557x V1.09` does not identify the system BIOS as version 1.09 and does not contradict the firmware UI's `EC 1.15` label. A separately retained BIOS 1.09 updater is historical package evidence, not evidence that every internal EC string changes with each BIOS release. Keeping these namespaces separate matters when comparing update packages, raw flash images and internal EC firmware revisions.

Exact kernel, graphics-stack and firmware-tool versions were not reconstructed from unrelated installations. Missing environment records are tracked under [P15 — additional platform inventory and exact versions](documentation-status.md#pending-evidence).

## CPU / graphics platform

The recorded platform characteristics are:

```text
CPU:      AMD Ryzen AI 9 365
family:   AMD Strix Point / Ryzen AI 300
cores:    10
threads:  20
iGPU:    Radeon 880M
```

The integrated Radeon path used the ordinary AMD Linux graphics stack, and Wayland operation was reported on the documented machine. No P916F-specific graphics-firmware replacement or override was established as a requirement. This is not a certification of every graphics API, external output, suspend state or future kernel.

The retained internal-panel resolution is:

```text
2880 × 1800
```

No refresh-rate value, panel model, EDID digest or adapter rating is published here because the investigation did not retain a correctly identified capture with equivalent evidence quality. These are part of [P15](documentation-status.md#pending-evidence), not values to infer from a related laptop.

## Internal display
### BGRT placement

After the recorded BIOS 1.15 update, Linux exposed the following ACPI BGRT metadata:

```text
status  = 0
type    = 0
version = 1
xoffset = 1040
yoffset = 387
```

Static firmware analysis identified an 800 × 600 OEM boot-animation resource. Its horizontal geometry agrees with the BGRT placement:

```text
1040 + 800 + 1040 = 2880
```

This is a geometric cross-check between live BGRT metadata and the statically identified resource. It does not prove that BGRT itself contains the animation or provide a firmware-update interface. Resource details and the separate logo-update routes are documented in [`firmware-bios.md`](firmware-bios.md) and [`boot-logo-research.md`](boot-logo-research.md).

## Embedded controller

The recorded ITE Super-I/O probe found:

```text
chip ID  = 0x5571
revision = 0x07
config   = 0x4E
```

The EC firmware is an IT557x MCS-51/8051-family image embedded in the platform flash. The active PMC2 logical device is:

```text
LDN             = 0x12
DATA            = 0x68
COMMAND/STATUS  = 0x6C
```

This PMC2 transport is central to the documented battery charge-limit work. The preferred raw-ROM EC carve, bank/address qualifications, H2RAM mapping and code landmarks are maintained in [`embedded-controller.md`](embedded-controller.md); threshold behavior and the retained transaction results are in [`battery-charge-limit.md`](battery-charge-limit.md).

Recovered AML also defines two 16-bit fan-telemetry fields and one profile field:

```text
FNS0  16-bit field at EC field offset 0x3B
FNS1  16-bit field at EC field offset 0x3D
FTVL   8-bit field at EC field offset 0x3F
```

These establish a firmware-visible channel model. They do not independently establish the mechanical fan count, RPM calibration, a complete target-RPM table or a manually controllable fan interface. The thermal method paths and their source boundaries are documented in [`thermal-performance.md`](thermal-performance.md).

## Battery and adapter

Linux exposed the following power-supply objects:

```text
Battery: /sys/class/power_supply/LCBT
AC:      /sys/class/power_supply/ACAD
```

The observed battery model string was:

```text
588974-3S-G-A0
```

Recorded telemetry included:

```text
capacity
status
voltage_now
power_now
energy_now
model_name
```

The observed `LCBT` device did not expose the usual generic threshold attributes:

```text
charge_control_start_threshold
charge_control_end_threshold
charge_behaviour
```

The firmware nevertheless contains a charge-limit subsystem reached through the EC PMC2 command family. Only the enabled `T1=80%`, `T2=100%` pair has been behaviorally validated; the exact user-facing meaning of `T2` and the behavior of other pairs remain open. See [`linux.md`](linux.md) and [`battery-charge-limit.md`](battery-charge-limit.md).

## Audio hardware

ALSA identified the internal analog path as:

```text
Realtek ALC256 Analog
```

The baseline narrative reports four physical speaker drivers, two per side. That count is retained as a reported description, not as a vendor statement or a Live-confirmed teardown result. Linux exposed one stereo endpoint:

```text
output_FL
output_FR
```

No separate LFE, 2.1 or 4.0 endpoint was observed. Logical FL/FR enumeration does not prove that additional physical drivers are absent or inactive, and it does not independently verify the reported four-driver layout. The Windows OEM stack was reported to use Nahimic / A-Volute processing; the equivalent parameters have not been recovered. See [`audio.md`](audio.md) and [P14 — physical audio topology and OEM tuning](documentation-status.md#pending-evidence).

## UEFI / Insyde firmware environment

The platform uses Insyde H2O firmware. Relevant components identified in the BIOS 1.15 image include:

```text
SetupUtility
OemBadgingSupportDxe
BootGraphicsResourceTableDxe
ChipsetSvcSmm
Insyde H2OFFT / IHISI update infrastructure
```

The image also contains hidden setup forms and OEM-specific resources. A form or generic Insyde mechanism is not treated as enabled on P916F-STX merely because it exists in another Insyde implementation; each applicable mechanism must be traced into this exact firmware. The retained setup identifiers, Quiet Boot observation and visibility boundaries are documented in [`firmware-bios.md`](firmware-bios.md).

## ACPI integration

The platform exposes a standard ACPI embedded-controller device:

```text
_HID = PNP0C09
GPE  = 0x0B
```

The DSDT separately declares a SystemMemory-backed EC window:

```asl
OperationRegion (ERAM, SystemMemory, 0xFEEC2300, 0x100)
```

The exact IT5571 firmware analysis maps the host-visible window to:

```text
EC XRAM 0x0300..0x03FF
```

Thus, for an offset `N` within the declared 256-byte aperture:

```text
host 0xFEEC2300 + N  <->  EC XRAM 0x0300 + N
```
This H2RAM aperture is distinct from the conventional ACPI EC I/O interface and from the ITE PMC2 command channel. For example, `XRAM[0x0394]` corresponds to host physical `0xFEEC2394` and is used as an SOC value by the charge-control decision logic. The public [`acpi-wmi.md`](acpi-wmi.md), [`thermal-performance.md`](thermal-performance.md) and [`research-sources.md`](research-sources.md#project-sources) pages preserve the relevant AML method details and source attribution.

## Platform-family caution

Other firmware targets encountered under the broader P916F naming family include:

```text
P916F-HPT-R
P916F-ARL
```

Their existence does not imply compatibility with `P916F-STX`. BIOS images, EC binaries, GPIO assumptions, flash layouts and EC register maps remain model-specific until validated directly. Generic Tongfang/Uniwill/MECHREVO EC knowledge is useful for orientation only and cannot replace machine-specific evidence.

## Pending evidence gates

The following boundaries are deliberate:

| Gate | Current state | Material needed |
|---|---|---|
| [P02 — complete ACPI-table identity](documentation-status.md#pending-evidence) | `NEEDS_EVIDENCE` | Original complete ACPI table/export header and digest, if available; excerpts do not establish a full-table identity |
| [P14 — physical audio topology and OEM tuning](documentation-status.md#pending-evidence) | `NEEDS_EVIDENCE` | Product specification or physical inspection for driver count, plus recovered OEM tuning evidence if available |
| [P15 — additional platform inventory and exact versions](documentation-status.md#pending-evidence) | `NEEDS_EVIDENCE` | Correct-machine capture for camera/microphone/modules and storage identifiers, kernel and audio-stack versions, panel identity/refresh/EDID and adapter details |

No claim above depends on running a firmware writer, issuing an EC setter, or importing hardware data from another machine.
