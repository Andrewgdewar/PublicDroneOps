# Scout-15 Roll Scale P/I 0.150 Review

**Date:** 2026-07-19  
**Candidate log:** `2026-07-19 12-54-26.bin`  
**Baseline log:** `2026-07-19 12-20-27.bin`

## Decision

**Reject roll P/I `0.150`, D `0.0045`; it is too aggressive.**

The candidate achieved the desired filtered-target authority but returned strong
5-6 Hz ringing and overshoot. Configuration was otherwise clean: gyro LPF 25 Hz,
pitch unchanged, no AutoTune/SystemID, and no in-flight gain changes.

## Matched PosHold comparison

| Metric | P/I 0.135, D 0.0040 | P/I 0.150, D 0.0045 |
| --- | ---: | ---: |
| Filtered-target peak | 0.64 | 0.89 |
| Filtered-target tail | 0.54 | 0.81 |
| Median overshoot | 23.1% | 41.0% |
| Overshoot p75 | 35.7% | 54.2% |
| Median ring count | 1.0 | 2.0 |
| Ring p75 | 1.5 | 4.0 |
| Ring p95 | 4.6 | 7.0 |
| 3-6 Hz error RMS | 1.54 deg/s | 3.27 deg/s |
| 6-10 Hz error RMS | 1.15 deg/s | 2.32 deg/s |
| Dominant ringing peak | 5.95 Hz, 3.4 dB | 5.35 Hz, 9.3 dB |
| Dmod minimum | 1.0 | 1.0 |

The candidate was more strongly excited, but the event distributions, doubled
control-band error and higher ring counts all point toward excessive loop gain.

## Health

- zero clipping
- Dmod minimum 1.0
- controller output p95/max 0.092/0.139
- vibration median/p95 11.17/22.22 m/s2
- current p95/max 14.57/16.98 A
- maximum IMU temperature 40.9 C
- maximum ESC temperature 39 C
- ESC error p95 0%

The rejection is control-response based, not a hardware-health failure.

## Next scale

Linear interpolation of measured transfer peaks places peak `0.80` near P `0.1445`.
Because ringing grew nonlinearly, use a conservative midpoint below that estimate:

```text
ATC_RAT_RLL_P=0.1425
ATC_RAT_RLL_I=0.1425
ATC_RAT_RLL_D=0.0043
```

This preserves the estimated D:P ratio and should target approximately 0.76-0.78
filtered-target peak. Accept only if authority improves over the `0.135` baseline
without materially increasing 5-6 Hz ringing.