# Audio

## Codec

Linux identifies the internal analog codec path as:

```text
Realtek ALC256 Analog
```

The codec is presented through the AMD/Ryzen HDA/ACP/SOF audio stack.

## Logical speaker topology

PipeWire/WirePlumber exposes the internal speaker endpoint as stereo:

```text
output_FL
output_FR
```

No separate logical endpoint has been observed for:

```text
LFE
2.1
4.0
separate subwoofer
```

The chassis contains four physical speaker drivers, two per side, but they are not exposed to Linux as four independent channels.

## Windows OEM processing

The Windows OEM audio stack uses Nahimic / A-Volute processing.

The exact OEM DSP/EQ coefficients and endpoint-effect configuration for this model have not been recovered. Linux therefore has working codec/speaker support but does not currently reproduce the OEM Windows processing chain.

## Established status

| Item | Status |
|---|---|
| ALC256 codec detection | Confirmed |
| Internal speaker playback | Confirmed |
| Logical channel layout | Stereo FL/FR |
| Separate LFE channel | Not exposed |
| Four independent speaker channels | Not exposed |
| OEM Nahimic/A-Volute processing on Windows | Present |
| Equivalent Linux OEM DSP profile | Not recovered |
