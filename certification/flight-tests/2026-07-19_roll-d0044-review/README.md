# Scout-15 Roll D 0.0044 Review

**Date:** 2026-07-19  
**Candidate log:** `2026-07-19 12-41-17.bin`  
**Baseline log:** `2026-07-19 12-20-27.bin`

## Configuration proof

The candidate flight used roll P/I `0.135`, roll D `0.0044`, pitch P/I `0.135`,
pitch D `0.0040`, gyro LPF 25 Hz and the enabled harmonic notch. No AutoTune,
SystemID or in-flight gain change occurred. The analyzed PosHold interval was 44.4 s.

## Decision

**Reject roll D `0.0044` at P/I `0.135`; restore roll D `0.0040`.**

The extra D did not improve the remaining 5-6 Hz ringing. It increased event
overshoot and upper-quartile ring count despite retaining ample limiter and output
headroom.

## Matched PosHold comparison

| Metric | D 0.0040 | D 0.0044 |
| --- | ---: | ---: |
| Events | 15 | 12 |
| Median command amplitude | 33.9 deg/s | 41.8 deg/s |
| Median overshoot | 23.1% | 33.8% |
| Overshoot p75 | 35.7% | 42.0% |
| Median ring count | 1.0 | 1.5 |
| Ring p75 | 1.5 | 3.2 |
| Ring p95 | 4.6 | 4.0 |
| Median rise | 78 ms | 70 ms |
| Tracking correlation | 0.763 | 0.801 |
| Dmod minimum | 1.0 | 1.0 |

In the common 45-55 deg/s command range, D `0.0044` produced several 3-4-cycle
events. The candidate's faster rise and higher correlation do not compensate for the
worse overshoot and ringing.

## Frequency-domain result

| Metric | D 0.0040 | D 0.0044 |
| --- | ---: | ---: |
| 3-6 Hz error RMS | 1.540 deg/s | 2.025 deg/s |
| 6-10 Hz error RMS | 1.147 deg/s | 1.637 deg/s |
| Dominant control-band error | 5.95 Hz, 3.4 dB | 6.23 Hz, 4.7 dB |
| D RMS, 20-100 Hz | 0.000482 | 0.000570 |

The filtered-target synthetic transfer curve rose from peak/tail `0.64/0.54` to
`0.72/0.64`, but the candidate flight used stronger excitation and its actual event
damping was worse. The transfer improvement cannot be credited to the D increase.

## Health and telemetry

- zero clipping
- Dmod minimum 1.0
- controller output p95/max 0.077/0.120
- vibration median/p95 10.90/23.28 m/s2
- current p95/max 14.81/18.98 A
- maximum ESC temperature 38 C
- maximum IMU temperature 46.7 C

All four ESC `Err` fields rose together for about 2.2 seconds near the start of
PosHold, with approximately 11% p95 and 46% maximum. Simultaneous timing across all
motors indicates a shared bidirectional-DShot telemetry disturbance rather than four
independent motor faults. It was not sustained, but the next flight must verify it
does not recur.

## Next step

Quadratic interpolation through D `0.0036`, `0.0040` and `0.0044` gives local minima
of `0.004089` from 3-6 Hz error RMS, `0.004104` from dominant ring-peak power,
`0.004067` from median ring count and `0.004016` from ring p75. Their mean is
`0.004069`. Treat this as **D approximately 0.0041**, not an exact optimum; the
different flight excitation implies practical uncertainty of at least +/-`0.0001`.

To address the muted response, preserve the interpolated D:P ratio while raising the
overall roll gain. First bounded scale:

```text
ATC_RAT_RLL_P=0.150
ATC_RAT_RLL_I=0.150
ATC_RAT_RLL_D=0.0045
```

At P `0.150`, exact ratio scaling from the interpolated D estimate gives `0.00452`,
rounded to `0.0045`. Accept only if command response moves toward 1.0 without
returning the ringing, noise, telemetry errors or thermal issues.