# Scout-15 — GPS 10 Hz (`GPS1_RATE_MS` 200 → 100)

**Aircraft:** Scout-15 quadcopter
**Flight event:** 2026-08-07
**Flight log:** `2026-08-07 19-32-36.bin`
**Objective:** Halve GPS moving-baseline latency to buy phase margin in the yaw angle loop, which
was limit-cycling on its own crossover
**Outcome:** **Adopted.** Yaw step overshoot 51.4% → 27.6%, ring 1.5 → 1.0, coherent limit cycle
largely eliminated. Zero cost — link at 18.6%, RTK unaffected. Position hold not resolvable
**Decision:** **Adopt `GPS1_RATE_MS = 100`.** Also corrects a serial port misidentification
**Parameter snapshot:** `certification/parameters/0807.param`

## Summary

Yaw carried 47% of its rate-error energy in a 0.60–0.67 Hz limit cycle sitting exactly on the yaw
angle-loop crossover (`ATC_ANG_YAW_P = 4.5` → 0.716 Hz). With no compass, heading comes entirely
from GPS moving baseline; at 5 Hz the zero-order-hold delay of T/2 = 100 ms costs **23° of phase at
0.64 Hz**. Doubling the rate halves that.

Unlike lowering `ATC_ANG_YAW_P`, this **removes lag rather than slowing the loop** — the two are
additive, so heading tracking speed is retained.

## ⚠ Preceding error — serial port misidentification

An earlier analysis labelled **SERIAL4 as the GPS port**. It is not.

```cpp
SerialProtocol_GPS      = 5,     // SERIAL1_PROTOCOL,5   <- GPS
SerialProtocol_Lidar360 = 11,    // SERIAL4_PROTOCOL,11  <- proximity lidar
SerialProtocol_RCIN     = 23,    // SERIAL5_PROTOCOL,23  <- RC
```

Consequences, all traceable to one unverified assumption:

- The 13766 B/s / 59.7% attributed to GPS was the **proximity lidar**
- The derived conclusion that 10 Hz would overrun the link at 119% was **wrong** — the GPS port was
  at 19.9% and 10 Hz would have been ~46% at the original 230400
- `SERIAL4_BAUD` was changed from 230400 to 460800, which **disabled the proximity sensor**

The lidar baud was restored to 230400. The UM982 had by then been reconfigured to 460800, so
`SERIAL1_BAUD` was set to 460 to match — harmless, and it leaves the GPS port at 18.6%.

**Lesson: `SERIALx_PROTOCOL` numbers must be resolved against `AP_SerialManager.h` before any
conclusion is drawn from port statistics.**

## Configuration proof — read from the log

| Item | Value |
| --- | --- |
| `GPS1_RATE_MS` | **100** — the single functional variable |
| `SERIAL1_BAUD` / `SERIAL4_BAUD` | **460 / 230** — GPS matched, lidar restored |
| `EK3_POSNE_M_NSE` | 0.5 — deliberately reverted to keep this test clean |
| `PSC_NE_POS_P` | 0.3 |
| `EK3_SRC1_POSZ` | 2 — rangefinder |
| `ATC_ANG_YAW_P` | 4.5 — unchanged |
| `INS_GYRO_FILTER` | 27 |
| `PRX1_TYPE` | 19 |

## Result 1 — the rate change took, and cost nothing

| Check | Result |
| --- | --- |
| GPS epoch rate | 5.00 → **10.00 Hz** |
| **`GPYW` / `UNIHEADINGA` rate** | **10.00 Hz — not capped below position rate** |
| RTK Fixed | **100%** after convergence |
| SERIAL1 GPS load | Rx 8550 B/s = **18.6%**, `RxDp` 0 |
| SERIAL4 proximity | Rx 13767 B/s = 59.8%, **2306 messages — restored** |
| SERIAL5 RC | Rx 2488 B/s = 24.9%, `RxDp` 0 |

The driver sends four streams at `rate_s` — `UNIHEADINGA`, `AGRICA`, `GNGGA`, `GNRMC` — via a
`FALLTHROUGH` in `AP_GPS_NMEA::send_config()`, so all four doubled.

**An apparent 94.1% RTK figure was boot convergence**, 65.5–81.4 s. Nothing exceeded 0.2 m HAcc
after 81.5 s. The receiver sustained 10 Hz moving-baseline heading without degradation.

## Result 2 — yaw improved

Step response, whole flight:

| | `18-20-49` (5 Hz) | `19-32-36` (10 Hz) |
| --- | --- | --- |
| overshoot | 51.4% | **27.6%** |
| ring | 1.5 | **1.0** |
| rise | 295 ms | 335 ms |
| **events** | 6 | **31** |

Hands-off hover, Loiter versus Loiter, wind controlled (mean lean 0.89° vs 0.61°):

| | 5 Hz | 10 Hz |
| --- | --- | --- |
| yaw err rms | 1.747 °/s | 1.849 °/s — within scatter |
| heading sd | 0.389° | 0.404° — within scatter |
| **0.55–0.75 Hz share** | **7.3%** | **0.2%** |

**The coherent limit cycle is essentially gone. Total yaw error is unchanged** — energy redistributed
broadband rather than reducing.

⚠ **Methodology correction.** An earlier figure of "47% in 0.60–0.67 Hz" used a 0.5 Hz low cutoff
for normalisation; the 7.3% / 0.2% pair above uses 0.2–5 Hz. **Only the matched pair is comparable.**
The 47% should not be quoted against them.

Confidence **~70%** — one window per configuration, and a PosHold window on the same flight gave
yaw err rms 4.090 against Loiter's 1.849, so within-flight scatter is 2.2×.

## Result 3 — position hold not resolvable

| window | mode | dur | box | rms |
| --- | --- | --- | --- | --- |
| `18-20-49` 5 Hz | Loiter | 28 s | 0.206 | 0.065 |
| `19-32-36` 10 Hz | PosHold | 14 s | 0.208 | 0.088 |
| `19-32-36` 10 Hz | Loiter | 14 s | 0.165 | 0.057 |

**The two 10 Hz windows straddle the 5 Hz result — 54% scatter between them.** Below the resolution
floor. Pilot assessment ("felt equivalent") agrees.

Windows were short and mode-split because the flight included deliberate manoeuvres and two mode
changes.

## Result 4 — attitude axes versus the previous best

Only flights with genuine manoeuvre content are comparable. `18-20-49` had `corr` 0.34 / 0.17 —
pure hover — and cannot be compared on `rms_err`.

| axis | `08-47-02` | `19-32-36` | |
| --- | --- | --- | --- |
| RLL rms_err | 3.086 | **2.900** | −6% |
| PIT rms_err | 4.455 | **3.579** | −20% |
| YAW ratio / corr | 0.81 / 0.89 | **0.84 / 0.92** | both improved |

Yaw `rms_err` rose 3.886 → 9.924, but that scales with commanded input amplitude; `ratio` and `corr`
are the amplitude-normalised metrics and both improved.

Control-effort margin, against a 0.10 gate:

| | roll p95 | pitch p95 | yaw p95 |
| --- | --- | --- | --- |
| `08-47-02` | **0.1273 — over gate** | 0.0350 | 0.0351 |
| `19-32-36` | **0.0251** | 0.0374 | **0.0696** |

`INS_GYRO_FILTER = 27` cut roll control effort roughly 5× versus the old baseline. **Yaw is now the
tightest axis at 1.4× margin.** Motor headroom 407 µs to `MOT_SPIN_MAX`.

## Result 5 — takeoff drift explained

Pilot reported drift in the hover.

| t | N range | E range | HAcc |
| --- | --- | --- | --- |
| **68 s** | **2.68 m** | 0.37 m | **1.170 m** |
| 88 s | 1.12 m | 0.25 m | 0.040 m |
| 108 s | **0.23 m** | **0.21 m** | 0.030 m |

Takeoff was at 68.6 s, **13 seconds before RTK converged at 81.4 s**. The drift was the position
estimate settling, not the tune.

**Operational rule adopted: wait ≥ 15 s after boot before arming.**

## Decision

**Adopt `GPS1_RATE_MS = 100`.** Yaw step response clearly improved, no measurable cost, RTK
unaffected, link at 18.6%.

`ATC_ANG_YAW_P = 4.5` **remains unchanged** — total yaw error did not fall, so the crossover is
still at 0.716 Hz and that lever stays available.

## Confidence and limits

- Yaw step-response improvement is well supported: 31 events versus 6.
- The limit-cycle reduction rests on **one window per configuration**, with 2.2× scatter observed
  between two windows on this flight. ~70% confidence.
- Position hold is **not** resolvable from this flight; windows were 14 s and mode-split.
- Vibration rose (see `2026-08-07_ek3-posne-mnse-rejected/` for the full four-flight trend — a step
  change in spikes rather than level).

## Next step

Re-test position hold with a single-mode, ≥ 60 s hands-off window taken after RTK convergence.
