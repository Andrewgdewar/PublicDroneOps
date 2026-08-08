# Scout-15 — `PSC_NE_POS_P` 0.5 → 0.3 and Rangefinder Height Source

**Aircraft:** Scout-15 quadcopter
**Flight event:** 2026-08-07
**Flight log:** `2026-08-07 18-20-49.bin`
**Objective:** Continue the position-cascade gain reduction to 0.3, and replace the barometric
height source with the rangefinder to attack the 0.14–0.19 m vertical wander
**Outcome:** **Best hold on record on both axes.** Horizontal box 2.8× tighter, rms 2.4× tighter.
Vertical wander fell below the barometer's ability to measure it — an order of magnitude
improvement. Scorecard 17/17
**Decision:** **Adopt both.** Do not reduce `PSC_NE_POS_P` further until confirmed in wind
**Parameter snapshot:** `certification/parameters/0807.param`

## Summary

Two variables changed, on orthogonal axes, each testing an independent hypothesis: the horizontal
gain continuing a trend that had not turned, and the height source addressing a fault already
localised to the sensor rather than the loop.

Both worked. The horizontal result extends a clean monotonic trend. The vertical result is large
enough that the independent witness can no longer resolve the residual.

## Configuration proof — read from the log

| Item | Value |
| --- | --- |
| `PSC_NE_POS_P` | **0.3** — variable 1 |
| `EK3_SRC1_POSZ` | **2 — rangefinder** — variable 2 |
| `EK3_SRC2_POSZ` | 1 — baro, fallback set |
| `EK3_OGN_HGT_MASK` | **3** — new, GPS datum correction while off-GPS |
| `EK3_GPS_VACC_MAX` | 0.5 — inert while `POSZ = 2` |
| `EK3_RNG_M_NSE` / `EK3_ALT_M_NSE` | 0.1 / 2.0 |
| `EK3_RNG_USE_HGT` / `_SPD` | 70 / 2.0 — inert while `POSZ = 2` |
| `PSC_NE_VEL_P` / `_I` / `_D` | 1.0 / 0.5 / 0 — unchanged |
| `INS_GYRO_FILTER` | 27 |
| `ATC_ANG_RLL_P` / `PIT_P` | 6.0 / 7.0 |
| `ATC_RAT_RLL_P` / `_I` / `_D` | 0.160 / 0.160 / 0.0052 |
| `ATC_RAT_PIT_P` / `_I` / `_D` | 0.160 / 0.160 / 0.0051 |
| `EK3_SRC1_POSXY` / `VELXY` / `YAW` | 3 / 3 / 2 — RTK GPS, GPS yaw, no compass |
| `MOT_THST_HOVER` | 0.1668 |
| Mode | Loiter throughout |
| GPS | `HAcc` 0.040 m, `VAcc` 0.050 m median, `SAcc` 0.030 m |

## Flight profile

| Event | Time |
| --- | --- |
| Boot | 46.1 s |
| GCS failsafe (pre-arm), cleared | 88.6 → 95.6 s |
| Takeoff | ~103 s |
| Steady hover at 1.75 m | 111 → 148 s |
| Descent, landing | 150 → 161 s |
| Disarm | 164.1 s |
| `MotorEStop` HIGH | 166.3 s — post-disarm safing, not an event |

Armed 118.2 s. Roll and pitch step-response detected **zero** command reversals, confirming this
was a pure hover with no manoeuvre content — the intended profile.

## Result 1 — horizontal hold

Hands-off window, all four sticks centred including throttle, single flight mode, airborne.
`box` = larger of the N/E extents; `rms` = radial deviation about the window mean.

| `POS_P` | log | window | box | rms | spectral peak |
| --- | --- | --- | --- | --- | --- |
| 1.0 | `14-28-49` | — | 1.20 m | 0.236 m | 0.19–0.20 Hz |
| 0.5 | `16-01-25` | 51 s @ 1.75 m | 0.569 m | 0.158 m | 0.119 Hz |
| **0.3** | **`18-20-49`** | **28 s @ 1.75 m** | **0.206 m** | **0.065 m** | **0.106 Hz** |

Like-for-like at identical hover altitude: **box −64%, rms −59%.**

The spectral peak falls monotonically with gain — 0.20 → 0.119 → 0.106 Hz — which is the expected
signature of lowering the outer-loop crossover. Prediction before flight was ~0.08 Hz; the measured
0.106 Hz is the correct order and direction.

### ⚠ Spectral resolution limit — the quoted peak is bin-limited

A 28 s window gives an FFT resolution of 1/28 = **0.0357 Hz**. The reported position-spectrum peaks
at 0.035 / 0.071 / 0.106 / 0.142 Hz are **exactly bins 1, 2, 3 and 4**. No structure inside that
range is resolvable from this flight.

The honest statement is that horizontal motion is **broadband below ~0.15 Hz**. The downward trend
in peak frequency across the three flights is real as a statement about *where the energy sits*, but
the specific value 0.106 Hz should not be quoted as a measured mode frequency. Resolving it needs a
**≥ 60 s** continuous window.

### Confounds checked and controlled

| | `18-20-49` (0.3) | `16-01-25` (0.5) w1 | `16-01-25` (0.5) w2 |
| --- | --- | --- | --- |
| hover altitude | **1.75 m** | **1.75 m** | 2.50 m |
| hover throttle | 0.166 | 0.168 | 0.169 |
| **mean lean (wind proxy)** | **0.89°** | **0.81°** | 0.86° |

Altitude, loading and steady wind are matched between the two 1.75 m windows.

**Lean *variance* is not a usable wind proxy across different position gains** — a lower `POS_P`
inherently commands less lean for the same disturbance, so variance is partly an output of the
change under test. The **mean** (DC) lean is the steady tilt required to hold against wind, and it
is the same in all three windows.

## Result 2 — vertical hold

The height source changed, so the roles of the sensors swapped. **A sensor that is the active
height source is a controlled variable and cannot grade itself** — the loop drives its own sensor's
error toward zero regardless of true motion. Each flight is therefore graded by whichever sensor
was *not* in the loop.

| log | height source | independent witness | detrended rms | p2p |
| --- | --- | --- | --- | --- |
| `16-01-25` | baro | **rangefinder** | 0.0855 m | 0.603 m |
| **`18-20-49`** | rangefinder | **baro** | **0.0321 m** | 0.400 m |

Measured improvement in independently-observed vertical wander: **2.7×.** The true improvement is
larger, because the witness itself is now the limiting instrument.

### The residual is below the witness's noise floor

Barometer noise measured on a stationary post-landing segment is **0.030 m rms**. The detrended
baro reading during this hover is **0.0321 m** — essentially the instrument's own noise.

Deconvolving, assuming independent errors:

$$\sigma_{\text{true}} = \sqrt{0.0321^2 - 0.030^2} \approx 0.011\ \text{m}$$

That subtraction is between near-equal quantities and is therefore fragile. Stated conservatively:
**true vertical wander is ≤ 3.2 cm and most likely 1–2 cm.**

### Cross-validation between the two sensors

The rangefinder (controlled) reports 0.0216 m detrended. Adding the barometer's own noise in
quadrature predicts what an independent baro should see:

$$\sqrt{0.0216^2 + 0.030^2} = 0.037\ \text{m} \quad\text{vs measured } 0.0321\ \text{m}$$

The two witnesses agree within noise. This is an internal consistency check, not an independent
confirmation, but it rules out a gross error in either channel.

### The estimator became honest

| log | EKF-reported vertical rms | independent witness | EKF error |
| --- | --- | --- | --- |
| `16-01-25` | 0.0355 m | 0.0855 m | **under-reporting 2.4×** |
| `18-20-49` | 0.0175 m | ~0.011–0.032 m | consistent |

On the previous flight the EKF believed the aircraft was substantially steadier than two agreeing
sensors said it was. Fusing to a low-noise height source corrected the *estimate*, not only the
control. `EK3_RNG_M_NSE` 0.1 versus `EK3_ALT_M_NSE` 2.0 is a 20× difference in fusion weight.

## Result 3 — health and scorecard

**Scorecard 17/17 pass, 0 marginal, 0 fail.**

| Metric | Value |
| --- | --- |
| IMU clipping (delta) | **0** |
| VIBE median | 2.45 / 2.67 / 2.73 |
| VIBE max | 4.59 / 5.64 / 5.35 |
| `Dmod` min, all axes | 1.000 |
| Attitude err rms roll / pitch | 0.214° / 0.734° |
| roll `mode` / pitch `mode` | 0.24 / 0.19 |
| `PM.NLon` | 1 |
| `PM.Mem` min | 265256 |

Roll `mode` 0.24 and pitch 0.19 are the lowest recorded on this airframe. The 5.87 Hz roll
structural mode that dominated this aircraft through early August does not appear in this flight.
`INS_GYRO_FILTER = 27` has held.

### Rangefinder reliability

| Metric | Value |
| --- | --- |
| Samples, armed | 1992 |
| `Stat == 4` (GoodRange) overall | 58.9% |
| **`Stat == 4` during hover (106–158 s)** | **100%** |
| Distance range in hover | 0.83 – 2.13 m |

The 41% of non-good samples are entirely ground time, where the sensor reads below
`RNGFND1_MIN = 0.2 m` and correctly reports `OutOfRangeLow`. **Zero dropouts in flight.**

## Result 4 — oscillation survey, all axes

Hover window, FFT of rate-loop error.

| axis | err rms | out rms | out p95 | dominant band | share |
| --- | --- | --- | --- | --- | --- |
| RLL | 0.935 °/s | 0.0040 | 0.0119 | 5.64–5.89 Hz | ~15% |
| PIT | 0.854 °/s | 0.0050 | 0.0118 | 5.92–6.24 Hz | ~10% |
| **YAW** | **1.747 °/s** | 0.0053 | **0.0400** | **0.60–0.67 Hz** | **47%** |

### The 5.87 Hz airframe mode is still present, but no longer amplified

The structural mode that dominated this airframe through early August still appears in the roll rate
error. It carried **32.9% of roll gyro energy** when first characterised; it now sits at roughly 15%
of rate-error energy, and `RATE.ROut` p95 is **0.0149 against a 0.10 gate**.

A structural mode cannot be removed in software. The objective was to stop driving it, and
`INS_GYRO_FILTER = 27` has achieved that.

### Yaw is now the worst-behaved axis

Nearly half of all yaw rate-error energy sits in a **0.60–0.67 Hz limit cycle**. Yaw rate error rms
is 2× roll and pitch, and output p95 is 3.4× higher. The step response agrees — yaw was the only
axis with detectable events, showing **51.4% overshoot, 295 ms rise, 998 ms settle, ring 1.5**.

Heading itself is well controlled (sd 0.389°, p2p 0.850°), so this costs control authority rather
than accuracy. Not urgent, but it is now the largest remaining oscillatory feature on the aircraft.

Leading hypothesis: GPS moving-baseline yaw updates at 5 Hz (`GPS1_RATE_MS = 200`) driving
`ATC_ANG_YAW_P = 4.5`. Untested.

## Result 5 — power and propulsion

| Metric | Hover window |
| --- | --- |
| Pack voltage | 24.22 V = 4.036 V/cell (6S) |
| Current | 10.04 A |
| Power | 243 W |
| Consumed, whole flight | 859 mAh |
| **Hover endurance estimate** | **~43 min** @ 9000 mAh, 80% usable |
| Hover throttle | 0.167 → implied thrust-to-weight ≈ **3:1** |

### Lateral thrust asymmetry

| Motor | Position | RPM | ESC °C | PWM |
| --- | --- | --- | --- | --- |
| M1 | right-front | 2916 | 32.0 | 1396 |
| M2 | left-rear | 3299 | 37.1 | 1428 |
| M3 | left-front | 3256 | 28.9 | 1438 |
| M4 | right-rear | 2941 | 23.2 | 1399 |

**RPM spread 12.3%.** Torque pairs are balanced — M1+M2 vs M3+M4 differs by only **+0.3%**, improved
from 4.9% on `14-28-49` — so this is **not** a yaw-torque imbalance.

The split is **left versus right**: M2 and M3 (left) run ~11% higher RPM and 35 µs more PWM than M1
and M4 (right). That is the signature of a **lateral CG offset**, and it costs asymmetric roll
authority: the left motors reach saturation sooner in a hard left correction.

ESC temperatures follow the same positional pattern as previous flights rather than trending, so
this is a standing build characteristic, not a developing fault. **Action: check lateral CG.**

## Result 6 — optical flow assessment (logged, not fused)

`EK3_SRC2_VELXY = 0` — flow was recorded but never entered the filter. The sensor performed well:
**quality mean 169, min 158, zero samples below threshold**, scalers calibrated to 338/311.

Despite that, flow cannot contribute at this aircraft's hold quality:

| Quantity | Value |
| --- | --- |
| True velocity sd | 0.0311 m/s |
| Implied flow signal at 1.75 m AGL | 0.0177 rad/s |
| Measured flow noise sd | 0.0819 rad/s |
| **Signal-to-noise** | **0.216 — 4.6× more noise than signal** |
| Flow-derived velocity noise at 1.75 m | 0.143 m/s |
| RTK `SAcc` | **0.020 m/s** |

All **eight** axis sign/swap conventions were tested to exclude a convention error as the cause. The
best achievable correlation against EKF velocity was |r| = 0.33.

Flow-derived velocity noise scales as *flow noise × height*, so the deficit **worsens with altitude**
— roughly 0.82 m/s at 10 m AGL. Flow is 7× worse than RTK at its most favourable altitude and ~40×
worse at survey height.

**Conclusion: fusing optical flow would degrade this hold, not improve it.** Its value on this
aircraft is GPS-denied redundancy only. Keep it calibrated and unfused.

## Result 7 — control authority margin

| Metric | Value |
| --- | --- |
| `ATC_ANGLE_MAX` | 30° |
| Lean used — mean / p95 / max | 0.90° / 1.20° / 1.36° |
| **Authority unused at p95** | **96%** |
| Horizontal accel available at 30° | 5.66 m/s² |
| Horizontal accel commanded | 0.155 m/s² — **36× headroom** |

### Reducing `PSC_NE_POS_P` does not weaken steady wind rejection

| Loop | I-term mean |
| --- | --- |
| North | **+0.1024** |
| East | **−0.1115** |

The position-loop integrators sit at a steady non-zero value, holding against light steady wind.

In the cascade, steady state implies position error → 0, so `PSC_NE_POS_P` contributes **nothing** to
holding a constant wind — the lean is supplied by `PSC_NE_VEL_I`. Lowering `POS_P` therefore:

- leaves **steady wind holding unchanged**
- makes **gust recovery ~1.67× slower** in the linear region (1 m error commands 0.3 m/s return
  instead of 0.5 m/s); for larger displacements `sqrt_controller` becomes acceleration-limited and
  the difference narrows

Rough envelope estimate at an assumed 3 kg AUW and $C_dA \approx 0.1\ \mathrm{m^2}$: comfortable hold
at 15° lean ≈ **11 m/s (22 kt)**, saturating near 30° at ≈ **16 m/s (32 kt)**. Mass and drag area are
estimates rather than measurements — treat as ±40%.

**The aircraft is nowhere near authority-limited. The open risk is transient excursion in gusts, not
strength.**

## Result 8 — GCS failsafe recurrence

`GCS Failsafe` triggered at 88.6 s and cleared at 95.6 s, blocking arming with
`PreArm: GCS failsafe on` at 92.8 s.

This occurred **on the ground, before arming**, and the failsafe correctly prevented takeoff. It is
the same telemetry-congestion signature documented in
`2026-08-07_rtk-baseline-gcs-failsafe/`, and it recurred despite `FS_GCS_TIMEOUT = 10` and the
stream-rate reductions. The mitigation reduced but has not eliminated it.

Not a flight risk on this sortie. Remains open.

## Decision

**Adopt `PSC_NE_POS_P = 0.3`.**
**Adopt `EK3_SRC1_POSZ = 2`.**

Both effects are far larger than the ±20% fixed-gain scatter that governs this airframe, and for
the horizontal result the measured ranges of the two configurations do not overlap.

**Do not reduce `PSC_NE_POS_P` further before a wind-day confirmation.**

## Confidence and limits

- **Only one valid hands-off window on this flight**, so no within-flight scatter estimate is
  available. The prior flight gave 9% between windows; the effect measured here is 170–180%, so it
  survives that bar comfortably — but a second window would have been better evidence.
- **28 s window**, against the ≥ 15 s standard. Adequate, not generous.
- **Calm air throughout** — mean lean 0.89°. Every flight in this campaign has been in near-calm
  conditions. **At reduced position gain the failure mode is drift, not oscillation, and calm air
  cannot reveal it.** This is the single largest open risk in the horizontal result.
- **The vertical residual is not directly measured**, only bounded. Resolving below ~3 cm requires
  an independent witness better than the barometer.
- Two variables changed in one sortie. They act on orthogonal axes and each addressed a separately
  diagnosed fault, but a coupling effect cannot be formally excluded from this flight alone.

## Next step

1. **Confirm `PSC_NE_POS_P = 0.3` in wind.** This is the test that can say "stop", and no test so
   far has been able to. Steady-wind authority is not in doubt; **transient gust excursion is**.
2. **Fly ≥ 60 s continuous hands-off.** Two independent reasons: it yields a within-flight scatter
   estimate, and it halves the FFT bin width so the position spectrum peak becomes resolvable.
3. **Check lateral CG.** Left arm pair runs ~11% higher RPM than right.
4. **Investigate the 0.64 Hz yaw limit cycle.** Candidate single-variable tests: reduce
   `ATC_ANG_YAW_P` from 4.5, or raise the GPS yaw update rate from 5 Hz (`GPS1_RATE_MS = 200`).
5. Only then consider `PSC_NE_POS_P = 0.2`. Cascade separation at 0.3 is 3.3× against
   `PSC_NE_VEL_P = 1.0`; 0.2 would be 5×, still within normal practice.

**Closed out by this flight:** optical flow fusion. Measured SNR 0.216 at the most favourable
altitude means it cannot improve on RTK at any height. Do not revisit as a hold-quality measure —
only as GPS-denied redundancy.
