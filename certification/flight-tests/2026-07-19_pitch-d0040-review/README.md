# Scout-15 Pitch D 0.0040 Review

**Date:** 2026-07-19  
**Candidate log:** `2026-07-19 12-09-37.bin`  
**Baseline log:** `2026-07-19 11-52-51.bin`

## Configuration proof

The candidate flight isolated the intended change:

```text
ATC_RAT_PIT_D: 0.0036 -> 0.0040
```

Pitch and roll P/I remained `0.135`, roll D remained `0.0036`, gyro LPF remained
25 Hz, raw logging was off, the harmonic notch remained enabled, and no AutoTune,
SystemID or in-flight gain changes occurred.

## Revised decision

**Use P `0.135`, D `0.0040` as a provisional pitch damping ratio, not a locked tune.**

The matched PosHold comparison supports the damping benefit of the 11.1% D increase.
It improved ringing without creating D limiting, high-frequency oscillation, clipping,
excessive controller output or a thermal problem. However, the filtered-target
WebTools response remained muted (candidate peak about `0.63`, tail about `0.51`).
Pitch will need a separate ratio-preserving gain-scale pass after roll establishes a
safe scaling workflow.

## Matched PosHold comparison

| Metric | D 0.0036 baseline | D 0.0040 candidate |
| --- | ---: | ---: |
| PosHold duration | 49.7 s | 61.4 s |
| Pitch events | 11 | 24 |
| Median command amplitude | 25.4 deg/s | 33.4 deg/s |
| Median overshoot | 31.1% | 31.9% |
| Overshoot p75 | 73.4% | 37.3% |
| Overshoot p95 | 89.8% | 67.5% |
| Median rise time | 110 ms | 67 ms |
| Median ring count | 2.0 | 0.0 |
| Ring p75 | 3.0 | 2.0 |
| Dmod minimum | 1.0 | 1.0 |
| Controller output maximum | 0.139 | 0.171 |

The candidate was flown with stronger commands, so whole-window RMS error and current
are not direct quality comparisons. Command-amplitude bins provide the cleaner check.

## Matched command bands

| Command band | Metric | D 0.0036 | D 0.0040 |
| --- | --- | ---: | ---: |
| 18-30 deg/s | Median overshoot | 70.7% | 15.0% |
| 18-30 deg/s | Overshoot p75 | 77.3% | 49.7% |
| 18-30 deg/s | Median rise | 137 ms | 55 ms |
| 18-30 deg/s | Median ring | 1.0 | 0.0 |
| 30-45 deg/s | Median overshoot | 24.3% | 35.6% |
| 30-45 deg/s | Median rise | 94 ms | 43 ms |
| 30-45 deg/s | Median ring | 2.5 | 0.5 |
| 45-60 deg/s | Median overshoot | 23.6% | 32.2% |
| 45-60 deg/s | Median rise | 76 ms | 76 ms |
| 45-60 deg/s | Median ring | 2.5 | 1.5 |

The candidate does not reduce every overshoot statistic, but it substantially improves
ringing and rise time and removes the severe low-amplitude weakness seen in the
baseline. Positive and negative candidate responses were also more consistent.

## Noise and health cost

- actual pitch-rate RMS at 20-100 Hz: 0.1583 -> 0.1704 deg/s (+7.6%)
- pitch D-term RMS at 20-100 Hz: 0.000561 -> 0.000673
- Dmod minimum: 1.0; no D limiting
- IMU clipping: zero
- vibration median/p95: 8.89 / 21.31 m/s2
- current median/p95/max: 9.66 / 14.11 / 20.54 A
- minimum voltage: 22.91 V
- maximum IMU temperature: 35.1 C
- maximum ESC temperature: 37 C
- ESC error p95: 0.06% or less

The measured noise increase is small in absolute terms and did not produce a safety
or thermal regression.

## Next isolation test

Keep pitch D `0.0040` and test roll D `0.0036 -> 0.0040` using only repeated separated
left/right roll commands. Do not combine pitch and roll testing in that log.