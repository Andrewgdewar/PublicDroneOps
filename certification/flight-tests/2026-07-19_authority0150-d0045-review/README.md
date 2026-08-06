# Scout-15 Fixed Authority P/I 0.150, D 0.0045 Review

**Date:** 2026-07-19  
**Log:** `2026-07-19 13-50-30.bin`

## Configuration and phases

- roll and pitch P/I `0.150`, D `0.0045`
- gyro LPF 25 Hz, harmonic notch enabled
- roll block 58.8-83.8 s
- pitch block 88.8-118.8 s
- high-speed sortie after approximately 123.8 s
- no AutoTune, SystemID or in-flight gain changes

## Authority result

The fixed P/I scale is established for the D sweep:

| Axis | Filtered-target peak / tail | Rate ratio |
| --- | ---: | ---: |
| Roll | 0.87 / 0.79 | 1.04 |
| Pitch | 0.74 / 0.69 | 1.09 |

Roll is near the requested 0.9 graph target. Pitch is improved but still lower. Hold
P/I fixed while adjusting D; do not move P/I based on this first damping point.

## Damping result at D 0.0045

| Metric | Roll | Pitch |
| --- | ---: | ---: |
| Median overshoot | 35.4% | 30.3% |
| Overshoot p75 | 44.9% | 48.1% |
| Median ring | 1.5 | 1.0 |
| Ring p75 | 5.2 | 3.2 |
| 3-6 Hz error RMS | 2.68 deg/s | 1.46 deg/s |
| 6-10 Hz error RMS | 2.59 deg/s | 1.26 deg/s |
| Dominant ring peak | 6.14 Hz, 10.9 dB | 5.86 Hz, 3.2 dB |
| Dmod minimum | 1.0 | 1.0 |
| D RMS, 20-100 Hz | 0.000680 | 0.000610 |

Both axes retain approximately 6 Hz ringing, especially roll. Dmod, output and noise
headroom permit a modest upward D bracket at unchanged P/I.

## Next D point

```text
Roll:  P/I=0.150, D=0.0048
Pitch: P/I=0.150, D=0.0048
```

This is a 6.7% D increase. If ringing improves, interpolate between `0.0045` and
`0.0048`. If ringing worsens, restore `0.0045` and test a lower D bracket instead.

## High-speed vibration gate

The sortie reached 19.7 m/s. IMU0 accumulated 74 clips beginning at 16.9 m/s;
IMU1 had zero clips. Vibration vector magnitude reached 100.5 m/s2. This occurred
before landing and is unrelated to landing impact.

Do not include another sortie in the D sweep. Keep speed below 10 m/s. High-speed
vibration requires a separate mechanical/aerodynamic investigation.