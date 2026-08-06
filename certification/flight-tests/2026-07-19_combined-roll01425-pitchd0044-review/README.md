# Scout-15 Combined Roll Scale and Pitch D Review

**Date:** 2026-07-19  
**Log:** `2026-07-19 13-06-00.bin`

## Configuration and blocks

- roll P/I `0.1425`, D `0.0043`
- pitch P/I `0.135`, D `0.0044`
- gyro LPF 25 Hz, harmonic notch enabled
- no AutoTune, SystemID or in-flight gain changes
- roll block approximately 64.5-104.5 s
- pitch block approximately 109.5-159.5 s

## Decisions

1. **Reject roll P/I `0.1425`, D `0.0043`.** Authority improved, but control-band
   ringing remained materially worse than the P/I `0.135`, D `0.0040` baseline.
2. **Reject pitch D `0.0044` at P/I `0.135`.** It did not improve the practical
   damping tradeoff over D `0.0040`.

The pilot-reported feel improved because both axes had more authority. The log still
shows that roll crossed the acceptable damping boundary, so subjective improvement
does not make this the final tune.

## Roll result

| Metric | Baseline 0.135 / 0.0040 | Candidate 0.1425 / 0.0043 |
| --- | ---: | ---: |
| Filtered-target peak | 0.68 | 0.76 |
| Filtered-target tail | 0.57 | 0.68 |
| Median overshoot | 14.6%* | 24.7% |
| Median ring | 1.0* | 2.0 |
| Ring p75 | 3.5* | 5.25 |
| 3-6 Hz error RMS | 1.26 deg/s* | 2.98 deg/s |
| 6-10 Hz error RMS | 0.69 deg/s* | 2.09 deg/s |
| Dominant ring peak | 4.96 Hz, 0.7 dB* | 5.95 Hz, 10.8 dB |
| Dmod minimum | 1.0 | 1.0 |

`*` The equal-duration baseline slice had only three detected events. The complete
baseline PosHold interval gives median overshoot 23.1%, median ring 1.0 and 3-6 Hz
error 1.54 deg/s. Both baseline views still favor the lower gain.

## Pitch result

| Metric | D 0.0036 | D 0.0040 | D 0.0044 |
| --- | ---: | ---: | ---: |
| Median overshoot | 30.9% | 31.7% | 31.1% |
| Overshoot p75 | 73.4% | 38.0% | 61.4% |
| Median ring | 2.0 | 0.0 | 0.0 |
| Ring p75 | 3.0 | 2.0 | 2.0 |
| Median rise | 110 ms | 61 ms | 75 ms |
| 3-6 Hz error RMS | 2.20 deg/s | 2.45 deg/s | 2.47 deg/s |
| 6-10 Hz error RMS | 1.12 deg/s | 1.08 deg/s | 1.42 deg/s |
| D RMS, 20-100 Hz | 0.000565 | 0.000613 | 0.000720 |

Pitch D `0.0040` remains the best practical ratio: it captures the ringing improvement
without the extra noise and upper-quartile overshoot of `0.0044`. Quadratic fits are
not trusted here because spectral and event metrics disagree and the flights had
different command distributions.

## Health

- vibration median/p95/max 9.48/20.55/43.47 m/s2
- zero clipping
- current median/p95/max 10.26/13.83/18.28 A
- minimum voltage 22.74 V
- maximum IMU temperature 47.2 C
- maximum ESC temperature 39 C
- ESC error p95 0.05% or less

All four ESCs had brief simultaneous error maxima around 52%, but p95 remained near
zero. This is consistent with a short shared bidirectional-DShot telemetry disturbance,
not sustained motor faults.

## Next scale

Test both axes at a smaller overall scale, keeping their preferred ratios:

```text
Roll:  P/I=0.140, D=0.0042
Pitch: P/I=0.140, D=0.00415
```

Use separated roll and pitch blocks again. This is a 3.7% P/I increase rather than
the failed 5.6% roll step. Accept each axis independently.