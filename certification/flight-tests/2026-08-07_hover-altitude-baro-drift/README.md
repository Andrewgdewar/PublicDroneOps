# Scout-15 Hover Assessment — Barometer Thermal Drift and the Loiter Wobble

**Aircraft:** Scout-15 quadcopter
**Flight event:** 2026-08-07
**Flight log:** `2026-08-07 11-36-20.bin`
**Objective:** Assess hover quality after the gyro-filter and angle-P adoptions, and evaluate
`EK3_RNG_M_NSE = 0.1`
**Outcome:** **The `EK3_RNG_M_NSE` change did not load** — the log shows 0.5. The flight instead
identified two unrelated real problems: **barometer thermal drift** and **an over-gained horizontal
velocity loop**
**Decision:** Return `PSC_NE_VEL_*` to firmware defaults. Refly `EK3_RNG_M_NSE`
**Parameter snapshot:** `certification/parameters/0807.param`

## Summary

A quiet hover flight that was intended to confirm the attitude tune and ended up diagnosing the
navigation stack. Armed 134.1 s, Loiter throughout.

Two findings, both actionable, neither anticipated:

1. **The barometer drifts thermally**, and because `EK3_SRC1_POSZ = 1` (Baro) the aircraft
   physically follows it.
2. **The position controller is commanding the horizontal wobble** — it is not the airframe
   responding to wind.

## Configuration proof — read from the log

| Item | Value |
| --- | --- |
| Firmware | ArduCopter 4.8.0-dev, commit `32e43699` |
| `INS_GYRO_FILTER` | 27 |
| `ATC_ANG_RLL_P` / `PIT_P` | 6.0 / 7.0 |
| `EK3_RNG_M_NSE` | **0.5 — the intended 0.1 did not load** |
| `EK3_SRC1_POSZ` | 1 (Baro) |
| `EK3_RNG_USE_HGT` / `_SPD` | 70 / 2 |
| `PSC_NE_POS_P` | 1.0 |
| `PSC_NE_VEL_P` / `_I` / `_D` | **1.5 / 1.0 / 0.25** |
| `RNGFND1_TYPE` / `_MIN` / `_MAX` | 24 / 0.2 / 20 |
| Armed | 134.1 s, one span, Loiter |

**Process failure worth recording:** the parameter change was made but never verified as loaded
before flight. The debrief procedure now reads config from the log *first*, every time, before any
interpretation.

## Finding 1 — the position controller commands the wobble

The documented diagnostic for over-high Loiter gains is oscillation in the **desired** roll and
pitch angles. Measured over the 55–130 s hover window:

| | rms | p2p | dominant |
| --- | --- | --- | --- |
| `DesRoll` | 1.33° | 9.69° | **0.24 Hz** (4.2 s) |
| `DesPitch` | 1.50° | 10.82° | **0.23 Hz** (4.4 s) |
| `Roll` | 1.29° | 9.16° | 0.24 Hz |
| `Pitch` | 1.49° | 11.05° | 0.23 Hz |

**Actual tracks desired almost exactly** — same frequency, same amplitude, rms within 3%. The
airframe is doing precisely what it is told, and it is being told to wobble ±5°.

The cause is that the horizontal velocity gains sit **above** firmware defaults:

| param | copter default | as flown | |
| --- | --- | --- | --- |
| `PSC_NE_POS_P` | 1.0 | 1.0 | ✓ |
| `PSC_NE_VEL_P` | **1.0** | **1.5** | 1.5× |
| `PSC_NE_VEL_I` | **0.5** | **1.0** | **2×** |
| `PSC_NE_VEL_D` | **0.0** | **0.25** | **D where stock has none** |

Defaults read from `AC_PosControl.cpp` (`POSCONTROL_NE_VEL_P/I/D`), not from memory.

Horizontal behaviour over the same window: path length **11.83 m**, median speed 0.103 m/s, max
0.830 m/s. Velocity tracking correlation was **+0.15 north / +0.05 east** — essentially zero, with
error rms (0.187 / 0.198) *exceeding* target rms (0.123 / 0.126).

## Finding 2 — barometer thermal drift

| | value |
| --- | --- |
| baro temperature | 39.89 → 34.69 °C over the flight |
| correlation, altitude vs temperature | **r = −0.595** |
| drift 0–40 s | +71 cm |
| drift 40–80 s | +134 cm |
| drift 80–120 s | −2 cm |

The drift is largest early and settles as the sensor equilibrates. Because `EK3_SRC1_POSZ = 1`, the
EKF follows the barometer and the aircraft follows the EKF — this is **real physical motion**, not
an estimation artefact.

Rangefinder health over the same period: **0 dropouts in 1142 samples.**

## Withdrawn conclusion

An earlier reading of this log compared the EKF altitude swing (0.31 m) against the rangefinder
swing (1.35 m) and concluded the rangefinder was the noisier sensor.

**That reasoning is circular and the conclusion is withdrawn.** EKF altitude is the *controlled*
variable — the controller actively drives it toward target, so its low variance is a measure of
control-loop performance, not of sensor quality. It cannot be used as an independent reference for
judging the sensor that feeds it.

Judged against the rangefinder as truth, the barometer captured 60% of true motion (25 cm RMS
error) and the **EKF only 18%** (49 cm RMS error).

## Attitude, for the record

| axis | rms err | corr |
| --- | --- | --- |
| roll | 3.362 | 0.45 |
| pitch | 5.387 | 0.23 |
| yaw | 7.059 | 0.96 |

Low correlations reflect a near-motionless hover — with almost no commanded demand there is little
signal to correlate against. **A hover log cannot assess rate-loop tracking**; those numbers are
reported for completeness, not for comparison against manoeuvring flights.

Altitude hold error rms 0.071 m, p2p 0.53 m over the full armed period.

## Decision

1. **`PSC_NE_VEL_P` 1.5 → 1.0, `_I` 1.0 → 0.5, `_D` 0.25 → 0.0** — back to firmware defaults.
2. **Refly `EK3_RNG_M_NSE = 0.1`**, verified loaded from the log before interpretation.
3. **No parameter fixes barometer thermal bias.** Options are to treat the first ~90 s as warm-up,
   minimise powered ground time, or move `EK3_SRC1_POSZ` off Baro — the last carries a BVLOS
   trade-off and was not taken.

## Confidence limits

- **"The controller commands the wobble": ~90%.** Desired and actual angles agree to 3% in
  amplitude and share a frequency; that is not a wind response.
- **"Gains are above stock": 100%.** Read from firmware source.
- **"Defaults will fix it": ~50%.** The mechanism is right; the magnitude is untested.
- **Baro thermal drift: ~85%.** r = −0.595 over one flight; the settling pattern is consistent with
  thermal equilibration but a single flight cannot exclude other causes.

## Health

Zero clips. Rangefinder 0 dropouts in 1142 samples.

## Next step

Refly with the stock velocity gains and a verified `EK3_RNG_M_NSE = 0.1`.
