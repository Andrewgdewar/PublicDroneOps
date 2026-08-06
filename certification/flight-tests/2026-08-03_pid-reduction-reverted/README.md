# Scout-15 Rate PID Reduction Trial and Revert

**Aircraft:** Scout-15 quadcopter
**Flights:** 2026-08-03
**Flight logs:** `2026-08-03 15-44-49.bin` (reduced), `2026-08-03 15-58-49.bin` (reverted)
**Objective:** Test the common "back off the rate PIDs for a smoother tune" heuristic
**Outcome:** **Reverted.** No benefit demonstrated; result was inconclusive, not negative
**Status:** Heuristic recorded as not applicable to this airframe

## Configuration proof

| Item | 15-44-49 (reduced) | 15-58-49 (reverted) |
| --- | --- | --- |
| `ATC_RAT_RLL_P` / `_I` | 0.13215 | 0.150 |
| `ATC_RAT_RLL_D` | 0.004581 | 0.0052 |
| `ATC_RAT_PIT_P` / `_I` | 0.13050 | 0.150 |
| `ATC_RAT_PIT_D` | 0.004437 | 0.0051 |
| `INS_GYRO_FILTER` | 18 | 18 |
| `INS_HNTCH_OPTS` / `HMNCS` | 0 / 3 | 0 / 3 |
| Armed time | 92.6 s | 118.6 s |

Reduction was approximately −13% on pitch and −11.9% on roll, applied to P, I and D together.

## Result

| | Roll rms err | Roll corr | Pitch rms err | Pitch corr |
| --- | --- | --- | --- | --- |
| Reduced (15-44) | 5.622 | 0.87 | 7.194 | 0.73 |
| Reverted (15-58) | 4.929 | 0.78 | 8.813 | 0.67 |

Event-based step response, `pidreview.py stepresponse`:

| Log | Pitch P / D | PIT over% | Roll D | RLL over% |
| --- | --- | --- | --- | --- |
| 15-03-42 baseline | 0.150 / 0.0051 | 32.5 | swept | 28.1 |
| 15-44-49 reduced | 0.131 / 0.0044 | **45.2** | 0.0046 | 28.8 |
| 15-58-49 reverted | 0.150 / 0.0051 | **38.6** | 0.0052 | 32.9 |

## Decision

Reverted to `0.150 / 0.150` P/I on both axes with roll D 0.0052 and pitch D 0.0051.

## Why the result is inconclusive rather than negative

Pitch overshoot did rise under reduction (32.5 → 45.2%). However the revert returned **38.6%**,
not the original 32.5%, on **identical gains**. Flight-to-flight scatter at fixed gains is therefore
roughly ±20% relative — comparable to the effect being measured.

An earlier claim that overshoot "doubled" under reduction was produced by an ad-hoc analysis script
using a manoeuvre-detection threshold below the noise floor, and is **withdrawn**. That script was
deleted; all subsequent analysis uses `tools/logtools/pidreview.py`.

## Method conclusion adopted after this trial

**Whole-flight metrics are not comparable between flights.** A gain change is only assessable by an
in-flight sweep segmented with `pidreview.py segments`, where every window shares the same airframe,
air and sortie. Comparing two ordinary flights cannot resolve changes of this size.

## Health and telemetry

Zero accelerometer clips in both flights. `Dmod` 1.00, `Dlim%` 0.0 throughout.

## Next step

Adopt in-flight sweeps for all subsequent gain work. See
[`../2026-08-04_roll-pi-sweep-review/README.md`](../2026-08-04_roll-pi-sweep-review/README.md).
