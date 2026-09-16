# Audio subsystem

This document records the internal audio path, Linux-visible topology, cross-platform behavior and the current limit of what has been established about the OEM DSP stack.

## 1. Codec path

Live ALSA probing identified the internal analog codec as:

```text
Realtek ALC256 Analog
```

The codec is exposed through the AMD/Ryzen audio stack. Basic playback works correctly enough to establish that the internal analog path is detected and usable under Linux.

## 2. Logical speaker topology under Linux

PipeWire/WirePlumber exposes the internal speaker endpoint as a stereo sink with the logical channels:

```text
output_FL
output_FR
```

A representative sink identifier seen during the investigation was similar to:

```text
alsa_output.pci-0000_c1_00.6.HiFi__Speaker__sink
```

The PCI identifier is not treated as stable API because numbering can change with firmware/kernel enumeration.

No separate logical channel was exposed for:

```text
LFE
2.1
4.0
separate subwoofer
```

## 3. Physical speaker layout versus logical channels

The documented chassis has four physical speaker drivers, two per side.

Linux nevertheless exposes the internal speaker path as one stereo FL/FR endpoint. This is technically plausible: the physical drivers can be grouped behind amplifier/crossover/DSP logic while the host codec presents only two logical channels.

Therefore:

```text
four physical speakers != four host-visible audio channels
```

and the absence of a Linux-visible LFE/4.0 endpoint is not, by itself, evidence that two physical drivers are unused.

## 4. Windows OEM processing

The Windows OEM software stack includes Nahimic / A-Volute processing.

The Windows sound character is materially different from the unprocessed Linux path, particularly in:

- perceived bass/body;
- loudness;
- tonal balance;
- spatial/enhancement processing.

This indicates that the OEM Windows experience depends on software processing beyond simple ALC256 codec enablement.

## 5. Linux behavior

Linux playback is functional, but the internal speakers sound substantially thinner/weaker than under the OEM Windows stack.

The same overall behavior was reproduced in an Ubuntu live environment. That cross-check is technically useful because it reduces the likelihood that the difference is caused only by one CachyOS user configuration, one PipeWire state directory or one EasyEffects preset.

The current evidence therefore supports:

```text
codec enumeration       -> working
speaker playback         -> working
logical stereo endpoint  -> working
OEM-equivalent tuning    -> missing/not recovered
```

## 6. PipeWire / ALSA observations

The investigated Linux stack used the ordinary ALSA -> PipeWire/WirePlumber path.

The key hardware-level result is not the exact user-space configuration, but the exposed channel model:

```text
FL / FR only
```

No evidence was found that the machine requires a user-visible 4-channel ALSA profile in order to drive all physical speaker elements.

## 7. Software processing experiments

Software EQ / processing through EasyEffects and ordinary Linux audio configuration can change the sound substantially, but no tested profile reproduced the OEM Windows/Nahimic result exactly.

Earlier troubleshooting also touched HDA/ACP/SOF-related module options. Those experiments did not establish that the laptop fundamentally requires a different base driver path: the codec and speakers already function, while the persistent gap is primarily in tuning/processing quality.

Accordingly, this repository does not document "disable SOF" or similar options as a platform requirement.

## 8. Why the current conclusion points to DSP/tuning

The evidence chain is:

1. Linux identifies the ALC256 analog path.
2. Internal-speaker playback works.
3. PipeWire exposes a coherent stereo endpoint.
4. The degraded sound reproduces outside the main CachyOS installation.
5. Windows uses a known OEM enhancement stack: Nahimic / A-Volute.
6. No separate missing Linux LFE/4.0 endpoint has been observed.

The strongest current interpretation is therefore that Linux lacks the OEM endpoint processing / equalization / speaker-compensation profile used by Windows.

This remains a technical interpretation rather than a recovered coefficient-level proof because the exact OEM DSP parameters have not yet been extracted.

## 9. What remains unknown

The following have not yet been recovered:

- exact Nahimic/A-Volute EQ coefficients for this speaker endpoint;
- dynamic-range compression parameters;
- bass enhancement parameters;
- channel-specific gain/delay/crossover behavior, if any;
- endpoint APO property set used by the OEM image;
- whether any additional amplifier-specific tuning is programmed by a Windows driver/service outside the standard APO chain.

Until those are recovered, a Linux EQ can approximate the Windows sound but cannot be described as an exact reproduction.

## 10. Current technical model

```text
Physical layout:
  4 speaker drivers
  2 per side

Linux logical topology:
  stereo FL / FR

Codec:
  Realtek ALC256 Analog

Windows enhancement:
  Nahimic / A-Volute

Linux status:
  functional playback
  OEM DSP profile not recovered
```

Future useful work is therefore centered on extracting or reconstructing the Windows endpoint processing rather than searching for an imaginary mandatory Linux 4.0 speaker profile.
