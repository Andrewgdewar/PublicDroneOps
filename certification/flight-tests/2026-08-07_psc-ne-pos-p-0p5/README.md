# Scout-15 Position Loop — `PSC_NE_POS_P` 1.0 → 0.5

**Aircraft:** Scout-15 quadcopter
**Flight event:** 2026-08-07
**Flight log:** `2026-08-07 16-01-25.bin`
**Objective:** Test the predicted fix for the 0.19–0.20 Hz position-controller oscillation, and
confirm the telemetry congestion fix
**Outcome:** **Prediction confirmed in direction, short in magnitude.** Every hold metric improved
23–52%, scatter collapsed from 78% to 9%, oscillation peak moved 0.20 → 0.12 Hz. Heartbeat gaps
fell 20 → 5
**Decision:** Adopt 0.5, then step to **0.3** — the trend has not turned
**Parameter snapshot:** `certification/parameters/0807.param`

## Summary

First flight testing the position-cascade diagnosis. One tuning variable, verified loaded from the
log before interpretation.

The oscillation did not disappear — it **moved down in frequency and lost roughly a third of its
amplitude**, which is exactly what halving the outer-loop gain should do. The residual peak at
0.12–0.13 Hz says there is more available.

## Configuration proof — read from the log

| Item | Value |
| --- | --- |
| `PSC_NE_POS_P` | **0.5** — the single tuning variable |
| `PSC_NE_VEL_P` / `_I` / `_D` | 1.0 / 0.5 / 0 — unchanged |
| `EK3_SRC2_VELXY` | 0 — flow still not fused |
| `FLOW_FXSCALER` / `FYSCALER` | 338 / 311 |
| `FS_OPTIONS` / `FS_GCS_TIMEOUT` | **16 / 10** — new |
| `INS_GYRO_FILTER` | 27 |
| `ATC_ANG_RLL_P` / `PIT_P` | 6.0 / 7.0 |
| GPS | **RTK Fixed 100%**, `HAcc` 0.030 m, RTCMFU 640 |
| Modes | Loiter 57.6 → 115.3 s, PosHold 115.3 → 156.5 s |

## Result 1 — position hold

Hands-off windows ≥ 20 s, single mode, airborne:

| | `POS_P` 1.0 (`14-28-49`) | **`POS_P` 0.5 (this flight)** | change |
| --- | --- | --- | --- |
| ampR | 0.646 | **0.365** | −44% |
| ampP | 0.698 | **0.336** | −52% |
| box | 1.20 m | **0.66 m** | −45% |
| rms | 0.236 m | **0.158 m** | −33% |

### The scatter collapsed, which matters more than the mean

| | window spread on ampR |
| --- | --- |
| `POS_P` 1.0 | 0.551 – 0.981 — **78%** |
| `POS_P` 0.5 | 0.350 – 0.382 — **9%** |

**The two flights' ranges do not overlap.** That clears the scatter bar that invalidated the RTK
comparison, and the collapse in variance is itself the signature of leaving an oscillatory regime —
a marginally stable loop produces wildly different amplitudes depending on how it happens to be
excited.

## Result 2 — the mechanism moved as predicted

| | `POS_P` 1.0 | `POS_P` 0.5 |
| --- | --- | --- |
| position error peak | 0.19 – 0.20 Hz | **0.12 – 0.13 Hz** |
| velocity loop phase | −91 to −92° | −87° |
| velocity loop gain | 1.13 – 1.18 | 1.41 – 1.60 |
| position error rms | 0.129 – 0.157 m | **0.087 – 0.124 m** |

Halving the outer gain halves the outer crossover, and the peak moved down accordingly. Predicted
crossover at `POS_P` 0.5 is 0.5 rad/s = 0.080 Hz; measured peak 0.12–0.13 Hz — same direction, same
order.

**The velocity loop phase is essentially unchanged at −87°**, as expected since the velocity loop
was not touched. Total loop phase is therefore still near −177°. **We have not eliminated the
marginal-stability relationship — we have moved it to a lower frequency where less disturbance
energy exists and the amplitude is smaller.** That is a real improvement, but it explains why the
peak persists.

### Correction to an overstated figure

An initial reading measured oscillation energy in a fixed **0.15–0.25 Hz** band and reported a
**4–6× reduction.** That band no longer contains the peak, which moved to 0.12–0.13 Hz, so the
comparison flattered the result. **Withdrawn.**

Re-measured across **0.05–0.45 Hz**, containing both old and new peaks:

| | `POS_P` 1.0 | `POS_P` 0.5 | change |
| --- | --- | --- | --- |
| peak amplitude | 0.125, 0.171 | 0.086, 0.109 | **−31 to −37%** |
| total band energy | 0.163, 0.170 | 0.119, 0.128 | **−25%** |
| 2D error rms | 0.175, 0.283 m | 0.135, 0.186 m | **−23 to −34%** |

**The honest figure is 25–35%, not 4–6×.**

## Result 3 — telemetry congestion fix confirmed

Ground-side changes: telemetry stream rates cut (raw sensors 2 → 0 Hz, extra 1 10 → 3 Hz, all
others 3 Hz) and GCS CSV parameter logging disabled.

| | `14-28-49` | **this flight** |
| --- | --- | --- |
| heartbeat gaps > 2 s | 20 | **5** |
| max gap | 5.10 s | **4.52 s** |
| GCS failsafe | **triggered** | none |

**A 75% reduction in stalls.** Max gap now sits comfortably under the raised `FS_GCS_TIMEOUT` of
10 s, and `FS_OPTIONS = 16` means a stall could not seize control in a pilot-controlled mode
regardless.

## Decision

**`PSC_NE_POS_P` 0.5 → 0.3.**

Rationale: the trend has not turned. Every metric improved monotonically, position rms is still
0.135–0.186 m against a target below 0.13 m, and the residual peak persists. Cascade design wants
the inner loop **3–5× faster** than the outer:

```
PSC_NE_VEL_P = 1.0  ->  velocity crossover 0.159 Hz
PSC_NE_POS_P = 0.5  ->  2x separation   (current)
PSC_NE_POS_P = 0.3  ->  3.3x separation (textbook minimum)
PSC_NE_POS_P = 0.2  ->  5x separation   (textbook maximum)
```

**Falsifiable prediction:** peak moves to ~0.08 Hz, another 15–25% off rms and peak amplitude. If
rms rises instead, 0.5 was at or past the optimum and should be restored.

⚠ **The failure mode at low gain is drift, not oscillation.** Wind on this flight was negligible
(mean lean 0.74–1.14° on the previous sortie). A low gain will look good in calm air and drift in
a breeze, so the eventual value needs confirming on a windier day before it is adopted permanently.

## Confidence limits

- **"0.5 is better than 1.0": ~90%.** Non-overlapping ranges across four windows, effect 25–35%,
  scatter collapsed, mechanism moved as predicted.
- **"0.3 will be better still": ~60%.** Trend supports it; the minimum has not been located.
- **Telemetry fix: ~85%.** 20 → 5 is large, but conditions were not identical.
- **Wind robustness: untested.**

## Health

Zero clips. RTK Fixed throughout. No failsafes. Flight ended with a normal motor-emergency-stop
shutdown at 157.5 s.

## Next step

Fly `PSC_NE_POS_P = 0.3` with a **≥ 60 s continuous hands-off window**, same profile. Then evaluate
`EK3_SRC2_VELXY = 5` as a separate single variable.
