# Scout-15 Combined Roll/Pitch P/I 0.140 Review

**Date:** 2026-07-19  
**Log:** `2026-07-19 13-17-04.bin`

## Configuration and blocks

- roll P/I `0.140`, D `0.0042`
- pitch P/I `0.140`, D `0.00415`
- gyro LPF 25 Hz, harmonic notch enabled
- roll block 58.2-90.2 s
- pitch block 94.2-130.2 s
- no AutoTune, SystemID or in-flight gain changes

## Decisions

1. **Reject roll P/I `0.140`, D `0.0042`.** Authority improved, but event and
   frequency-domain ringing remained materially worse than the P/I `0.135`, D
   `0.0040` baseline.
2. **Provisionally accept pitch P/I `0.140`, D `0.00415`.** Moderate/large command
   damping and spectral error improved, with no limiting or health regression.

## Roll comparison

| Metric | Baseline 0.135 / 0.0040 | Candidate 0.140 / 0.0042 |
| --- | ---: | ---: |
| Filtered-target peak | 0.64-0.68 | 0.79 |
| Filtered-target tail | 0.54-0.57 | 0.75 |
| Median overshoot, all events | 23.1% | 26.6% |
| Median ring | 1.0 | 2.0 |
| Ring p75 | 1.5 | 3.0 |
| 3-6 Hz error RMS | 1.54 deg/s | 1.86 deg/s |
| 6-10 Hz error RMS | 1.15 deg/s | 1.48 deg/s |
| Dominant ringing peak | about 5-6 Hz | 6.43 Hz, 4.0 dB |
| Dmod minimum | 1.0 | 1.0 |

For common 30-60 deg/s events, median overshoot changed 23.1% -> 31.5%, ring
1.0 -> 2.0 and ring p75 1.5 -> 4.8. Roll `0.140` is above the damping boundary.

## Pitch comparison

| Metric | Baseline 0.135 / 0.0040 | Candidate 0.140 / 0.00415 |
| --- | ---: | ---: |
| Filtered-target peak | 0.64 | 0.63 |
| Filtered-target tail | 0.55 | 0.62 |
| Median overshoot, 30-60 deg/s | 34.3% | 29.2% |
| Overshoot p75, 30-60 deg/s | 37.0% | 41.2% |
| Median ring, 30-60 deg/s | 1.0 | 0.0 |
| Ring p75, 30-60 deg/s | 2.0 | 1.0 |
| 3-6 Hz error RMS | 2.61 deg/s | 1.60 deg/s |
| 6-10 Hz error RMS | 1.14 deg/s | 0.84 deg/s |
| D RMS, 20-100 Hz | 0.000620 | 0.000539 |
| Dmod minimum | 1.0 | 1.0 |

Small 18-30 deg/s event overshoot was high because normalized error is sensitive to
small denominators. Moderate/large commands, ring counts, spectral error and the
pilot's improved feel support the candidate. Keep pitch at `0.140/0.00415` while roll
is refined, then confirm both axes in normal flight.

## Health

- vibration median/p95/max 8.64/19.17/38.88 m/s2
- zero clipping
- current median/p95/max 10.65/13.98/17.59 A
- minimum voltage 22.54 V
- maximum IMU temperature 45.3 C
- maximum ESC temperature 40 C
- ESC error p95 0.05% or less

No hardware-health gate failed.

## Next roll midpoint

The useful roll boundary lies narrowly between P/I `0.135` and `0.140`. Test only:

```text
ATC_RAT_RLL_P=0.1375
ATC_RAT_RLL_I=0.1375
ATC_RAT_RLL_D=0.00415
```

Keep pitch at the provisional `0.140/0.00415` values and do not add pitch maneuvers.

An all-point trend check used the three fixed-P D tests and the ratio-preserving scale
tests at P `0.135`, `0.140`, `0.1425` and `0.150`. Linear and quadratic fits agree
that the useful transition lies near P `0.137-0.138`. Local interpolation at `0.1375`
predicts peak/tail approximately `0.715/0.645`, 3-6 Hz error approximately 1.70 deg/s,
median ring approximately 1.5 and ring p75 approximately 2.25. Different excitation
between flights limits the precision, so these values are acceptance guidance rather
than a claimed mathematical optimum.