# Scout-15 Short-Hop Hover and COIN-D6 Review

**Date:** 2026-07-24

**Log:** `2026-07-24 17-03-54.bin`

**Firmware:** ArduCopter V4.8.0-dev (`32e43699`), MatekH743-bdshot

**Purpose:** Verify the rewired COIN-D6 supply/UART, evaluate yaw expo, and
investigate the pilot-reported busy or unstable hover in approximately 32 C weather
with wind.

## Summary

The flight is a **conditional pass**. There were no EKF, logging, scheduler, CAN,
UART, motor, battery, or accelerometer-clipping failures. The COIN-D6 remained healthy
at 10 Hz throughout both sorties with no UART drops and no correlation between its
closest return and throttle. The yaw change felt good and produced stable tracking.

The hover was measurably busier than the July 22 calm-hover reference, but it did not
drift away or exhibit runaway PID hunting. During hands-off Loiter windows, horizontal
position error was 0.17-0.20 m RMS and 0.43-0.44 m maximum. Desired tilt moved several
degrees at low frequency, showing that Loiter was actively correcting wind. A known
5-6 Hz roll/pitch component was also present and stronger than in the calm reference,
but VIBE remained below the 30 m/s2 gate and neither PID used D limiting.

One configuration issue was identified:

1. `LOIT_OPTIONS=1` enabled coordinated turns, deliberately adding lateral
   acceleration during yaw when horizontal velocity was present. It is changed to
   `0` to remove the reported roll-with-yaw behavior.

This flight does not prove that any rate PID caused the reported hover behavior. Do
not change roll/pitch P/I/D or Loiter position gains from this evidence alone. A PID
change requires a controlled, matched A/B flight that changes one value while holding
weather, maneuver, filter, mass, and flight profile as constant as practical.

## Flight Profile

The log contains two armed sorties.

| Metric | Sortie 1 | Sortie 2 |
| --- | ---: | ---: |
| Armed duration | 91.7 s | 106.5 s |
| Approx. airborne duration | 82.5 s | 102.5 s |
| Modes | Loiter | Loiter, PosHold, Loiter |
| Maximum relative altitude | 10.18 m | 4.93 m |
| Maximum groundspeed | 1.02 m/s | 2.05 m/s |
| Maximum climb rate | 0.82 m/s | 2.08 m/s |
| Maximum descent rate | 1.15 m/s | 1.33 m/s |
| Mean current | 10.73 A | 10.87 A |
| Maximum current | 18.28 A | 18.94 A |
| Mean power | 243 W | 243 W |
| Energy consumed | 5.56 Wh | 6.92 Wh |

This flight did not exercise automated takeoff, RTL, waypoint S-curves, survey speed,
or survey turnaround arcs. It cannot validate those recent settings.

## Hover Diagnosis

### Position and Attitude Hold

Hands-off windows required altitude above 1.5 m, less than 3 m/s groundspeed, less
than 0.5 m/s vertical speed, and centered roll, pitch, and yaw RC inputs.

| Metric | Sortie 1 Loiter | Sortie 2 Loiter | July 22 calm reference |
| --- | ---: | ---: | ---: |
| Position error RMS | 0.20 m | 0.17 m | 0.08 m |
| Position error p95 | 0.37 m | 0.32 m | 0.13 m |
| Position error maximum | 0.43 m | 0.44 m | 0.19 m |
| Velocity error RMS | 0.35 m/s | 0.29 m/s | 0.12 m/s |
| Desired tilt median | 2.67 deg | 2.40 deg | 1.63 deg |
| Desired tilt p95 | 6.42 deg | 4.53 deg | 5.63 deg |
| Roll attitude-error RMS | 0.63 deg | 0.65 deg | 0.27 deg |
| Pitch attitude-error RMS | 0.60 deg | 0.48 deg | 0.30 deg |
| Altitude-error RMS | 0.074 m | 0.048 m | 0.061 m |
| Vertical-rate p95 | 0.27 m/s | 0.29 m/s | 0.17 m/s |

The aircraft moved more than the calm reference, but the error remained bounded.
The large visible motion was dominated by low-frequency desired attitude changes,
not by actual attitude diverging from a level command. In wind, a position-holding
multicopter must lean and move slightly while rejecting gusts; a visually motionless
airframe is not a realistic control target.

### Frequency and PID Evidence

The longest centered-stick Loiter windows showed dominant desired-attitude content
around 0.3-1 Hz and actual/error content around 5-6 Hz. The 5-6 Hz component also
appeared in the July 22 clean-hover reference and in earlier tuning flights. It is a
persistent airframe/control component, not a new self-sustaining oscillation.

| Evidence | Result |
| --- | --- |
| Roll D limiting | 0% |
| Pitch D limiting | 0% |
| Hands-off rate-output p95 | approximately 0.03-0.04 |
| Roll reversal events | 3; insufficient for a new gain decision |
| Pitch reversal events | 4; insufficient for a new gain decision |
| Yaw reversal events | 23 |
| IMU clipping | 0 on both IMUs |

Whole-log pitch tracking was under-responsive (`actual/desired ratio 0.52`), not
over-amplified. This aggregate is influenced by low excitation and transition data
and is not suitable for selecting a new gain.

Historical fixed-P/I `0.150` tests at a different gyro filter and under different
excitation found D `0.0045` preferable to `0.0048`. That comparison does not isolate
today's D `0.0051` and cannot prove it caused today's hover behavior. The flown gain
set is retained:

```text
ATC_RAT_PIT_P = 0.150  # unchanged
ATC_RAT_PIT_I = 0.150  # unchanged
ATC_RAT_PIT_D = 0.0051 # unchanged

ATC_RAT_RLL_P = 0.150  # unchanged
ATC_RAT_RLL_I = 0.150  # unchanged
ATC_RAT_RLL_D = 0.00465 # unchanged
```

## Vibration

| Interval | IMU0 vector median / p95 / max | IMU1 vector median / p95 / max |
| --- | ---: | ---: |
| Sortie 1 | 7.87 / 16.37 / 23.75 m/s2 | 5.35 / 10.97 / 16.63 m/s2 |
| Sortie 2 | 7.67 / 10.65 / 13.73 m/s2 | 5.39 / 7.54 / 9.97 m/s2 |
| Hands-off hover | 7.78 / 13.94 / 21.97 m/s2 | 5.43 / 9.47 / 14.80 m/s2 |

IMU0 vibration by vertical state:

| State | Median / p95 / maximum |
| --- | ---: |
| Hover | 7.41 / 12.07 / 21.97 m/s2 |
| Climb | 6.91 / 11.29 / 14.57 m/s2 |
| Descent | 10.89 / 20.03 / 23.75 m/s2 |

Descent was the roughest state, consistent with the aircraft's documented disturbed-
air or settling-with-power sensitivity. All values remained below the 30 m/s2 p95
gate, with no clipping or vibration-compensation event.

## Yaw and Coordinated-Turn Result

The flown yaw configuration was:

```text
PILOT_Y_EXPO = 0.5
PILOT_Y_RATE = 202.5 deg/s
PILOT_Y_RATE_TC = 0.10 s
```

| Metric | Result |
| --- | ---: |
| Actual/desired rate ratio | 0.806 |
| Desired/actual correlation | 0.931 |
| Error RMS | 8.68 deg/s |
| Median reversal overshoot | 17.2% |
| Median rise time | 323 ms |
| Median ring | 1.0 |
| Maximum commanded rate used | 138.2 deg/s |
| Maximum actual rate | 108.1 deg/s |

Yaw was stable and the pilot reported good feel. Do not change yaw PID, rate, expo,
or time constant from this flight.

The RC data does not show systematic transmitter mixing: during yaw-input records,
the p95 roll and pitch channel deviations were both zero. The observed roll-with-yaw
behavior is consistent with the enabled Loiter coordinated-turn term, which adds
`velocity x yaw-rate` lateral acceleration. The review change is:

```text
LOIT_OPTIONS = 0 # changed from 1; coordinated turns disabled
```

## COIN-D6 Validation

Flown configuration:

```text
PRX1_TYPE = 19
PRX1_CSUM = 1
PRX1_INT_THR = 20
PRX1_MIN = 0.8 m
PRX1_MAX = 12 m
SERIAL4_PROTOCOL = 11
SERIAL4_BAUD = 230
AVOID_ENABLE = 1
OA_TYPE = 0
```

Results:

- 2,214 filtered proximity frames were logged.
- Both sorties maintained exactly 10 Hz with a 0.101 s maximum in-flight gap.
- Every airborne frame reported proximity status `Good`.
- UART4 received approximately 13.77 kB/s with zero dropped bytes.
- No closest-distance correlation with throttle (`r = 0.001`).
- No avoidance action was enabled or taken.
- Airborne closest return was 3.78 m median and 1.62 m p05.
- Only two airborne frames were below 1.0 m.

Four sustained sub-1.5 m episodes occurred late in sortie 2 at 1.7-2.0 m AGL. They
lasted 0.7-1.8 seconds, appeared in distinct directions, and had no throttle or tilt
signature. They are more consistent with real nearby objects than power-induced
single-frame phantoms.

`PRX_LOG_RAW` was zero during this flight, so raw-versus-filtered sector behavior
cannot be reconstructed. The next snapshot sets:

```text
PRX_LOG_RAW = 1
```

This changes logging only. Avoidance remains disabled.

## Motors, ESCs, and Thermal Health

| Motor | Position | Command med / p95 / max | RPM med / p95 / max | RPM per command | ESC max temp |
| --- | --- | ---: | ---: | ---: | ---: |
| M1 | front-right | 1409 / 1448 / 1526 | 2950 / 3416 / 3666 | 7.25 | 36 C |
| M2 | back-right | 1416 / 1459 / 1535 | 2944 / 3441 / 3716 | 7.14 | 44 C |
| M3 | back-left | 1476 / 1515 / 1609 | 3483 / 3633 / 4110 | 7.26 | 49 C |
| M4 | front-left | 1466 / 1513 / 1570 | 3396 / 3591 / 3808 | 7.18 | 41 C |

RPM-per-command spread was approximately 1.7%. Raw/filtered RPM correlation was
0.992-0.998, and there were no RPM dropouts. Ordinary ESC telemetry error p95 was
0-0.04%; p99 was 6.3-7.4%, with no ESC above 20% and only M1 briefly above 10%.

Extended DShot status logs contain persistent alert/warning-shaped bits and a fixed
stress value of 120 on two instances. The same pattern exists in prior known-good
hover and envelope logs, always with no EDT2 error bit, maximum stress 0-1, clean RPM,
and low ordinary telemetry error. Treat this as a longstanding Bluejay/EDT2 decoding
artifact unless it changes or gains independent RPM/error evidence.

Thermal maxima in 32 C ambient conditions:

- ESC: 49 C
- IMU: 42.9 C
- barometer: 50.9 C
- MCU: 54 C

No thermal limiting or health failure occurred.

## Power, Navigation, and Compute Health

- Mean airborne current: approximately 10.8 A.
- Maximum current: 18.9 A.
- Mean power: approximately 243 W.
- Loaded minimum voltage: 22.10 V.
- Resting voltage start/end: 23.39 / 22.63 V.
- Learned hover thrust moved to `0.1725`, still leaving substantial thrust headroom.
- GPS: 22-28 satellites, median HDop 0.60, maximum 1.00.
- GPS yaw healthy for 99% of records.
- Optical-flow quality: 104 minimum, 167 median.
- No EKF lane switch, GPS glitch, or failsafe action.
- No DataFlash drops, internal errors, scheduler errors, or CAN errors.
- Scheduler load: 33.2% median, 34.9% maximum.
- Home/origin records occurred only at startup/re-arm; the prior 1 Hz home-rewrite
  fault did not recur.

## Next Validation

Load the updated `certification/parameters/0724.param`, then perform one calm,
low-altitude diagnostic flight:

1. Hover hands-off in Loiter for 30-60 seconds at 3-5 m.
2. Apply yaw-only inputs with roll/pitch centered; verify coordinated lean is gone.
3. Repeat a hands-off hover after yaw testing.
4. Perform one gentle climb and one descent no faster than 1.0 m/s.
5. Do not combine this with high-speed, Auto, RTL, or survey-turn testing.
6. Review `PRXR`, `PRX`, `ATT`, `RATE`, `PIDP`, `VIBE`, `ESC`, `RCIN`, `PSCN`, and
   `PSCE` before making another gain change.

Acceptance targets:

- zero clipping, EKF errors, RPM dropouts, and D limiting;
- IMU0 VIBE p95 below 20 m/s2 in hover and below 30 m/s2 overall;
- hands-off position error below 0.25 m RMS and 0.5 m maximum in light wind;
- no growth in pitch 5-6 Hz error/output versus this flight;
- proximity status `Good`, 10 Hz continuity, and no power-correlated raw returns.

If a PID A/B test is later required, repeat the same hover and separated pitch inputs
twice with one pitch-D value changed between otherwise matched flights. Do not select
a replacement value until the second log shows lower 5-6 Hz error/output without
worse overshoot, ringing, tracking, vibration, or motor telemetry.

## Limitations

- No airspeed sensor was available, so wind cannot be quantified directly.
- The log contains only 3 roll and 4 pitch reversal events, insufficient for a fresh
  rate-gain optimization.
- `PRX_LOG_RAW` was disabled during this flight.
- The July 22 calm-hover comparison used pitch P/I `0.170`, not today's `0.150`.
- The flight was not a dedicated stationary-hover test and included vertical and yaw
  maneuvers.