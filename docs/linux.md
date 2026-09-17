# Linux platform integration

This reference records Linux-visible interfaces and platform behavior for the investigated MECHREVO Xingyao 14 / `P916F-STX`. It covers hardware/firmware integration, not desktop-environment configuration or a shipped driver. The principal live records are from CachyOS / an Arch-family installation on the documented unit; an Ubuntu live environment was used for an audio comparison. Device-availability statements are scoped to those recorded environments, not to every kernel or distribution. Sources are [S1 / `SRC-BASELINE`](research-sources.md#project-sources), the identified ACPI excerpts [`SRC-AML-A`](research-sources.md#project-sources), [`SRC-AML-B`](research-sources.md#project-sources), [`SRC-AML-C`](research-sources.md#project-sources) and [`SRC-AML-D`](research-sources.md#project-sources), and the general Linux interface definitions in [E1](research-sources.md#external-interface-and-licensing-references).

Source S11 records PipeWire server `1.6.7` and a loaded audio-module inventory. Exact kernel, ALSA and installed WirePlumber package versions remain unestablished; the version displayed beside a PipeWire client is not independently a package-version query. See [P15](documentation-status.md#pending-evidence).

## Platform identity

```text
MECHREVO XINGYAO Series-P916F-STX
AMD Ryzen AI 9 365
AMD Radeon 880M
32 GiB memory on the documented unit
2880 × 1800 internal display
```

The Radeon 880M operated through the ordinary AMD Linux graphics stack, and Wayland operation was reported in the principal environment. This observation is not a certification of every graphics API, external output, suspend state or future kernel.

## Power-supply exposure

Linux exposed the following power-supply devices:

| Device | Recorded path |
|---|---|
| Battery | `/sys/class/power_supply/LCBT` |
| AC adapter | `/sys/class/power_supply/ACAD` |

The battery model string was:

```text
/sys/class/power_supply/LCBT/model_name
588974-3S-G-A0
```

The inspected `LCBT` device exposed these attributes:

| Attribute | Interpretation | Unit or representation |
|---|---|---|
| `capacity` | Reported state of charge | Integer percent |
| `status` | Charging state | Text, including `Charging`, `Discharging` and `Not charging` |
| `voltage_now` | Reported battery voltage | µV |
| `power_now` | Reported battery power | µW; direction depends on the driver's reporting behavior |
| `energy_now` | Reported stored energy | µWh |
| `model_name` | Device model identifier | Text |
| `ACAD/online` | Reported adapter presence | Integer state; `1` was observed with AC connected |

These are Linux power-supply conventions, not a guarantee of measurement accuracy or a complete electrical model. `current_now` was absent from the inspected `LCBT` tree.

A representative discharging state was:

```text
89% - Discharging
```

A representative AC-online capped snapshot was:

```text
capacity   = 79%
status     = Not charging
power_now  = 0
energy_now = 63154000
ACAD       = online=1
```

`63154000 µWh = 63.154 Wh`. Stable capped samples reported `power_now=0`; a zero value at one sample does not measure adapter power and does not establish zero battery current for an interval. The complete retained measurements are in [`validation.md`](validation.md).

The device also lacked these generic threshold attributes:

```text
charge_control_start_threshold
charge_control_end_threshold
charge_behaviour
```

Their absence means that the P916F charge limiter was not surfaced through the generic Linux power-supply threshold ABI in the inspected environment. It does not mean that the firmware feature is absent.

The recovered S3 text includes a separate original sysfs snapshot. Exact output lines are retained below; the shell prompt and host identity are omitted:

```text
capacity                             95
status                               Discharging
voltage_now                          13240000
power_now                            750000
```

The voltage and power values are in µV and µW. The captured command conditionally printed only existing attributes from its requested list; `current_now` and the three generic threshold/behaviour attributes produced no lines and are also absent from the displayed LCBT listing. This supports absence in that capture, not every kernel. The directory listing additionally includes `alarm`, `capacity_level`, `cycle_count`, `energy_full`, `energy_full_design`, `energy_now`, `manufacturer`, `model_name`, `present`, `serial_number`, `technology`, `type`, `voltage_min_design`, `extensions`, `hwmon2` and `power`; the names do not disclose the unqueried values. The device link targets `PNP0C0A:00`. No serial value is published.

The S3 Huawei-WMI search output lists driver bind/unbind/uevent, driver override, modalias, power-management and subsystem probing attributes, but no battery-threshold attribute. It is a recorded sysfs inspection, not a new WMI command invocation or proof that all OEM WMI functions are unavailable.

## Firmware charge control

The machine-specific charge limiter was instead reached through the ITE PMC2 transport. The retained validated state was:

```text
state = 1
T1    = 80
T2    = 100
```

With this pair, Linux recorded charging below the configured region and a transition to `Not charging` around displayed 79–80%. Stable capped samples reported `power_now=0`, and the state and thresholds were read back after a normal reboot. Only this pair has been behaviorally validated; the exact user-facing role of `T2`, the hysteresis width and the behavior of other threshold pairs remain unresolved. The command framing, field map, response handling and retained raw results are documented in [`battery-charge-limit.md`](battery-charge-limit.md).

### Energy observation and timing

The high-load observation ran with AC connected and retained these snapshots:

```text
20:47:14  cap=79%  status=Not charging  power=0       energy=63154000
20:52:27  cap=78%  status=Charging      power=28128000 energy=62661000
```

The reported energy decrease is:

```text
(63154000 - 62661000) µWh = 493000 µWh = 0.493 Wh
```

The timestamped snapshots span 313 seconds. The `stress-ng` record reports a five-minute run, i.e. 300 seconds of configured stressor runtime. These are different intervals and must not be substituted for one another in an average-power calculation. The result supports net battery-energy contribution during the observation and is consistent with battery-assist behavior; it does not establish adapter wattage, instantaneous adapter draw or the complete charger topology.

The stress invocation selected 20 CPU workers. That workload setting is not independent evidence of the processor's physical/logical topology; the canonical processor identity and topology boundary are documented in [`hardware-platform.md`](hardware-platform.md).

## Distinct EC-facing mechanisms

Three interfaces appear in the Linux/firmware analysis and must not be conflated:

| Mechanism | Host location | Documented role | Evidence class | Source coverage |
|---|---|---|---|---|
| Standard ACPI EC | I/O `0x62` / `0x66` | Conventional ACPI EC transactions for AML EC fields | Static-confirmed | Selected AML/ACPI excerpts |
| H2RAM | Physical `0xFEEC2300..0xFEEC23FF` | SystemMemory aperture backed by EC shared state | Static-confirmed; live-correlated | Selected DSDT and retained live correlation |
| ITE PMC2 | Data `0x68`; command/status `0x6C` | Working host transport for the recovered charge-limit protocol | Live-confirmed | Retained live probe/transaction report |

The DSDT declaration for H2RAM is:

```asl
OperationRegion (ERAM, SystemMemory, 0xFEEC2300, 0x100)
```

Static analysis of the exact IT5571 firmware maps it to EC XRAM `0x0300..0x03FF`:

```text
host 0xFEEC2300 + N  <->  EC XRAM 0x0300 + N
```

For example:

```text
EC XRAM 0x0394
host    0xFEEC2394
```

`XRAM[0x0394]` is used as an SOC/battery-percentage input by the charge-control decision logic. The charge-threshold fields `XRAM[0x0D13]` and `XRAM[0x0D14]` are outside this 256-byte H2RAM aperture; they are not extensions of the same host window.

The H2RAM aperture is not the conventional ACPI EC transaction path and is not the PMC2 command channel. The mapping and field details are maintained in the public [`acpi-wmi.md`](acpi-wmi.md), [`embedded-controller.md`](embedded-controller.md) and [`research-sources.md`](research-sources.md#project-sources) pages.

## ACPI and WMI integration

The battery object is `LCBT` and provides the standard ACPI methods:

```text
_BIX
_BST
_BTP
```

`_BTP` writes ACPI battery trip-point fields. It is not the P916F charge-limit implementation at `XRAM[0x0D13]` and `XRAM[0x0D14]`.

The ACPI lid path is separate from operating-system lid policy. The firmware lid device uses EC-backed state and issues notifications; the setup option is named `Dynamic LID` / `AmdDynamicLid`. Its presence does not establish open-lid power-on behavior. `systemd-logind` can independently select what the OS does after a lid-close event without changing the firmware signal.

Recovered OEM AML also exposes methods for fan telemetry and thermal profiles, including `GFNS`, `GVER`, `GPFM`, `SPFM`, `GKBT` and `SKBT`. The September source-scoped `SPFM` excerpt uses `ECMD(0x94)` and `ECMD(0x95)`; an older excerpt uses `0x91` and `0x92` but lacks complete firmware identity. These maps must not be combined. Static method presence does not provide retained live validation of every setter. See [`thermal-performance.md`](thermal-performance.md) for the source-level control-flow boundary.

The `WMAA` wrapper returns a two-element package, not an unconditional flat 256-byte result:

```text
WMAA return = Package(2)
  element 0 = Buffer(0x04)
  element 1 = Buffer(0x0100)  (method result buffer)
```

The direct return shape and the method-specific helper buffer are separate layers. The public [`acpi-wmi.md`](acpi-wmi.md) page preserves the dispatch contract, and [`research-sources.md`](research-sources.md#project-sources) records the source attribution. The helper's response status is represented by response buffer byte 0; a returned `0x01` is therefore read as `response buffer byte 0 is 0x01`, not as a generic WMI status assertion.

## Rejected and limited paths

A `huawei-wmi` platform device was observed, but it exposed no usable battery-control attributes. A Huawei-style threshold GET identified as method/function `0x1103` returned failure or unsupported. The write-side `0x1003` path was not attempted after the GET failed. Presence of `huawei-wmi` therefore does not imply a working Huawei threshold API, and failure of that operation does not establish that other OEM WMI methods are unavailable.

A proposed dedicated I2EC base at `0x380` failed the recorded cross-check: the read-only path returned `0xFF` while the H2RAM/MMIO path returned a valid SOC value. It is not a substitute for the working PMC2 transport.

## Firmware access boundary

The retained PSP-related line was:

```text
/sys/bus/pci/devices/0000:c1:00.2/rom_armor_enforced:1
```

This single attribute is not a complete flash-access diagnosis. Complete `flashrom` diagnostics and a full firmware-service acquisition transcript are not retained. Neither universal failure of Linux firmware reads nor unrestricted H2OFFT access follows from the line. Direct SPI access, a vendor firmware service and offline archive analysis are distinct routes; their evidence boundaries are documented in [`firmware-access.md`](firmware-access.md) and [P07–P10](documentation-status.md#pending-evidence).

No firmware writer, SPI operation or new hardware acquisition is required for this documentation revision.

## Boot graphics and display

After BIOS 1.15, Linux exposed:

```text
status  = 0
type    = 0
version = 1
xoffset = 1040
yoffset = 387
```

Static firmware analysis identified an 800 × 600 boot-animation resource. On the retained 2880-pixel-wide panel:

```text
1040 + 800 + 1040 = 2880
```

The relationship is corroborating geometry, not a logo-replacement mechanism. See [`boot-logo-research.md`](boot-logo-research.md).

## Audio and peripheral inventory

ALSA exposed:

```text
Realtek ALC256 Analog
```

PipeWire/WirePlumber exposed the internal speaker endpoint as:

```text
output_FL
output_FR
```

The baseline report describes four physical speaker drivers, but that count lacks an independently retained product specification or physical-inspection record. Stereo logical channels neither verify nor disprove that reported layout. Playback was functional in the principal CachyOS installation, while the Ubuntu live comparison reproduced the overall perceived deficit against the Windows OEM result. The exact OEM-equivalent DSP profile remains unrecovered; details are in [`audio.md`](audio.md).

Source [S11](research-sources.md#project-sources), `Pasted text(71).txt`, recovers the device/audio enumeration. **Linux exposed two V4L2 device entries named FHD Camera.** Their displayed IDs were `45` and `46`; the video source was `134`, `FHD Camera (V4L2)`. These are session-local graph IDs, not physical camera identifiers. The capture does not establish two physical cameras, sensor models, USB identities or capabilities.

The audio source names were `Ryzen HD Audio Controller Digital Microphone` and `Ryzen HD Audio Controller Stereo Microphone`. ALSA included `acp-pdm-mach` with platform identity `MECHREVO-XINGYAOSeries-Standard-XINGYAOSeries_P916F_STX`. The selected speaker, analog/digital capture paths, PCI driver bindings, codec pin messages and `xingyao.fw` observation are documented in [audio](audio.md#recovered-device-and-kernel-capture).

The graph listed a default configured sink `easyeffects_sink` and source `alsa_input.pci-0000_c1_00.6.HiFi__Mic2__source`. Configured defaults and the currently selected graph objects are different observations; neither establishes a required processing setup. The source contains no `platform_profile` inspection.

### Recorded audio module inventory

The S11 `lsmod` output lists the modules below. This inventory distinguishes loaded modules from the PCI listing's **driver in use** (`snd_acp_pci` for `c1:00.5`, `snd_hda_intel` for `c1:00.1` and `c1:00.6`). Module presence and reference counts do not prove that a module actively drives every endpoint or that its removal is safe.

| Group | Loaded module names |
|---|---|
| Sequencer/timing | `snd_seq_dummy`, `snd_hrtimer`, `snd_seq`, `snd_seq_device`, `snd_timer` |
| ACP and machine support | `snd_acp_legacy_mach`, `snd_acp_mach`, `snd_soc_nau8821`, `snd_acp3x_rn`, `snd_acp70`, `snd_acp_pdm`, `snd_acp_i2s`, `snd_soc_dmic`, `snd_acp_pcm`, `snd_acp_pci`, `snd_amd_acpi_mach`, `snd_acp_legacy_common`, `snd_acp_config` |
| SOF | `snd_sof_amd_acp70`, `snd_sof_amd_acp63`, `snd_sof_amd_vangogh`, `snd_sof_amd_rembrandt`, `snd_sof_amd_renoir`, `snd_sof_amd_acp`, `snd_sof_pci`, `snd_sof_xtensa_dsp`, `snd_sof`, `snd_sof_utils` |
| PCI alternatives / matching | `snd_pci_ps`, `snd_soc_acpi_amd_match`, `snd_soc_acpi_amd_sdca_quirks`, `snd_pci_acp6x`, `snd_pci_acp5x`, `snd_rn_pci_acp3x`, `snd_pci_acp3x` |
| SoundWire / SDCA | `soundwire_amd`, `soundwire_generic_allocation`, `snd_amd_sdw_acpi`, `soundwire_bus`, `snd_soc_sdca`, `snd_intel_sdw_acpi` |
| HDA | `snd_hda_codec_alc269`, `snd_hda_codec_realtek_lib`, `snd_hda_codec_atihdmi`, `snd_hda_scodec_component`, `snd_hda_codec_generic`, `snd_hda_codec_hdmi`, `snd_hda_intel`, `snd_hda_codec`, `snd_hda_core`, `snd_intel_dspcfg` |
| Shared ALSA / ASoC | `snd_soc_core`, `snd_pcm_dmaengine`, `snd_compress`, `snd_hwdep`, `snd_pcm`, `snd_ctl_led`, `snd`, `snd_soc_acpi` |

The dependency column names `snd_acp70` as a user of `snd_acp_pdm`, `snd_acp_i2s` and `snd_acp_pcm`; it names `snd_acp_mach` as a user of `snd_soc_nau8821`, and `snd_acp_legacy_mach` as a user of `snd_acp_mach`. These are module reference relationships, not independent proof of a physical NAU8821 codec. SOF variants and older ACP modules also appear loaded; their presence is not a recommendation to switch drivers.

## Standard profile portability

No retained sysfs inspection establishes whether this unit exposes the Linux `platform_profile` interface. The OEM `GPFM` method is not the standard Linux ABI and must not be used as evidence that `platform_profile` is present or absent. Portability of the recovered profile controls to a kernel-facing `platform_profile` implementation therefore remains unknown.

## Integration boundary

The recovered evidence is sufficient to document PMC2 ports, command framing, enable state, threshold getters/setters and the behaviorally validated 80%/100% pair. It does not define a production Linux driver contract. A native implementation would still require explicit platform/revision matching, transport ownership, response/error handling and user-facing semantics. This repository documents the interfaces and evidence rather than shipping a driver.

## Pending evidence gates

| Gate | Current state | Material needed |
|---|---|---|
| [P07 — full flashrom failure log](documentation-status.md#pending-evidence) | `NEEDS_EVIDENCE` | Original command, result, environment and complete diagnostic capture |
| [P08 — PSP protection/version fields](documentation-status.md#pending-evidence) | `NEEDS_EVIDENCE` | Original multi-attribute PSP/ROM Armor capture; do not infer fields from `rom_armor_enforced=1` |
| [P10 — protected-region map and DXE equality](documentation-status.md#pending-evidence) | `NEEDS_EVIDENCE` | Original `-pq`/comparison records or both identified source byte ranges and comparison output |
| [P14 — physical audio topology and OEM tuning](documentation-status.md#pending-evidence) | `NEEDS_EVIDENCE` | Product specification/inspection and recovered OEM processing evidence |
| [P15 — additional platform inventory and exact versions](documentation-status.md#pending-evidence) | Partially recovered | S11 supplies camera/microphone/module names and PipeWire server version; kernel/ALSA/WirePlumber package versions, storage identifiers and panel/adapter details remain missing |
| [T34 / platform profile portability](documentation-status.md#pending-evidence) | `NEEDS_EVIDENCE` | Identified `platform_profile` sysfs inspection on this unit and kernel |
