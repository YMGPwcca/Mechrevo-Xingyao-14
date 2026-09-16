# Audio notes

## Confirmed Linux codec path

Live ALSA probing on the researched MECHREVO Xingyao 14 identified:

```text
Realtek ALC256 Analog
```

The machine's audio stack is attached through the AMD/Ryzen HDA/ACP/SOF ecosystem, but the important machine-specific endpoint is that Linux successfully detects and drives the ALC256 analog speaker path.

## PipeWire/WirePlumber speaker topology

A live `wpctl status` inspection exposed the internal speaker endpoint through the Ryzen HD Audio controller and showed only the normal stereo channels:

```text
output_FL
output_FR
```

No separate Linux-visible channel was present for:

```text
LFE
2.1
4.0
separate subwoofer
```

A representative sink name from the investigated installation was similar to:

```text
alsa_output.pci-0000_c1_00.6.HiFi__Speaker__sink
```

PCI numbering is installation/kernel dependent and is included only as an example, not as a stable hardware identifier.

## Physical speaker layout versus exposed channels

The laptop uses a multi-speaker/four-speaker OEM layout, but Linux exposes those drivers as a **single stereo FL/FR endpoint** rather than as individually addressable speakers.

That means the existence of multiple physical drivers does not imply that PipeWire should expose a separate subwoofer or 4-channel profile. The internal amplifier/crossover/OEM tuning can still distribute the stereo signal among physical drivers behind the codec/amp path.

## Windows versus Linux sound quality

The Windows OEM stack uses **Nahimic / A-Volute** processing and tuning.

Compared with Windows, Linux playback was observed as audibly thinner/weaker, particularly in perceived bass, loudness/body and the processed spatial character.

Crucially, the same behavior reproduced on an **Ubuntu live environment**. That makes it unlikely that the problem is merely one CachyOS user configuration or one broken PipeWire preset.

The strongest current interpretation is therefore:

```text
basic codec / speaker playback works
+
OEM Windows DSP/tuning is missing on Linux
=
functional but noticeably worse speaker sound
```

## Software experiments already tried

The Linux installation has used:

```text
ALSA
PipeWire
WirePlumber
EasyEffects
```

and various HDA/ACP-related configuration experiments were attempted during troubleshooting.

One earlier configuration involved `dmic_detect=0`; another attempted audio-path/module options during AMD ACP/SOF debugging. None reproduced the OEM Windows/Nahimic speaker tuning exactly.

These experiments should not be confused with a proven need to disable SOF/ACP on this laptop: basic audio works, and the persistent quality difference points more strongly at missing OEM DSP/EQ than at total codec-path failure.

## What has not been recovered

No exact Xingyao-14-specific Linux EQ/DSP profile equivalent to the OEM Nahimic configuration has been recovered yet.

Potentially relevant Windows-side state includes Nahimic/A-Volute APO configuration and endpoint effect properties, but this repository currently has no validated coefficient/profile dump that can be reproduced on Linux.

## Current conclusion

- **Codec detection:** working.
- **Stereo speaker playback:** working.
- **Separate LFE/subwoofer Linux channel:** not exposed.
- **Windows OEM sound processing:** present through Nahimic/A-Volute.
- **Linux OEM-equivalent tuning:** not recovered.

Future audio work should focus on recovering or approximating the OEM DSP/EQ behavior rather than assuming the machine requires a completely different basic audio driver.
