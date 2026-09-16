# Audio notes

## Linux hardware path

Linux detects the laptop's audio through AMD/Ryzen HDA/ACP-related devices. ALSA probing on the researched machine identified a **Realtek ALC256 Analog** codec path for the internal speakers.

PipeWire exposed the internal speakers as a normal stereo sink, with left/right channels rather than a separately exposed 2.1/4.0/LFE topology.

A representative sink name observed earlier was similar to:

```text
alsa_output.pci-0000_c1_00.6.HiFi__Speaker__sink
```

Exact PCI numbering can change between kernels/firmware and should not be treated as stable ABI.

## Physical speakers vs Linux channel model

The chassis has multiple physical speaker drivers, but Linux presented them as a **stereo FL/FR endpoint**. No dedicated subwoofer/LFE channel was exposed in the observed PipeWire graph.

Therefore the multiple physical drivers are most likely paired internally into left/right speaker groups rather than being presented as independently addressable channels.

## Windows vs Linux sound quality

The laptop's Windows image/OEM stack uses **Nahimic** processing/tuning.

On Linux, raw speaker playback works, but the missing OEM DSP profile produces noticeably worse sound: thinner tonal balance, weaker perceived bass/loudness and less of the processed spatial effect heard under Windows.

The same poor sound character reproduced in an Ubuntu live environment, so it was not specific to CachyOS or one custom PipeWire configuration.

## Software experiments

The Linux installation has used PipeWire/WirePlumber and EasyEffects for software EQ/processing.

Experiments also included ordinary HDA/ALSA profile changes and kernel audio-path options, but none reproduced the OEM Windows/Nahimic tuning exactly.

## Current conclusion

There is no evidence that the internal speakers are electrically unavailable or misdetected on Linux. The major gap is **OEM DSP/tuning**, not basic codec support.

Any future work should focus on obtaining/recreating an EQ/DSP profile rather than assuming the laptop needs a completely different Linux audio driver.
