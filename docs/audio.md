# Audio subsystem

This reference records the internal codec path, Linux-visible topology and the cross-platform behavior observed on the investigated MECHREVO Xingyao 14 / `P916F-STX`. The baseline report is [S1 / `SRC-BASELINE`](research-sources.md#project-sources). It contains no calibrated acoustic measurements, complete amplifier schematic or recovered OEM DSP coefficients. The reported four-driver layout therefore remains distinct from the directly observed logical audio interfaces.

## 1. Codec path

Live ALSA probing identified the internal analog path as:

```text
Realtek ALC256 Analog
```

The codec is exposed through the AMD/Ryzen audio stack. Basic playback functioned under Linux, establishing that the internal analog path was detected and usable.

## 2. Logical speaker topology under Linux

PipeWire/WirePlumber exposed the internal speaker endpoint with these logical channels:

```text
output_FL
output_FR
```

A representative sink identifier in the baseline report was:

```text
alsa_output.pci-0000_c1_00.6.HiFi__Speaker__sink
```

The PCI-derived identifier is an observed enumeration string, not a stable API across firmware or kernel enumeration changes.

No separate logical endpoint was observed for:

```text
LFE
2.1
4.0
separate subwoofer
```

This is a statement about the exposed audio interfaces. It does not establish the number or activity of physical drivers inside the chassis.

## 3. Physical speaker layout versus logical channels

The baseline narrative reports four physical speaker drivers, two per side. The original product specification or a physical-inspection record supporting that count has not been recovered. The count is therefore retained as a **reported hardware description**, not classified as Live-confirmed and not attributed to a vendor statement.

A stereo host interface can feed more than two physical drivers through amplifier, crossover or DSP logic. Conversely, `output_FL` and `output_FR` do not verify that additional drivers exist or are active. The correct distinction is:

```text
reported physical layout  !=  host-visible channel count
four reported drivers      !=  four host-visible audio channels
```

The absence of an LFE or 4.0 endpoint is not evidence that two physical drivers are unused. Logical stereo enumeration also cannot independently validate the reported four-driver layout. Closing that question requires [P14 — physical audio topology and OEM tuning](documentation-status.md#pending-evidence).

## 4. Windows OEM processing

The Windows OEM software stack was reported to include Nahimic / A-Volute processing. Compared with the investigated Linux configuration, the perceived differences concerned:

- bass and overall body;
- loudness;
- tonal balance;
- spatial or other enhancement processing.

These are comparative listening observations, not calibrated frequency-response, distortion or loudness measurements. The retained record does not support reconstructing their exact magnitude.

The OEM processing names identify the reported software stack, but they do not by themselves prove which coefficients, endpoint APO properties or amplifier programming were applied to this unit.

## 5. Linux behavior

Playback was functional in the principal CachyOS installation, but the internal speakers were perceived as thinner and weaker than under the Windows OEM result. The same overall deficit was reported in an Ubuntu live environment.

The second Linux environment reduces the likelihood that the difference was caused solely by one CachyOS user configuration, one PipeWire state directory or one EasyEffects preset. It does not isolate the contribution of codec programming, amplifier settings, endpoint processing or application-level enhancement.

| Layer | Recorded result | Evidence class | Source coverage and limit |
|---|---|---|---|
| Codec enumeration | ALC256 analog path present | Live-confirmed | ALSA observation; does not characterize every pin or amplifier |
| Speaker playback | Functional under Linux | Live-confirmed | Recorded playback observation; no calibrated acoustic characterization |
| Host channel model | Stereo FL/FR | Live-confirmed | PipeWire/WirePlumber enumeration; does not establish physical driver count |
| Cross-installation comparison | Similar perceived deficit in Ubuntu live environment | Comparative only | Retained listening comparison; no controlled acoustic measurement |
| OEM-equivalent processing | Not recovered | Not established | No exact coefficients or complete Windows endpoint configuration |

The comparison preserves playback findings without converting a subjective deficit into a hardware-topology claim.

## 6. PipeWire / ALSA observations

The investigated Linux stack used the ordinary ALSA → PipeWire/WirePlumber path. The hardware-level result is the exposed channel model:

```text
FL / FR only
```

No retained evidence establishes that the machine requires a user-visible four-channel ALSA profile to drive all physical speaker elements. A forced 4.0 profile is therefore not documented as a verified remedy.

### Recovered device and kernel capture

Source [S11](research-sources.md#project-sources), `Pasted text(71).txt`, records the following Linux interfaces. These are observations from that capture, not a new hardware test or a universal device configuration.

| Layer | Captured observation | Boundary |
|---|---|---|
| PipeWire | Server `pipewire-0` reports `1.6.7`; the listed clients also display `1.6.7` | Client display strings do not independently identify installed WirePlumber package versions |
| Internal sink | `Ryzen HD Audio Controller Speaker`, selected sink, volume `1.00` | Snapshot state, not calibrated gain |
| Capture endpoints | `Ryzen HD Audio Controller Digital Microphone`; `Ryzen HD Audio Controller Stereo Microphone` | Logical endpoints, not physical microphone count |
| Capture state | Digital endpoint volume `1.00`; selected stereo endpoint volume `0.32 MUTED` | Capture-time state only |
| Active monitor inputs | `ALC256 Analog:capture_FL`, `capture_FR`; `Digital Microphone:capture_FL`, `capture_FR` | Both paths appear active in PulseAudio Volume Control streams; not an acoustic quality test |
| ALSA cards | `0 Generic`, `1 Generic_1`: `HD-Audio Generic`; `2 acppdmmach`: `acp-pdm-mach` | Card numbers are enumeration-local |
| ACP machine identity | `MECHREVO-XINGYAOSeries-Standard-XINGYAOSeries_P916F_STX` | Identifies this capture's reported platform, not BIOS revision |
| Playback | Card 0 HDMI devices `3`, `7`, `8`, `9`; card 1 device `0`, `ALC256 Analog`; each reports one subdevice available out of one | Enumeration, not a playback test of all HDMI outputs |
| Radeon HDA PCI | `c1:00.1`, `1002:1640`, driver in use `snd_hda_intel` | Subsystem `1d05:e004` |
| AMD audio coprocessor PCI | `c1:00.5`, `1022:15e2`, revision `70`, driver in use `snd_acp_pci` | Subsystem `1d05:e004`; listed candidate modules are not all active drivers |
| Ryzen HDA PCI | `c1:00.6`, `1022:15e3`, driver in use `snd_hda_intel` | Subsystem `1d05:e004` |

The ALSA HDA card resources are recorded as `0xb04c8000`, IRQ `127`, and `0xb04c0000`, IRQ `128`. They are observed host resources, not EC addresses or portable constants. The PCI listing also records a permission error reading `/sys/bus/pci/devices/0000:00:08.3/label`; the capture is not represented as an error-free complete PCI inventory.

The recorded kernel log shows `snd_hda_intel` applying patch firmware named `xingyao.fw` to both `0000:c1:00.1` and `0000:c1:00.6`. The message payloads are:

```text
snd_hda_intel 0000:c1:00.1: Applying patch firmware 'xingyao.fw'
snd_hda_intel 0000:c1:00.6: Applying patch firmware 'xingyao.fw'
```

Only the message payloads are excerpted; timestamps and host identifiers are omitted. The source does not include the patch bytes, digest, provenance or contents. This observation does not identify what the patch changes, establish Nahimic processing or speaker tuning, or show that it is required on other kernels. The log also records `c1:00.1` bound to `c1:00.0` through `amdgpu_dm_audio_component_bind_ops [amdgpu]`.

The captured codec message payloads are:

```text
ALC256: SKU not ready 0x50f00010
autoconfig for ALC256: line_outs=2 (0x1b/0x14/0x0/0x0/0x0) type:speaker
   speaker_outs=0 (0x0/0x0/0x0/0x0/0x0)
   hp_outs=1 (0x21/0x0/0x0/0x0/0x0)
   mono: mono_out=0x0
   inputs:
     Mic=0x12
```

These are codec autoconfiguration fields, not physical speaker counts. In particular, `speaker_outs=0` does not contradict the separate `line_outs=2 ... type:speaker` report or prove that speakers are absent. The kernel creates HDMI/DP input entries for PCM `3`, `7`, `8`, `9` and an internal headphone entry. Full loaded-module coverage is maintained in [Linux inventory](linux.md#recorded-audio-module-inventory).

## 7. Software processing experiments

Software EQ and processing experiments through EasyEffects and ordinary Linux audio configuration changed the sound, but no tested profile reproduced the Windows OEM/Nahimic result exactly. The complete preset and measurement set were not retained; this page does not invent a replacement preset or channel map.

Earlier troubleshooting also considered HDA/ACP/SOF-related module options. Those experiments did not establish a different base-driver path as a platform requirement: the codec and speakers already function, while the persistent difference is in tuning and processing quality. This repository therefore does not prescribe disabling SOF or another module option as a verified fix.

A firmware setup menu may contain codec verb-table choices. Their static presence or default is not independent proof of the codec, amplifier topology or appropriate tuning for this unit.

## 8. Why the current conclusion points to DSP/tuning

The available observations are consistent with a missing or different OEM endpoint-processing, equalization or speaker-compensation configuration under Linux. This is an **Inferred** interpretation, not coefficient-level proof. The exact Nahimic/A-Volute parameters have not been recovered, and additional Windows driver/service programming of an amplifier has not been excluded.

The evidence does not support describing the issue simply as undetected speakers. It also does not establish DSP or EQ as the sole cause. Logical FL/FR, physical driver allocation and OEM processing are separate layers.

## 9. What remains unknown

The following remain unresolved:

- exact Nahimic/A-Volute EQ coefficients for this endpoint;
- dynamic-range compression parameters;
- bass-enhancement parameters;
- channel-specific gain, delay or crossover behavior, if any;
- endpoint APO properties used by the OEM image;
- additional amplifier-specific tuning programmed by a Windows driver or service outside the standard APO chain;
- physical driver count and wiring/topology, requiring primary inspection evidence;
- exact kernel, HDA/ACP/SOF and ALSA/WirePlumber package versions; endpoint names and loaded module names are recovered in S11.

The recovered endpoint and kernel observations do not establish OEM-equivalent processing. Physical topology remains P14; the unresolved environment and standard-profile portions of P15 are tracked separately in [documentation status](documentation-status.md#pending-evidence).

## 10. Current technical model

```text
Physical layout:
  Four speaker drivers reported, two per side
  Physical count: Not established independently

Linux logical topology:
  Stereo FL / FR

Codec:
  Realtek ALC256 Analog

Windows enhancement:
  Nahimic / A-Volute (reported OEM stack)

Linux status:
  Functional playback
  OEM DSP profile not recovered
```

The next useful evidence is a source-attributed physical inspection or product specification and a recovered Windows endpoint-processing configuration—not an assumed mandatory Linux 4.0 profile. These requirements are tracked in [P14 — physical audio topology and OEM tuning](documentation-status.md#pending-evidence) and [P15 — additional platform inventory and exact versions](documentation-status.md#pending-evidence).
