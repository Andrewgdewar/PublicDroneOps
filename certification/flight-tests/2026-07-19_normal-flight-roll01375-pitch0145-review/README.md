# Scout-15 Normal Flight Roll 0.1375 / Pitch 0.145 Review

**Date:** 2026-07-19  
**Log:** `2026-07-19 13-32-51.bin`

## Configuration and phases

- roll P/I `0.1375`, D `0.00415`
- pitch P/I `0.145`, D `0.0043`
- gyro LPF 25 Hz, harmonic notch enabled
- roll block 56.8-86.8 s
- pitch block 86.8-116.8 s
- park sortie approximately 116.8-151.8 s, maximum 18.1 m/s
- no AutoTune, SystemID or in-flight gain changes

The pilot reported a hard landing. Landing data after the sortie are excluded from
the controller comparisons and from the speed-related vibration interpretation.

## Decisions

1. **Reject roll P/I `0.1375`, D `0.00415`; restore `0.135/0.0040`.** The midpoint
   still exhibits the same 5-6 Hz ringing trend as the higher roll scales.
2. **Reject pitch P/I `0.145`, D `0.0043`; restore the provisionally accepted
   `0.140/0.00415`.** Event response looked fast, but spectral error and D noise
   regressed relative to the `0.140` pitch candidate.

## Roll block

| Metric | Preferred 0.135 / 0.0040 | Candidate 0.1375 / 0.00415 |
| --- | ---: | ---: |
| Filtered-target peak | approximately 0.64-0.68 | 0.76 |
| Filtered-target tail | approximately 0.54-0.57 | 0.68 |
| Median overshoot | 23.1% | 36.2% |
| Median ring | 1.0 | 2.0 |
| Ring p75 | 1.5 | 3.5 |
| 3-6 Hz error RMS | 1.54 deg/s | 1.95 deg/s |
| 6-10 Hz error RMS | 1.15 deg/s | 1.72 deg/s |
| Dominant ring peak | about 5-6 Hz | 6.23 Hz, 7.9 dB |
| Dmod minimum | 1.0 | 1.0 |

For common 30-60 deg/s events, median ring changed 1.0 -> 1.0 but ring p75 changed
1.5 -> 1.2 only because the candidate had fewer high-amplitude events; across all
candidate events median/p75 were 2.0/3.5. The spectral error increase confirms the
roll midpoint remains above the clean feedback-gain boundary.

## Pitch block

| Metric | Accepted candidate 0.140 / 0.00415 | Candidate 0.145 / 0.0043 |
| --- | ---: | ---: |
| Filtered-target peak | 0.63 | 0.61 |
| Filtered-target tail | 0.62 | 0.55 |
| Median overshoot | 29.2% | 31.1% |
| Median ring | 1.0 | 0.0 |
| Ring p75 | 1.0 | 1.0 |
| 3-6 Hz error RMS | 1.60 deg/s | 2.18 deg/s |
| 6-10 Hz error RMS | 0.84 deg/s | 1.31 deg/s |
| D RMS, 20-100 Hz | 0.000539 | 0.000747 |
| Dmod minimum | 1.0 | 1.0 |

The `0.145` pitch scale did not increase the mean filtered-target response and raised
control-band error and D noise. The faster event response does not justify keeping it.

## Sortie and vibration

Controller outputs remained unsaturated and Dmod stayed 1.0 during the sortie. One
IMU clip occurred at 124.5 s, 15.9 m/s, during a high-vibration burst. This was well
before the hard landing.

| Ground speed | Vibration vector median / p95 / max |
| --- | ---: |
| 0-5 m/s | 6.38 / 14.87 / 27.44 m/s2 |
| 5-10 m/s | 12.93 / 20.61 / 28.68 m/s2 |
| 10-15 m/s | 28.42 / 53.55 / 59.84 m/s2 |
| 15-20 m/s | 39.54 / 60.11 / 67.23 m/s2 |

Vibration rises sharply above 10 m/s and cannot be attributed to the landing. Keep
the next confirmation flight at or below 10 m/s and investigate the high-speed
mechanical/aerodynamic vibration separately before using 15-18 m/s operationally.

## Best current feedback tune

```text
Roll:  P/I=0.135, D=0.0040
Pitch: P/I=0.140, D=0.00415
Angle P: 4.5
Gyro LPF: 25 Hz
```

Confirm this asymmetric tune in one normal flight without changing gains. If clean,
freeze feedback gains and use feedforward/input shaping for further command-response
improvement rather than raising feedback gains into the ringing region.