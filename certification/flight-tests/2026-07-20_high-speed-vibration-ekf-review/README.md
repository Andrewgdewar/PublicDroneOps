# Scout-15 High-Speed Vibration and EKF Incident Review

**Public mechanical event report:**
[`../2026-07-22_motor-vibration-mechanical-event/README.md`](../2026-07-22_motor-vibration-mechanical-event/README.md)

**Date:** 2026-07-20  
**Incident log:** `2026-07-20 19-14-34.bin`  
**Comparison logs:** `2026-07-04 10-34-44.bin` (18 Hz), all July 19 flights, especially `13-50-30.bin` and `14-11-15.bin`  
**Status:** Grounded for mechanical inspection and low-speed validation

## Executive conclusion

The near loss of control was initiated by a real, speed- and load-dependent airframe
vibration. It was not initiated by a pitch-D step. At 23.0 m/s, IMU0 vibration had
reached 144.7 m/s2 with 494 accumulated clips when EKF3 switched from lane 0 to lane
1. Vibration compensation activated 0.77 seconds later. The absolute 153.3 m/s2
maximum occurred during the subsequent PosHold braking maneuver, not before the EKF
switch. Pitch tracking degraded to an 85.3 deg/s peak rate error during that recovery.

The 18 to 25 Hz gyro-filter change did not create the physical acceleration measured
by VIBE. Earlier 18 Hz flights also had severe high-speed vibration. However, the
incident combined less gyro filtering with higher P/I/D gains, and post-filter rate
and D-term energy approximately doubled above 20 m/s. These changes plausibly reduced
control margin once the mechanical vibration became extreme. The data cannot isolate
their individual contributions.

Battery, GPS and processor health were not initiating causes. Before and during the
event, GPS retained 3D status with 28 satellites and 0.6 HDOP. Battery health remained
true; minimum loaded voltage was 21.18 V (3.53 V/cell) during recovery. Scheduler,
I/O and internal error counters remained zero.

## Configuration proof

The incident log contains:

- `INS_GYRO_FILTER=25`
- ESC harmonic notch enabled, mode 3, 25 Hz minimum, harmonics 1 and 2
- roll P/I `0.150`, D `0.00465`
- pitch P/I `0.170`, saved D `0.0051`
- scripted pitch D active through RC options 301 and 302
- `FS_VIBE_ENABLE=1`, `FS_EKF_ACTION=1`
- `ATC_ANGLE_MAX=30`, `LOIT_SPEED_MS=12.5`

The incident occurred in PosHold. PosHold is lean-angle controlled and is not limited
by `LOIT_SPEED_MS`.

## Incident chronology

| Event | Log time | Speed | Vibration / clipping |
| --- | ---: | ---: | --- |
| Final D step to `0.0053` | 343.365 s | low-speed test | 160.9 s before EKF event |
| First IMU0 clipping | 446.811 s | 14.8 m/s | clip count 2 |
| Vibration first exceeds 100 m/s2 | 497.111 s | 20.7 m/s | 7.13 s before EKF event |
| EKF3 lane switch 0 to 1 | 504.246 s | 23.0 m/s | 144.7 m/s2, 494 clips |
| GPS glitch/compass warning | 504.311 s | about 23 m/s | 65 ms after lane switch |
| Vibration compensation ON | 505.011 s | decelerating | 0.77 s after lane switch |
| Pitch stick centered | about 505.25 s | about 20.5 m/s | PosHold braking begins |
| Maximum vibration | 506.311 s | decelerating | 153.3 m/s2 |
| Glitch cleared | 514.711 s | low speed | about 10.5 s after onset |
| Vibration compensation OFF | 529.511 s | low speed | about 25.3 s after onset |

Vibration therefore preceded estimator and control degradation. The EKF event did
not precede and cause the vibration.

No failsafe mode change occurred at the lane switch; the aircraft remained in
PosHold. Pitch input stayed fully forward through the switch, then centered at about
505.25 s. `PHLD_BRK_ANGLE=30` allowed a full 30-degree braking lean and
`PHLD_BRK_RATE=8` ramped toward it. The hard braking amplified the post-trigger peak,
but the pre-braking 144.7 m/s2 and 494 clips had already caused the EKF lane switch.

## PID findings

### Controlled portion

Before the speed run, pitch tracking across the scripted test was healthy:

- normalized pitch rate error: 0.468
- actual/desired rate ratio: 1.028
- correlation: 0.894
- no D limiting (`Dmod` minimum 1.0)

Pitch D `0.0052` is the best-supported provisional value at P/I `0.170`:

| Pitch D | Events | Median overshoot | Median rise | Median ring | Decision |
| ---: | ---: | ---: | ---: | ---: | --- |
| 0.0050 | 24 | 20.9-30.5% | 86-99 ms | 1.5-2.0 | usable, less consistent |
| 0.0052 | 18 | 14.7-22.8% | 98-101 ms | 0-2.5 | best compromise |
| 0.0053 | 14 | 20.9% | 120 ms | 2.0 | slower, no damping benefit |
| 0.0054 | 5 | 27.0% | 92 ms | 4.0 | rejected |

### Degradation with vibration

| Phase | Pitch nerr | Pitch ratio | Pitch correlation | Pitch output p95 |
| --- | ---: | ---: | ---: | ---: |
| Controlled D test | 0.468 | 1.028 | 0.894 | 0.075 |
| Normal flight after D test | 0.607 | 1.011 | 0.818 | 0.086 |
| Speed ramp | 0.869 | 1.035 | 0.636 | 0.116 |
| Severe vibration before EKF | 2.258 | 1.965 | -0.004 | 0.175 |
| EKF event and recovery | 1.414 | 1.362 | 0.319 | 0.182 |

D and output increased gradually during the speed ramp. They increased sharply only
after severe VIBE and EKF degradation. FFT analysis found broad 3-10 Hz maneuver and
controller content, not a new narrow PID oscillation growing before the event.

## Vibration envelope

### July 20 speed run

| Groundspeed | VIBE median | VIBE p95 | Clip increments |
| ---: | ---: | ---: | ---: |
| 6-8 m/s | about 15 | 20-25 | 0 |
| 8.0-8.5 m/s | 17.8 | 21.3 | 0 |
| 8.5-9.0 m/s | 17.9 | 36.0 | 0 |
| 9.5-12.5 m/s | 24-28 | 34-40 | 0 |
| 13-14 m/s | 38 | 49-50 | 0 |
| 14.0-14.5 m/s | 41.8 | 62.7 | 0 |
| 14.5-15.0 m/s | 51.2 | 75.1 | 4 |
| 18.5-19.5 m/s | about 48-50 | 76-79 | 4 |
| 21.5-22.0 m/s | 104.2 | 111.3 | 66 |
| 22.0-22.5 m/s | 114.6 | 130.9 | 143 |
| 22.5-23.0 m/s | 125.4 | 137.9 | 221 |

Both speed and throttle independently predict VIBE. In the pre-EKF speed run:

- correlation with speed: 0.838
- correlation with throttle output: 0.879
- correlation with current: 0.861
- standardized speed/throttle/current regression: 0.504 / 0.606 / -0.009
- model R2: 0.947

At matched throttle `0.14-0.19`, median VIBE rose from 15.8 at 6-8 m/s to 47.1 at
18-20 m/s. At matched speed 14-18 m/s, increasing throttle from 0.12-0.16 to
0.24-0.28 raised median VIBE from 40.6 to 63.7 m/s2. This is a combined aerodynamic
and propulsion-load problem, not merely GPS speed or throttle alone.

### Historical repeatability

- July 19 `13-32-51`: first clip at 15.9 m/s, max VIBE 67.2 m/s2.
- July 19 `13-50-30`: first clip at 17.0 m/s, max VIBE 100.5 m/s2.
- July 19 `14-11-15`: 18.3 m/s, no clipping, max VIBE 69.0 m/s2.
- July 20 incident: first clip at 14.8 m/s, 144.7 m/s2 at the EKF switch, and a
  post-trigger braking maximum of 153.3 m/s2.
- July 4 at 18 Hz: first clip at 13.1 m/s, 5,181 IMU0 clips, max VIBE 118.1 m/s2,
  but no EKF event.

The onset varies with wind, maneuver direction and load. A groundspeed threshold is
therefore only one layer of protection.

## Filtering and spectral findings

The dominant post-filter rate contamination during high-speed flight was around
21-29 Hz. During the July 20 20-24 m/s interval:

- roll/pitch 20-100 Hz rate energy reached 0.493 / 0.451
- roll/pitch 20-100 Hz D-term energy reached 0.00218 / 0.00225

These were approximately twice the comparable 18 Hz values. This comparison is
confounded by the higher July 20 feedback gains and different conditions; it does
not prove 25 Hz caused the event. It does show less remaining noise margin.

ESC speed during the pre-EKF 22-23 m/s interval was approximately 4,008-4,944 RPM
median across the four motors, with no ESC errors and normal temperatures. The
corresponding motor fundamental is roughly 67-82 Hz, above the observed 21-29 Hz
post-filter disturbance. No single ESC showed a telemetry failure.

### Motor localization

The flight log does not identify one failed motor. Motor command-to-RPM behavior was
consistent across all four motors:

- July 20 RPM-per-PWM slopes were 5.66 / 6.04 / 5.67 / 5.94 for M1-M4.
- High-speed command-to-RPM residual p95 was 344 / 363 / 301 / 333 RPM.
- Residual error did not correlate materially with VIBE.
- No motor had an RPM dropout, abrupt desynchronization or event-time ESC error.
- Maximum temperatures were 30 / 36 / 26 / 22 C; M2 was warmest but not abnormal.

M1 front-right and M3 back-left, the CW pair, had the strongest raw RPM correlation
with VIBE and ran at nearly the same higher speed during the incident. Much of this
was commanded by the mixer, so it is not proof of a defect. A secondary post-filter
spectral residual near 169 Hz aligns with the second harmonic of their approximately
82 Hz motor speed. The dominant disturbance remained 22-29 Hz, below every motor
fundamental and consistent with a structural mode being excited by propulsion and
aerodynamic load.

If one position must be inspected first, use this low-confidence order:

1. **M1 front-right**, then **M3 back-left**, with special attention to both CW props.
  M1 had the strongest VIBE correlation and the lowest command-to-RPM slope in both
  high-speed comparison flights, but confidence that M1 alone is responsible is only
  about 25%.
2. **M2 back-right** next because it was consistently warmest and had a weak 66 Hz
  roll residual. Its temperature and command response were still healthy.
3. **M4 front-left** has no direct fault signature despite contributing to the
  multivariate vibration prediction.

The most efficient A/B test is to replace both CW props on M1/M3 with a verified
balanced pair, inspect both motor bells/bearings/mounts, and repeat only the 4/6/8 m/s
sequence. If unchanged, repeat with the CCW M2/M4 pair. A no-prop, equal-command motor
sweep with disarmed/raw logging can identify motor bell or bearing imbalance, but it
cannot reproduce propeller aerodynamic loading or the full-frame flight resonance.

### July 22 no-prop motor test

**Log:** `2026-07-22 09-46-08.bin`  
**Logging:** raw gyro at approximately 2 kHz on two IMUs, `INS_RAW_LOG_OPT=9`  
**Test:** four individual 1.9-second pulses at approximately 970, 1,230, 1,710 and
2,220 RPM per motor

The initial bench test identified the M3 back-left position as a near-synchronous
vibration outlier. At the highest matched test point:

| Motor | Position | RPM | Raw gyro fundamental, IMU0 | Raw gyro fundamental, IMU2 |
| --- | --- | ---: | ---: | ---: |
| M1 | front-right | 2,225 | -45.7 dB | -62.0 dB |
| M2 | back-right | 2,216 | -44.9 dB | -61.8 dB |
| **M3** | **back-left** | **2,214** | **-32.3 dB** | **-49.0 dB** |
| M4 | front-left | 2,233 | -41.0 dB | -57.9 dB |

The M3-position response was 8.7-12.8 dB above the next motor on both raw gyro
instances, equivalent to approximately 2.7-4.4 times the vibration amplitude. It also
appeared at the lower 1,225 and 1,700 RPM test points and became more dominant with
speed. Combined with the abnormal bench sound, this initially supported M3 bearing,
bell, shaft, mounting-face or position-specific structural eccentricity.

M2 did not show M3's fundamental imbalance, but its RPM coefficient of variation was
consistently the highest: 1.85-3.41%, versus approximately 0.82-2.08% for the other
motors. M2 also produced the only small ESC error counts during the pulses. This is
not proof of an active M2 failure, but it supports replacing M2 given its repair
history.

M1 showed a separate fifth-harmonic line at the highest test point. Its fundamental,
RPM stability and ESC behavior did not indicate simple mechanical imbalance. Retain
M1 as an inspection item if vibration remains after replacing M2/M3.

**Initial bench decision:** replace both rear motors, M2 and M3. M3 was the primary
motor suspect; M2 was a precautionary replacement. Repeat the same no-prop test after
installation before fitting props.

### July 22 post-replacement matched test

The final no-prop test repeated PWM 1250 at 2,216-2,258 RPM on all four motors. The
M3-position peak remained 8.5-8.6 dB above M4 on both raw gyro sensors, compared with
8.7-8.9 dB before replacement. The relative outlier was unchanged within 0.4 dB.

The replacement M3 rotated at 37.5 Hz while its dominant response occurred at 39.0
Hz. At the FFT bin nearest exact rotation frequency, M3 and M4 were comparable. The
unresolved response is therefore a nearby structural mode coupled most strongly
through the M3 back-left arm, mount, frame or FC transmission path. Motor replacement
did not remove the narrow bench peak. The mounting and structural path were inspected
with no visible defect identified. A subsequent prop-loaded hover completed with
IMU0 VIBE median/p95/max `6.42/8.49/10.88 m/s2`, zero clips and no EKF or ESC errors.
An evening flight then reached 7.76 m/s with zero clips, no EKF or ESC faults and
IMU0 VIBE p95 below 25 m/s2 in every speed band. This clears the post-repair 4/6/8
m/s gate but not 10 m/s or the former high-speed envelope.

## Mechanical interpretation

Both IMUs measured nearly identical raw acceleration and gyro dispersion during the
failure. This rules against one failed IMU. Reseating the FC can reduce transmitted
vibration, especially if a cable, frame contact or compressed grommet bypasses the
isolation, but the source is external to one sensor.

Most likely contributors, which may coexist:

1. Flexible arm/frame or motor-mount mode excited by forward-flight aerodynamic load.
2. Propeller balance, tracking, hub or motor-bearing excitation amplified by the frame.
3. FC isolation problem or cable bridge amplifying transmission to both IMUs.
4. Battery, payload, landing gear or plate movement changing the structural mode.
5. Reduced gyro-filter and PID margin amplifying the control consequence after onset.

The lightweight frame is a credible contributor because lower torsional stiffness
can move structural modes into the 20-30 Hz region. This requires inspection and
controlled A/B testing; the log alone cannot identify the physical component.

## Temporary operating limits

Until mechanical work and staged validation are complete:

- **Commanded maximum groundspeed: 8 m/s (28.8 km/h).**
- **Hard abort boundary: 10 m/s (36 km/h).** Decelerate and land; this is not a
  normal target.
- Use Loiter for translation. Set its commanded limit to 8 m/s before flight.
- Do not use PosHold for speed testing. `LOIT_SPEED_MS` does not constrain PosHold.
- Do not rely on groundspeed in significant wind; there is no airspeed sensor.
- Abort and land for any accelerometer clip increment.
- Abort for sustained VIBE above 30 m/s2 or any excursion above 60 m/s2.
- In level flight at or below 8 m/s, treat sustained throttle output above 0.20 as a
  load warning and land to inspect the log.

Eight m/s is selected because the July 20 8.0-8.5 m/s bin had zero clipping and a
21.3 m/s2 p95, while the next half-metre-per-second bin already reached a 36.0 m/s2
p95. The 2 m/s gap to the hard boundary provides braking, wind and estimation margin.
This is a temporary diagnostic ceiling, not a demonstrated final aircraft envelope.

`FS_VIBE_ENABLE=1` activated only after the EKF lane switch. It is recovery logic,
not an adequate substitute for these operating limits.

## Required next steps

1. Remove, inspect and balance or replace all propellers; verify tracking and hubs.
2. Rear motors M2 and M3 were replaced, but the M3-position high-RPM response persisted.
3. Inspect and re-torque the M3 motor mount, back-left arm tube/clamps, joints, rear
   center plates, landing gear, wiring contact and payload attachments.
4. Reseat the FC on undamaged, evenly loaded isolation gummies.
5. Ensure the FC touches nothing and all wiring has slack with no isolation bypass.
6. Secure the battery and payload against movement.
7. Use the fully known 18 Hz **and stock rate-gain** baseline for the initial safety
  validation: roll/pitch P/I `0.135`, D `0.0036`. Do not combine 18 Hz with the
  current higher gains without treating it as a new tune.
8. Hover-stage validation passed on `2026-07-22 14-52-42.bin`.
9. The 4/6/8 m/s gate passed on `2026-07-22 19-46-11.bin`.
10. Perform one controlled 10 m/s Loiter validation flight.
11. Land and inspect the log after each envelope expansion.
12. Do not test above 10 m/s until two separate flights pass the 4/6/8/10 sequence.
13. Do not resume top-speed testing until the high-speed vibration source is identified.

The selected pitch tune, including provisional pitch D `0.0052`, remains recorded for
later restoration after the airframe passes the mechanical validation. Temporarily
using stock gains is diagnostic risk reduction, not a rejection of the low-speed PID
result.

## Analysis methods

The review used `tools/logtools/pidreview.py` commands `extract`, `compare`,
`stepresponse`, `segments`, `rank` and `params`, plus direct `pymavlink` extraction
of `VIBE`, `IMU`, `RATE`, `PIDR`, `PIDP`, `ESC`, `GPS`, `CTUN`, `BAT`, `MSG`, `ERR`,
`MODE`, `PTDS` and `XKFM`. Spectral comparisons used windowed FFTs over the longest
continuous sample run in each matched speed bin.
