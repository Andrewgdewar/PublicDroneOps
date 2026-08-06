# Scout-15 Roll D In-Flight Sweep Review

**Aircraft:** Scout-15 quadcopter
**Flights:** 2026-08-03
**Flight logs:** `2026-08-03 13-24-01.bin` (baseline), `2026-08-03 15-03-42.bin` (sweep)
**Objective:** Locate roll rate D by in-flight sweep rather than flight-to-flight comparison
**Outcome:** `ATC_RAT_RLL_D` 0.00465 → **0.0052** adopted
**Method note:** First use of a Lua rate-gain stepper with a dedicated log stream

## Configuration proof

| Item | Value |
| --- | --- |
| Firmware | ArduCopter 4.8.0-dev, commit `32e43699` |
| Frame | 580 mm Quad-X, carbon tube arms |
| Flight controller | Matek H743-SLIM |
| Motors / props | Angel X4108 380 KV, T-Motor MS1504 15x5.6 |
| Battery | 6S2P Li-ion |
| Gyro low-pass | `INS_GYRO_FILTER=18` |
| Harmonic notch | `INS_HNTCH_OPTS=0`, `HMNCS=3`, `BW=15`, `MODE=3` (ESC telem) |
| Roll / pitch P/I | `0.150` both axes, held constant |
| Pitch D | `0.0051`, held constant |

Baseline flight 13-24-01 flew roll D `0.00465`: roll rms error 4.707 deg/s, corr 0.74;
pitch rms error 5.084 deg/s, corr 0.67. Zero clips, `Dmod` 1.00, `Dlim%` 0.0.

## Method

`roll-d-step.lua` on RC options 301 / 302 stepped `ATC_RAT_RLL_D` by ±0.0001 in flight.
The script calls `Parameter:set()`, which is runtime-only and **emits no `PARM` record**, so the
value was written to a custom `RLDS` log stream. `tools/logtools/pidreview.py` was extended to read
that stream (`SCRIPTED_PARAM_STREAMS`); without it the entire flight collapses to a single segment
at the boot value.

## Sweep result — `pidreview.py segments RLL`

| start_s | dur_s | D | ev | over% | rise | ring |
| --- | --- | --- | --- | --- | --- | --- |
| 61.4 | 11.3 | 0.00465 | 0 | – | – | – |
| 79.0 | 14.3 | 0.00510 | 7 | 29.5 | 80 | 1.0 |
| 94.3 | **66.7** | **0.00520** | 7 | **27.3** | 137 | **1.0** |
| 165.5 | 11.2 | 0.00510 | 0 | – | – | – |
| 192.5 | 26.9 | 0.00510 | 5 | 19.2 | 93 | 2.0 |
| 221.5 | 21.9 | 0.00530 | 4 | 32.9 | 120 | 2.5 |

## Decision

`ATC_RAT_RLL_D = 0.0052` adopted. It gave the lowest overshoot at ring 1.0 over the longest
window, and 0.0053 degraded on both metrics.

## Resolution limit — recorded honestly

The sweep **cannot separate 0.0051 from 0.0052**. Two windows at the *same* D of 0.00510 returned
over% 29.5 / ring 1.0 and over% 19.2 / ring 2.0. Scatter at a single gain therefore exceeds the
spread between adjacent gains, and the chosen 0.0052 value (over% 27.3) sits inside that band.

Only 0.0053 stands clearly apart, and that rests on 4 events.

Step size was ~1.9% of the gain. **Too fine to resolve.** Subsequent P/I sweeps used 6.7% steps,
which resolved cleanly.

## Health and telemetry

Zero accelerometer clips. `Dmod` 1.00 and `Dlim%` 0.0 in every segment — the D-term slew limiter
never engaged, so D noise was not a constraint at any swept value.

## Next step

Two windows returned zero events because they were flown as pure hover. Command reversals on the
swept axis are required to generate step events; subsequent sweeps held each gain for at least
15 s with continuous reversals.
