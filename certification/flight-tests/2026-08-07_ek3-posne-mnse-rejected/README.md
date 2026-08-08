# Scout-15 — `EK3_POSNE_M_NSE` 0.5 → 0.1 — **Rejected**

**Aircraft:** Scout-15 quadcopter
**Flight event:** 2026-08-07
**Flight log:** `2026-08-07 20-02-03.bin`
**Objective:** Test whether the EKF's position-noise floor was limiting hold quality by discarding
RTK precision, and whether tightening it would restore position-loop phase margin
**Outcome:** **Falsified.** Hold degraded by 55% rms / 85% box against the baseline, in *calmer*
air, with only 7% within-flight scatter. The filter was correct to de-weight GPS
**Decision:** **Revert to 0.5.** A second hypothesis — that estimator lag caused the 0.19 Hz
position oscillation — dies with it
**Parameter snapshot:** `certification/parameters/0807.param`

## Summary

The EKF floors its assumed GPS position noise at `EK3_POSNE_M_NSE` regardless of what the receiver
reports. With RTK reporting 0.030–0.040 m and the floor at 0.5, the filter was being told the
measurement was 12–16× worse than it actually was. Lowering the floor to 0.1 was expected to tighten
the position estimate and reduce loop lag.

**It made the hold worse, decisively.** With RTK HAcc at 0.03 m and the aircraft holding to 0.065 m
rms, GPS noise is a large fraction of the motion being controlled. Trusting it more simply fed that
noise into the loop.

This was the best-structured test flight of the campaign, which is why the result is trustworthy.

## Configuration proof — read from the log

| Item | Value |
| --- | --- |
| `EK3_POSNE_M_NSE` | **0.1** — the single variable |
| `EK3_VELNE_M_NSE` | 0.3 — unchanged |
| `PSC_NE_POS_P` | 0.3 — unchanged |
| `PSC_NE_VEL_P` / `_I` | 1.0 / 0.5 — unchanged |
| `EK3_SRC1_POSZ` | 2 — rangefinder |
| `EK3_SRC1_POSXY` / `VELXY` / `YAW` | 3 / 3 / 2 |
| `EK3_OGN_HGT_MASK` | 3 |
| `GPS1_RATE_MS` | **100** — adopted from `19-32-36` |
| `SERIAL1_BAUD` / `SERIAL4_BAUD` | 460 / 230 |
| `INS_GYRO_FILTER` | 27 |
| `ATC_ANG_RLL_P` / `PIT_P` / `YAW_P` | 6.0 / 7.0 / 4.5 |
| `MOT_THST_HOVER` | 0.1679 |

## Flight quality — the cleanest test conditions achieved so far

| Criterion | Result |
| --- | --- |
| RTK Fixed | **100%, zero samples above 0.2 m HAcc** |
| RTK converged before takeoff | **yes** — takeoff 63.3 s, no degraded samples at any point |
| Flight mode | **Loiter throughout, zero mode changes** |
| Continuous hands-off window | **56 s** |
| IMU clipping | 0 |

The three defects that weakened previous comparisons — mid-window mode changes, takeoff before RTK
convergence, and sub-30 s windows — were all absent.

## Result — the change is rejected

**Windows must be equal length.** A longer window captures more low-frequency wander and inflates
both `box` and `rms`, so the 56 s window was split into two 28 s halves to match the baseline.

| | alt | **mean lean (wind)** | box | rms |
| --- | --- | --- | --- | --- |
| `18-20-49` `POSNE` **0.5** — baseline | 1.75 m | 0.89° | **0.206 m** | **0.065 m** |
| `20-02-03` `POSNE` **0.1** — first 28 s | 2.01 m | 0.66° | 0.382 m | 0.105 m |
| `20-02-03` `POSNE` **0.1** — last 28 s | 1.98 m | 0.56° | 0.376 m | 0.098 m |

**rms +55%, box +85%.**

### Why this result is trustworthy

1. **Within-flight scatter is 7%** (rms 0.105 vs 0.098) — the tightest recorded on this aircraft, and
   far below the effect size. Previous flights ran 9–78%.
2. **The ranges do not overlap** the baseline.
3. **The wind was lower on the losing flight** — mean lean 0.56–0.66° versus 0.89°. Conditions
   favoured the new configuration and it still lost.

Altitude differed by 0.24 m (1.99 vs 1.75). That is the one uncontrolled variable, and it is small
relative to an 85% effect.

### Mechanism

```cpp
R_OBS[3] = sq(constrain_ftype(gpsPosAccuracy, frontend->_gpsHorizPosNoise, 100.0f));
```

RTK HAcc is 0.030 m. The aircraft holds to 0.065 m rms. **GPS measurement noise is therefore
roughly half the amplitude of the motion under control.** Lowering the floor from 0.5 to 0.1 told
the filter to follow that noise, and the controller dutifully chased it.

The default floor is not a bug. It is the filter correctly refusing to differentiate a signal whose
noise is comparable to the state being estimated.

## A second hypothesis dies with it

Prior to this flight it was proposed that the 0.19–0.20 Hz position oscillation — the one that drove
`PSC_NE_POS_P` from 1.0 down to 0.3 — might have been caused by **position-estimate lag** rather than
excessive gain, and that fixing the estimator would permit `POS_P` to return to its documented
0.50–4.00 range.

**Tightening the estimator made hold worse, so estimator lag cannot have been the limiting factor.**

`PSC_NE_POS_P = 0.3` was the correct value for the correct reason. Any future increase in hover
stiffness must come from raising the **inner** loop (`PSC_NE_VEL_P`, currently at its 1.0 default
against a documented range of 0.10–10.00), not from the estimator.

## Behaviour when RTK is lost — measured, not assumed

Compared against `12-17-52`, a 3D-fix-only flight:

| | RTK Fixed | 3D only | change |
| --- | --- | --- | --- |
| GPS yaw valid | 99.1% | **97.7%** | none meaningful |
| **`YAcc` heading accuracy** | 1.933° | **2.017°** | **none** |
| **EKF velocity noise used** | 0.300 m/s | **0.300 m/s** | **identical** |
| EKF position noise used | 0.500 m | **1.660 m** | 3.3× de-weight, automatic |
| Height source | rangefinder | rangefinder | unaffected |

**Heading survives RTK loss.** With no compass this was the material risk. Moving-baseline yaw
resolves carrier phase between the two *onboard* antennas and does not require base-station
corrections — `YAcc` is unchanged.

**Velocity fusion is identical.** `SAcc` is 0.030 with RTK and 0.040 without; both sit below the
`EK3_VELNE_M_NSE = 0.3` floor, so the filter uses 0.300 either way.

**Position de-weights automatically** because the parameter is a floor, not an override.

At matched gain, the no-RTK flight actually held marginally better (`12-17-52` box 1.05 / rms 0.163
versus `14-28-49` box 1.20 / rms 0.236 at `POS_P` 1.0). **Nothing in the current tune is
RTK-dependent for flight quality.** RTK earns its place in absolute accuracy — geo-referencing, RTL
precision, waypoint placement — not in hold.

## Health

**Scorecard 16/17 pass, 1 marginal, 0 fail.**

| Metric | Value |
| --- | --- |
| IMU clipping | 0 |
| **VIBE max** | **16.79 — MARGINAL, gate is 15** |
| `Dmod` min, all axes | 1.000 |
| `RATE.ROut` / `POut` / `YOut` p95 | 0.0277 / 0.0204 / 0.0589 (gate 0.10) |
| Attitude err rms roll / pitch | 0.452° / 0.384° |

### Vibration — no change once robust statistics are used

| flight | hover max | **hover p95** | **hover median** | manoeuvre p95 |
| --- | --- | --- | --- | --- |
| `08-47-02` | 5.65 | 4.23 | 2.84 | 3.95 |
| `18-20-49` | 5.64 | 4.21 | 3.19 | — (no manoeuvres) |
| `19-32-36` | 10.92 | 4.55 | 3.10 | 8.37 |
| `20-02-03` | 11.67 | 4.31 | 3.19 | **10.73** |
| `20-42-17` | 7.11 | 3.84 | 2.84 | 5.08 |

**Hover p95 and median are flat across all five flights.** Only the *maximum* varied, and maximum is
a rare-spike statistic that also grows with sample count — hover sample counts ranged 377 to 1515.

The two flights with elevated maxima are the two with the heaviest manoeuvring, and their
**manoeuvre** p95 (8.37 and 10.73) is where those maxima originate. Zero clips on every flight.

> **Withdrawn.** An earlier reading of this data — "hover vibration doubled, inspect the airframe" —
> was wrong. It quoted the maximum while ignoring p95 and median, which is the exact error warned
> against in `tuning.md` §9.0. **There is no evidence of a vibration change.** The `20-02-03`
> scorecard MARGINAL on VIBE max 16.79 reflects manoeuvre loading, not airframe condition.

## Decision

**Revert `EK3_POSNE_M_NSE` to 0.5.**

`18-20-49` remains the best flight on record — box 0.206 m, rms 0.065 m, vertical residual at the
barometer noise floor.

## Confidence and limits

- The rejection is **high confidence**. Effect size 55–85% against 7% within-flight scatter, with
  non-overlapping ranges and wind favouring the rejected configuration.
- **`EK3_VELNE_M_NSE` was not tested.** It remains at 0.3 and is floored 100% of the time, but the
  measured `SAcc` maximum of 0.230 m/s nearly reaches that floor, so the available gain is far
  smaller than it was for position — and this flight suggests the direction is wrong regardless.
- Altitude differed by 0.24 m between the compared windows.
- One flight per configuration. The within-flight split is strong evidence but is not the same as a
  repeat sortie.

## Next step

**Raise the inner loop, not the outer.** `PSC_NE_VEL_P` has never been moved from its 1.0 default
against a documented range of 0.10–10.00. It sets velocity-loop bandwidth, which is what limits how
hard the aircraft can hold without the position loop losing phase margin — and it is the only
untouched path to a stiffer hover.
