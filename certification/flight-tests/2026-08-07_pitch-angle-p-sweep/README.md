# Scout-15 Pitch Angle-P Sweep — `ATC_ANG_PIT_P` 4.5 → 7.5

**Aircraft:** Scout-15 quadcopter
**Flight event:** 2026-08-07
**Flight log:** `2026-08-07 10-50-16.bin`
**Objective:** Sweep the pitch angle-P gain, which had never been swept and sat at the 4.5 firmware
default while roll flew at 6.0
**Outcome:** **Partial.** 4.5 is measurably worse than the 6.5–7.5 group. Within that group nothing
is resolvable — the spread is smaller than the scatter
**Decision:** **Adopt `ATC_ANG_PIT_P = 7.0`.** No oscillation boundary was found; the sweep ran out
of resolution before it ran out of stability
**Parameter snapshot:** `certification/parameters/0807.param`

## Summary

First flight at `INS_GYRO_FILTER = 27`, so it doubles as the confirmation sortie for the gyro
filter adoption. That confirmation is the stronger of the two results.

The pitch sweep itself found what the roll angle-P sweep found: **every tracking metric improves
with gain right up to the point where measurement resolution runs out.** It never reached an
oscillation boundary. `ring` was **0.0 at every single setting including 7.5**, which is the top of
the ladder rather than the top of the aircraft's tolerance.

## Configuration proof — read from the log

| Item | Value |
| --- | --- |
| Firmware | ArduCopter 4.8.0-dev, commit `32e43699` |
| `INS_GYRO_FILTER` | **27** — first flight at the adopted value |
| Rate PIDs | RLL 0.160/0.160/0.0052, PIT 0.160/0.160/0.0051 — unchanged |
| `ATC_ANG_RLL_P` | 6.0 — unchanged, unswept, used as control |
| `ATC_ANG_PIT_P` | **4.5 at boot**, swept in flight by script |
| `ATC_RAT_*_SMAX` | 12 |
| `FILT1_TYPE` | 0 — PID notch off |
| `PSC_D_ACC_P` / `_I` | 0.017 / 0.034 |
| `INS_RAW_LOG_OPT` | 9 |

Script: `certification/scripts/angle-p-step.lua`, `AXIS="PIT"`, RC9 (301) +0.5, RC10 (302) −0.5,
clamped 3.0–9.0. Stream `PAPS` segmented by `pidreview.py segments`.

## Result — the gyro filter confirmation

Roll was **not swept** on this flight, so its windows are a clean like-for-like check of
`INS_GYRO_FILTER = 27` against the 18 Hz baseline.

| | 18 Hz (08-47-02) | **27 Hz (this flight)** |
| --- | --- | --- |
| roll `ring` | 2.5 | **1.0** |
| roll `mode` | 1.48 | **0.20** |
| roll `lean` | 3.0 | 6.3 |
| roll `mode/lean` | 0.49 | **0.032** |

Normalised for manoeuvre intensity — which is mandatory, see the `lean` trap — the airframe mode
carries **15× less energy** than it did at 18 Hz. Rise time 75 ms.

**This is the result that matters from this sortie.** The gyro filter adoption is confirmed on an
axis that was not being manipulated.

## Result — the pitch sweep

| `angP` | events | `attRMS/lean` | `mode` | `lean` | `ring` |
| --- | --- | --- | --- | --- | --- |
| 4.5 | 15 | 0.249 | 0.18 | 5.1 | 0.0 |
| 5.0 | 6 | 0.195 | 0.51 | 7.8 | 0.0 |
| 6.0 | 3 | 0.146 | 0.76 | 9.1 | 0.0 |
| 6.5 | 9 | 0.157 | 0.81 | 9.6 | 0.0 |
| **7.0** | 4 | **0.125** | 0.82 | 9.3 | 0.0 |
| 7.5 | 8 | 0.128 | 0.76 | 11.1 | 0.0 |
| 4.5 (return) | 3 | 0.196 | 0.21 | 2.6 | 0.0 |

### What is and is not resolvable

**The two 4.5 windows — identical gain — differ by 27%** (0.249 vs 0.196). That is the resolution
floor of this airframe, and it is what every other comparison has to beat.

- **4.5 vs the 6.5–7.5 group: −44%.** Comfortably outside 27%. **Resolvable.**
- **7.0 vs 7.5: 2.4%.** An order of magnitude inside the scatter. **Not resolvable.**
- **6.0 vs 7.0: 14%.** Inside the scatter. **Not resolvable.**

The only defensible statement this flight supports is: *4.5 is worse than the 6.5–7.5 group, and
within that group the aircraft does not care.* 7.0 was selected from the indistinguishable set.

The 6.0 and 7.0 windows carry only **3 and 4 events**. Below the ~6-event threshold those rows are
indicative only.

## Decision

**`ATC_ANG_PIT_P = 7.0` adopted.** Pitch and roll are now 7.0 / 6.0 rather than 4.5 / 6.0.

**No stability boundary was found.** `ring` stayed at 0.0 through 7.5 and `mode/lean` did not turn
upward. The ladder's `MAX_P` was subsequently raised to 9.0 for a boundary hunt, which has not
been flown. Roll's own angle-P sweep **rejected 8.0** — roll improved 21% but pitch control
degraded 44%, `ring` hit 4.0 — so the pitch boundary is expected somewhere in 8.0–9.0 at ~65%
confidence, and is unmeasured.

## Confidence limits

- **Gyro filter confirmation: ~90%.** Measured on an unswept axis, 15× is far outside any scatter.
- **"4.5 is worse": ~85%.** −44% against a 27% floor.
- **"7.0 is the right value": ~40%.** It is *a* value from an indistinguishable group. 6.5 or 7.5
  would have been equally defensible on this data.
- **No claim is made about the boundary.**

## Health

Zero clips in every window. VIBE median **flat at 4.65–5.54** across the whole ladder — angle-P is
an outer-loop gain and does not touch the rate loop's noise path, which is exactly as expected and
a useful negative control.

`scorecard`: 14/17 pass, 2 marginal, **1 fail** — pitch attitude error RMS 1.246° against a 1.0°
gate. That gate is inherited from the acceptance criteria and is intensity-sensitive; this was an
aggressive sweep flight with `lean` up to 11.1°, so it is not comparable to a hover.

## Sensor noise, measured incidentally

Static-hover segments of this log gave the cleanest sensor noise figures on record:

| | static hover | whole flight incl. manoeuvring |
| --- | --- | --- |
| rangefinder | **0.5 cm** | 8.3 cm |
| barometer | **2.2 cm** | 10.7 cm |

Whole-flight drift: rangefinder **+3 cm**, barometer **−222 cm** — 21× its own noise. This is the
first clean measurement of the baro drift that later flights investigated directly.

## Next step

Boundary hunt on pitch angle-P (`7.0 → 9.0`) remains unflown and is low priority — the aircraft
flies well and the gain is in an indistinguishable band. Superseded in the queue by navigation
work.
