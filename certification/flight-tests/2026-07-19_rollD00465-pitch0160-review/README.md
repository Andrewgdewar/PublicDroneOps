# Scout-15 Roll D 0.00465 and Pitch P/I 0.160 Review

**Date:** 2026-07-19  
**Log:** `2026-07-19 14-11-15.bin`

## Configuration and blocks

- roll P/I `0.150`, D `0.00465`
- pitch P/I `0.160`, D `0.0048`
- gyro LPF 25 Hz, harmonic notch enabled
- roll block 49.7-73.7 s
- pitch block 73.7-97.7 s
- normal-flight sortie after the deliberate blocks, maximum speed 18.3 m/s
- no AutoTune, SystemID or in-flight gain changes

## Decisions

1. **Accept roll D `0.00465` at P/I `0.150`.** It is the best damping compromise
   among D `0.0045`, `0.00465` and `0.0048`.
2. **Pitch P/I `0.160` is still below the desired authority.** Keep pitch D fixed at
   `0.0048` and test the next bounded P/I point, `0.170`.

## Roll D comparison

| Metric | D 0.0045 | D 0.00465 | D 0.0048 |
| --- | ---: | ---: | ---: |
| Filtered-target peak / tail | 0.87 / 0.79 | 0.79 / 0.76 | 0.89 / 0.84 |
| Median overshoot | 35.4% | 33.7% | 51.4% |
| Median ring | 1.5 | 2.0 | 2.0 |
| Ring p75 | 5.25 | 2.5 | 3.0 |
| Tracking RMS | 11.67 deg/s | 9.31 deg/s | 11.43 deg/s |
| Correlation | 0.815 | 0.849 | 0.839 |
| 3-6 Hz error RMS | 2.68 deg/s | 1.74 deg/s | 2.82 deg/s |
| 6-10 Hz error RMS | 2.59 deg/s | 1.39 deg/s | 1.68 deg/s |
| D RMS, 20-100 Hz | 0.000680 | 0.000571 | 0.000648 |
| Output p95 | 0.082 | 0.073 | 0.086 |

The synthetic transfer magnitude at the midpoint is below the endpoints, but every
damping, tracking, noise and output metric favors `0.00465`. Keep it for roll.

## Pitch authority comparison

| Metric | P/I 0.150, D 0.0048 | P/I 0.160, D 0.0048 |
| --- | ---: | ---: |
| Filtered-target peak / tail | 0.67 / 0.62 | 0.69 / 0.64 |
| Rate ratio | 1.06 | 0.99 |
| Correlation | 0.760 | 0.791 |
| Median overshoot | 30.7% | 36.9% |
| Median ring | 0.0 | 2.0 |
| 3-6 Hz error RMS | 2.19 deg/s | 2.22 deg/s |
| 6-10 Hz error RMS | 1.51 deg/s | 1.45 deg/s |

Pitch authority improved only slightly and remains below the requested graph target.
Do not change D while establishing pitch P/I. Test P/I `0.170`, D `0.0048`, then hold
P/I fixed and tune pitch D.

## Health

- zero clipping on both IMUs
- vibration median/p95/max 10.93/44.02/68.99 m/s2
- current median/p95/max 10.44/15.81/21.61 A
- minimum voltage 22.75 V
- maximum speed 18.3 m/s
- maximum IMU temperature 44.3 C
- maximum ESC temperature 38 C
- ESC error p95/max 0.04/12.8%

No health gate failed, but the sortie again showed elevated high-speed vibration.
Do not include a sortie in the pitch authority test.