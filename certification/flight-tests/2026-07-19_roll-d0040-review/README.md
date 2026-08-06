# Scout-15 Roll D 0.0040 Review

**Date:** 2026-07-19  
**Candidate log:** `2026-07-19 12-20-27.bin`  
**Baseline log:** `2026-07-19 11-52-51.bin`

## Configuration and timing proof

The log contains a 3.6-second accidental first arm at roll D `0.0036`. Roll D was
changed at 90.782 seconds, before the real flight armed at 101.234 seconds. The
120.1-second real flight and 88-second PosHold maneuver interval therefore used:

```text
ATC_RAT_RLL_D=0.0040
ATC_RAT_PIT_D=0.0040
ATC_RAT_RLL_P/I=0.135
ATC_RAT_PIT_P/I=0.135
INS_GYRO_FILTER=25
```

No AutoTune or SystemID ran, and no gains changed during the real flight.

## Revised decision

**Use P `0.135`, D `0.0040` as a provisional damping ratio, not a locked roll tune.**

The candidate improves event damping and tracking efficiency without a noise,
limiting, output, clipping or thermal regression. However, WebTools' filtered-target
step response is muted: peak `0.64`, tail `0.54`, versus baseline peak `0.79`, tail
`0.61`. Damping improved, but the current overall gain scale is too low.

Current ratio candidate:

| Axis | P | I | D | Angle P | FLTT / FLTD |
| --- | ---: | ---: | ---: | ---: | ---: |
| Pitch | 0.135 | 0.135 | 0.0040 | 4.5 | 10 / 10 |
| Roll | 0.135 | 0.135 | 0.0040 | 4.5 | 10 / 10 |

## Matched PosHold comparison

| Metric | D 0.0036 baseline | D 0.0040 candidate |
| --- | ---: | ---: |
| PosHold duration | 49.7 s | 88.0 s |
| Roll events | 7 | 15 |
| Median overshoot | 21.2% | 23.1% |
| Overshoot p75 | 35.1% | 35.7% |
| Median ring count | 2.0 | 1.0 |
| Ring p75 | 3.5 | 1.5 |
| Tracking RMS | 9.82 deg/s | 8.06 deg/s |
| Rate ratio | 0.922 | 0.879 |
| Correlation | 0.738 | 0.763 |
| Output p95 / max | 0.077 / 0.135 | 0.070 / 0.108 |
| Dmod minimum | 1.0 | 1.0 |

Median and upper-quartile overshoot are effectively unchanged, while ringing,
tracking error and output demand improve. The lower rate ratio means this is a more
damped response, not a more aggressive one.

## Matched command bands

| Command band | Metric | D 0.0036 | D 0.0040 |
| --- | --- | ---: | ---: |
| 30-45 deg/s | Median overshoot | 26.8% | 23.1% |
| 30-45 deg/s | Median rise | 77 ms | 62 ms |
| 30-45 deg/s | Median ring | 2.0 | 1.0 |
| 45-60 deg/s | Median overshoot | 20.1% | 23.1% |
| 45-60 deg/s | Median ring | 3.0 | 1.0 |

Direction balance also improved on the previously weak positive direction: median
positive overshoot changed from 42.4% to 20.6%, with median ring count changing from
2 to 1. Negative-direction commands in the candidate were generally smaller, so
direction statistics are supporting context rather than a perfect pair.

## Noise and health

| Metric | D 0.0036 | D 0.0040 |
| --- | ---: | ---: |
| Actual roll-rate RMS, 20-100 Hz | 0.15293 deg/s | 0.12196 deg/s |
| Roll D-term RMS, 20-100 Hz | 0.000544 | 0.000482 |
| Actual roll-rate RMS, 100-180 Hz | 0.07224 deg/s | 0.06998 deg/s |
| Roll D-term RMS, 100-180 Hz | 0.000260 | 0.000279 |

Candidate maneuver health:

- clipping: zero
- vibration median/p95: 12.81 / 27.53 m/s2
- current median/p95/max: 9.95 / 14.54 / 19.42 A
- minimum voltage: 22.86 V
- maximum IMU temperature: 39.5 C
- maximum ESC temperature: 35 C
- ESC error p95: 0%

The small D-term increase above 100 Hz is negligible in absolute terms and did not
propagate into higher actual-rate noise.

## Next step

The remaining tracking-error energy is concentrated around 5-6 Hz. A subsequent
fixed-P/I test at roll D `0.0044` worsened median overshoot, ring p75 and 3-6 Hz error,
so the preferred ratio remains P `0.135`, D `0.0040`. See
`../2026-07-19_roll-d0044-review/README.md`.

Next, scale roll P/I/D together while preserving this ratio: P/I `0.150`, D
`0.0045`. Do not jump directly from the current muted response to an assumed
1.0-equivalent gain multiplier; closed-loop response is not linear enough to justify
that shortcut.