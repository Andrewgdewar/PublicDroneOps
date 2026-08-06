# Scout-15 Mechanical Vibration Event and Bench Investigation

**Aircraft:** Scout-15 quadcopter  
**Flight event:** 2026-07-20  
**Bench investigation:** 2026-07-22  
**Flight log:** `2026-07-20 19-14-34.bin`  
**Bench log:** `2026-07-22 09-46-08.bin`  
**Event classification:** Mechanical vibration anomaly with EKF degradation  
**Aircraft outcome:** Recovered and landed; no crash  
**Corrective action:** Rear motors M2 and M3 replaced  
**Verification status:** Post-repair hover and 10 m/s forward-speed validation passed;
automated mission and payload verification pending

**Detailed engineering analysis:**
[`../2026-07-20_high-speed-vibration-ekf-review/README.md`](../2026-07-20_high-speed-vibration-ekf-review/README.md)

**Evening envelope, yaw, efficiency and thermal review:**
[`../2026-07-22_evening-envelope-yaw-review/README.md`](../2026-07-22_evening-envelope-yaw-review/README.md)

**Night yaw-fix, pitch-oscillation and extended-envelope review:**
[`../2026-07-22_night-yaw-pitch-envelope-review/README.md`](../2026-07-22_night-yaw-pitch-envelope-review/README.md)

## Summary

During a high-speed PosHold flight, Scout-15 developed severe speed- and
load-dependent vibration. At 23.0 m/s groundspeed, the primary IMU had accumulated
494 accelerometer clips and reported a vibration vector magnitude of 144.7 m/s2.
EKF3 switched estimator lanes and subsequently enabled vibration compensation. The
pilot centered the pitch input and the aircraft decelerated. PosHold braking amplified
the vibration to a later maximum of 153.3 m/s2, but severe vibration and clipping had
already occurred before the EKF lane switch.

The aircraft remained in PosHold; no failsafe flight-mode change occurred. It was
recovered and landed without a crash.

A no-prop bench test was performed two days later using matched individual motor
commands, ESC RPM telemetry and approximately 2 kHz raw gyro logging. The M3
back-left position produced a repeatable near-synchronous vibration line substantially
above the other three positions at matched RPM. M2, back-right, showed the least
stable RPM and had a previous repair history. Both rear motors were replaced as the
initial corrective action.

The final matched-RPM post-repair test showed that the dominant M3-position response
remained 8.5-8.6 dB above M4, essentially unchanged from the original 8.7-8.9 dB
difference. The replacement motor's exact rotation-frequency response was comparable
to M4, while a nearby fixed 39 Hz peak remained dominant. This shifts the primary
diagnosis from the removed M3 motor to the M3 arm, mount, frame or vibration
transmission path. The motor replacement did not close the mechanical event.

## Aircraft Configuration

| Item | Configuration during event |
| --- | --- |
| Frame | 580 mm Quad-X, carbon tube arms |
| Flight controller | Matek H743-SLIM |
| Motors | Angel X4108 380 KV |
| Propellers | T-Motor MS1504 15x5.6 polymer/carbon composite |
| Battery | 6S2P Li-ion |
| Firmware | ArduCopter 4.8.0-dev, commit `32e43699` |
| Gyro low-pass filter | `INS_GYRO_FILTER=25` |
| Harmonic notch | ESC telemetry tracking, harmonics 1 and 2 |
| Roll rate gains | P/I `0.150`, D `0.00465` |
| Pitch rate gains | P/I `0.170`, runtime D `0.0053` |
| Flight mode | PosHold |

The motor positions follow ArduPilot Quad-X numbering:

```text
        FRONT
    M4 (CCW)  M1 (CW)
         \      /
          \    /
          /    \
         /      \
    M3 (CW)   M2 (CCW)
        BACK
```

## In-Flight Observation

### Event chronology

| Event | Log time | Groundspeed | Evidence |
| --- | ---: | ---: | --- |
| First IMU0 clipping | 446.811 s | 14.8 m/s | Clip count increased from zero |
| Vibration first exceeded 100 m/s2 | 497.111 s | 20.7 m/s | 7.13 s before EKF switch |
| EKF3 lane switch | 504.246 s | 23.0 m/s | 144.7 m/s2, 494 clips |
| GPS glitch/compass warning | 504.311 s | about 23 m/s | 65 ms after lane switch |
| Vibration compensation enabled | 505.011 s | decelerating | 0.77 s after lane switch |
| Pitch input centered | about 505.25 s | about 20.5 m/s | PosHold braking began |
| Maximum vibration | 506.311 s | decelerating | 153.3 m/s2 |
| Estimator glitch cleared | 514.711 s | low speed | Recovery in progress |
| Vibration compensation disabled | 529.511 s | low speed | Estimator recovered |

The absolute maximum occurred during braking. It was not the initiating event. At the
EKF lane switch, before braking, vibration was already 144.7 m/s2 and the primary IMU
had accumulated 494 clips.

### Speed-dependent vibration

| Groundspeed band | Median VIBE | VIBE p95 | Clip increments |
| ---: | ---: | ---: | ---: |
| 6-8 m/s | about 15 m/s2 | 20-25 m/s2 | 0 |
| 8.0-8.5 m/s | 17.8 m/s2 | 21.3 m/s2 | 0 |
| 9.5-12.5 m/s | 24-28 m/s2 | 34-40 m/s2 | 0 |
| 14.0-14.5 m/s | 41.8 m/s2 | 62.7 m/s2 | 0 |
| 14.5-15.0 m/s | 51.2 m/s2 | 75.1 m/s2 | 4 |
| 21.5-22.0 m/s | 104.2 m/s2 | 111.3 m/s2 | 66 |
| 22.0-22.5 m/s | 114.6 m/s2 | 130.9 m/s2 | 143 |
| 22.5-23.0 m/s | 125.4 m/s2 | 137.9 m/s2 | 221 |

Vibration correlated with both groundspeed and motor load. This indicated a
propulsion-driven excitation amplified by an airframe structural mode, rather than a
single estimator or software fault.

## Bench Test Method

All propellers were removed. Disarmed logging and raw gyro logging were enabled:

```text
LOG_DISARMED = 3
INS_RAW_LOG_OPT = 9
```

Mission Planner Motor Test issued four individual 1.9-second pulses per command
level. Each motor was tested at approximately:

- 970 RPM
- 1,230 RPM
- 1,710 RPM
- 2,220 RPM

The log contained:

- Individual motor output commands (`RCOU`)
- ESC RPM telemetry (`ESC`)
- Raw gyro data at approximately 2 kHz (`GYR`)
- Both IMU streams (`IMU` and `VIBE`)

Motor output channels were mapped using the logged servo functions and confirmed by
the ESC instance that reported non-zero RPM during each pulse.

## Bench Results

### Once-per-revolution vibration

At the highest matched test point, all motors were within 19 RPM of one another:

| Motor | Position | RPM | Fundamental, raw gyro 0 | Fundamental, raw gyro 2 |
| --- | --- | ---: | ---: | ---: |
| M1 | front-right | 2,225 | -45.7 dB | -62.0 dB |
| M2 | back-right | 2,216 | -44.9 dB | -61.8 dB |
| **M3** | **back-left** | **2,214** | **-32.3 dB** | **-49.0 dB** |
| M4 | front-left | 2,233 | -41.0 dB | -57.9 dB |

The M3-position near-synchronous line was 8.7-12.8 dB above the next motor on both raw
gyro instances. This is approximately 2.7-4.4 times the vibration amplitude at
matched RPM. The response also appeared at the lower 1,225 and 1,700 RPM points and
became more dominant as RPM increased.

The frequency tracked M3 rotational speed:

| M3 test RPM | Rotation frequency | Dominant fundamental observation |
| ---: | ---: | --- |
| 1,225 RPM | 20.4 Hz | Strongest motor fundamental in test group |
| 1,700 RPM | 28.3 Hz | Strongest motor fundamental in test group |
| 2,214 RPM | 36.9 Hz | 8.7-12.8 dB above next motor |

This behavior initially supported mechanical eccentricity at the motor or its mounting
position, including a bearing, bell, shaft, mounting-face, arm or frame defect. An
abnormal sound had also been observed from the removed M3 during bench operation.
The post-replacement test later separated these hypotheses and showed that the dominant
high-RPM response followed the M3 position rather than the removed motor.

### M2 RPM stability

M2 did not show M3's strong fundamental imbalance, but its no-load RPM was the least
stable at every matched command:

| Command point | M2 RPM coefficient of variation | Other motors, range |
| --- | ---: | ---: |
| about 970 RPM | 3.41% | 0.87-1.63% |
| about 1,230 RPM | 2.98% | 0.82-2.08% |
| about 1,710 RPM | 2.37% | 0.86-1.20% |
| about 2,220 RPM | 1.85% | 0.97-1.28% |

M2 also produced the only small ESC error counts during the test pulses. These values
were not sufficient to establish an active M2 failure in isolation. Because M2 had
previously sustained damage and been repaired, it was replaced as a precaution.

### Other motors

M1 showed a fifth-harmonic line at the highest command but did not show M3's strong
once-per-revolution imbalance, RPM instability or ESC errors. M4 did not show a
specific fault signature. Both remain subject to comparison during post-repair
verification.

## Findings

1. The flight event was initiated by physical vibration that increased with speed and
   propulsion load.
2. Severe vibration and accelerometer clipping preceded the EKF lane switch.
3. PosHold braking caused the absolute vibration maximum but did not initiate the
   event.
4. The initial no-prop test identified the M3 back-left position as a near-synchronous
   vibration outlier on both raw gyro sensors.
5. Replacing M3 did not remove the dominant high-RPM relative outlier. The unresolved
   response is therefore associated with the M3 arm, mount, frame or transmission path,
   not the removed motor alone.
6. M2 showed abnormal relative RPM variability and had a previous repair history.
7. The lightweight frame likely amplified propulsion excitation through a structural
   mode; replacing a motor removes the source candidate but does not by itself prove
   the complete airframe issue resolved.
8. The gyro-filter and PID configuration affected control margin but did not create
   the mechanical acceleration measured by the IMUs.

## Corrective Action

The following repair was made:

- Replaced M2, back-right.
- Replaced M3, back-left.
- Preserved the removed M3 for bearing, bell and shaft inspection.

The repair removed the initially suspected motor and the previously repaired rear
motor. Final matched-RPM verification showed that this action did not remove the
dominant M3-position response, so additional structural corrective action is required.

## Required Verification

The corrective action is not considered closed until all verification steps pass.

### Post-repair bench test 1

**Log:** `2026-07-22 12-47-57.bin`  
**Configuration:** `INS_GYRO_FILTER=18`, raw gyro logging enabled,
`SERVO_BLH_RVMASK=2`  
**Test range:** approximately 808, 1,240, 1,540 and 1,830 RPM per motor

The replacement motors produced zero ESC errors at every test point. At the directly
matched approximately 1,240 RPM point, the rear-motor once-per-revolution amplitudes
changed as follows:

| Motor | Raw gyro 0, before -> after | Amplitude change | Raw gyro 2, before -> after | Amplitude change |
| --- | ---: | ---: | ---: | ---: |
| M2 | -29.6 -> -46.0 dB | 85% lower | -38.2 -> -53.6 dB | 83% lower |
| M3 | -25.3 -> -34.9 dB | 67% lower | -33.6 -> -43.5 dB | 68% lower |

M3 RPM stability improved from 2.05% to 1.49% coefficient of variation at the
matched point. M2 RPM variability remained approximately unchanged, 2.99% before
versus 3.09% after, indicating that its earlier variability was not isolated to the
removed motor. No post-repair pulse produced an ESC error.

At the highest post-repair point, approximately 1,830 RPM, all four motors excited a
common 32.3 Hz frame response while their true rotation frequencies were 30.1-30.8
Hz. At the exact rotation-frequency FFT bin, M3 measured -42.9 dB and M4 measured
-42.6 dB; M3 was therefore no longer a unique motor-frequency outlier. The stronger
32.3 Hz response is interpreted as a frame transfer mode, not direct evidence of
replacement-motor imbalance.

This test is a partial bench pass. It did not reproduce the original approximately
2,214-2,233 RPM point where removed M3 was 8.7-12.8 dB above the other motors. One
final no-prop pulse set at the original 25% command, approximately 2,200 RPM, is
required before closing bench verification.

### Post-repair bench test 2 - matched high-RPM point

**Downloaded:** 2026-07-22 12:59 local time  
**DataFlash filename:** `2026-07-18 17-59-46.bin` (FC timestamp was stale)  
**Test range:** 2,216-2,258 RPM, PWM 1250, all four motors

The final test reproduced the original command and RPM point. All four motors
completed their pulses without ESC errors.

| Motor | Raw gyro 0, before -> after | Raw gyro 2, before -> after | Result |
| --- | ---: | ---: | --- |
| M1 | -45.7 -> -47.0 dB | -62.0 -> -63.9 dB | Comparable/slightly lower |
| M2 | -44.9 -> -50.0 dB | -61.8 -> -67.1 dB | Lower after replacement |
| **M3 position** | **-32.3 -> -30.6 dB** | **-49.0 -> -47.5 dB** | **Dominant peak persisted** |
| M4 | -41.0 -> -39.2 dB | -57.9 -> -56.0 dB | Comparable test-to-test shift |

Before replacement, M3 was 8.7 dB above M4 on raw gyro 0 and 8.9 dB above M4 on
raw gyro 2. After replacement, the differences were 8.6 and 8.5 dB. The relative
outlier was therefore unchanged within 0.4 dB.

The replacement M3 rotated at 2,250 RPM, or 37.5 Hz. Its dominant peak occurred at
39.0 Hz. At the FFT bin nearest exact motor rotation, M3 measured -49.7/-66.5 dB and
M4 measured -49.1/-65.4 dB on the two gyros, making their direct motor-frequency
responses comparable. The unresolved peak is a nearby structural mode most strongly
coupled through the M3 back-left position.

**Bench decision:** motor replacement is complete. The narrow position-specific bench
peak persisted, but broadband vibration, ESC telemetry and exact-frequency motor
response remained acceptable. The M3 mounting and structural path were inspected with
no visible defect identified. Proceed with conservative prop-loaded validation while
continuing to monitor the position-specific response.

### Post-repair hover flight

**Log:** `2026-07-22 14-52-42.bin`

**Duration:** 84.6 seconds armed, approximately 64 seconds airborne

**Maximum groundspeed:** 1.41 m/s

**Modes:** Loiter, brief PosHold, Loiter

The aircraft used the authoritative July 22 configuration: gyro filter 18 Hz, roll
P/I `0.150` and D `0.00465`, pitch P/I `0.170` and D `0.0051`, PRX1 disabled and
`SERVO_BLH_RVMASK=2`.

| Metric | IMU0 | IMU1 |
| --- | ---: | ---: |
| Airborne VIBE median | 6.42 m/s2 | 4.35 m/s2 |
| Airborne VIBE p95 | 8.49 m/s2 | 5.85 m/s2 |
| Airborne VIBE maximum | 10.88 m/s2 | 7.93 m/s2 |
| Accelerometer clip increments | 0 | 0 |

No `ERR` records, EKF lane switches, vibration-compensation events or failsafe mode
changes occurred. Battery, GPS, rangefinder, optical flow and scheduler health were
normal. Airborne roll and pitch attitude error RMS was 0.25 and 0.27 degrees.

All four ESCs reported zero airborne errors and normal temperatures. Correct physical
mapping and median airborne values were:

| Motor | Position | Output | Median command | Median RPM | RPM per command |
| --- | --- | ---: | ---: | ---: | ---: |
| M1 | front-right | 4 | 1,403 | 2,982 | 7.44 |
| M2 | back-right | 1 | 1,404 | 2,971 | 7.37 |
| M3 | back-left | 2 | 1,426 | 3,166 | 7.47 |
| M4 | front-left | 3 | 1,440 | 3,358 | 7.56 |

The RPM-per-command spread was small and followed commanded loading. No motor was a
flight-telemetry outlier.

Compared using the same airborne filter, IMU0 VIBE median/p95 was `6.42/8.49`, versus
`6.67/13.61` on the July 13 18 Hz maiden and `6.04/8.33` on the July 19 25 Hz filter
hover. The post-repair hover is therefore within the best same-prop baseline range.

This is a hover-stage pass only. The flight did not exercise the 4/6/8 m/s sequence
and cannot clear the speed-dependent resonance or validate pitch step response.

### Post-repair forward-speed and PID flight

**Log:** `2026-07-22 19-46-11.bin`

**Duration:** 118.1 seconds armed

**Maximum groundspeed:** 7.76 m/s

**Modes:** Loiter, PosHold PID maneuvers, Loiter

No `ERR` records, EKF lane switches, vibration-compensation events, failsafe mode
changes or accelerometer clips occurred. Battery, GPS, rangefinder, optical flow and
scheduler health remained normal.

| Speed band | IMU0 VIBE median | VIBE p95 | VIBE maximum | Clips |
| ---: | ---: | ---: | ---: | ---: |
| 0-2 m/s | 10.8 m/s2 | 18.8 m/s2 | 24.8 m/s2 | 0 |
| 2-4 m/s | 13.3 m/s2 | 20.7 m/s2 | 25.7 m/s2 | 0 |
| 4-6 m/s | 15.6 m/s2 | 21.6 m/s2 | 26.7 m/s2 | 0 |
| 6-8 m/s | 14.3 m/s2 | 19.8 m/s2 | 30.4 m/s2 | 0 |

The 30.4 m/s2 maximum was one 0.10-second sample during a pitch maneuver at 6.3 m/s.
Total time above 25 m/s2 was 0.50 seconds, with a maximum continuous duration of
0.40 seconds. No excursion exceeded 60 m/s2.

Compared with the pre-repair incident flight at 6-8 m/s, IMU0 VIBE p95 improved from
24.6 to 19.8 m/s2 and maximum improved from 48.5 to 30.4 m/s2. Median vibration was
similar, 13.6 before versus 14.3 m/s2 after. The repair and 18 Hz configuration reduced
the high-end vibration excursions without creating a new broadband vibration issue.

All four motor RPM-per-command values were tightly grouped from 7.52 to 7.59. Raw and
filtered RPM correlation was at least 0.993, temperatures were normal, and the
bidirectional-DShot telemetry error-rate p95 and p99 were zero for all four ESCs.

The PosHold block provided 9 roll and 17 pitch command events:

| Axis | Rate ratio | Correlation | Error RMS / p95 / max | Event overshoot | Ring | Assessment |
| --- | ---: | ---: | ---: | ---: | ---: | --- |
| Roll | 1.11 | 0.86 | 3.80 / 8.90 / 23.51 deg/s | 26.7% median | 1.0 | Accepted |
| Pitch | 1.23 | 0.79 | 7.67 / 18.27 / 27.06 deg/s | 34.6% median | 1.0 | Stable, slightly aggressive |

Neither axis activated D limiting. The WebTools-compatible transfer response showed
no synthetic overshoot or ringing, with roll peak/tail `0.87/0.69` and pitch
`0.86/0.84`. Roll is accepted. Pitch is safe for continued envelope qualification
but remains the axis to refine later; no gain change is made from this flight.

This flight passes the post-repair 4/6/8 m/s qualification gate. It does not qualify
10 m/s, automated missions, payload operation or the former high-speed envelope.

### Bench verification

1. Keep the completed bench logs as the mechanical baseline.
2. Continue pre-flight inspection of the M3 mount, back-left arm and FC isolation path.
3. Repeat no-prop testing only if sound, RPM stability or flight vibration regresses.
4. Normal flight logging is restored:

```text
LOG_DISARMED = 0
INS_RAW_LOG_OPT = 0
```

### Flight verification

Initial flight verification will use the previously demonstrated conservative control
baseline:

```text
INS_GYRO_FILTER = 18
ATC_RAT_RLL_P = 0.135
ATC_RAT_RLL_I = 0.135
ATC_RAT_RLL_D = 0.0036
ATC_RAT_PIT_P = 0.135
ATC_RAT_PIT_I = 0.135
ATC_RAT_PIT_D = 0.0036
LOIT_SPEED_MS = 8
```

The validation sequence is:

1. Hover.
2. Separated steady passes at 4, 6 and 8 m/s in Loiter.
3. Land and review the DataFlash log.
4. Require zero new accelerometer clips.
5. Require VIBE p95 below 30 m/s2 at each speed.
6. Abort for sustained VIBE above 30 m/s2 or any excursion above 60 m/s2.
7. Test 10 m/s only after the 4/6/8 m/s sequence passes.
8. Require two separate clean flights before considering speeds above 10 m/s.

## Closure Criteria

This mechanical event may be closed when:

- The completed no-prop and hover evidence remains free of broadband motor or ESC
   degradation.
- Two post-repair flights complete the staged validation without clipping.
- VIBE p95 remains below 30 m/s2 through 10 m/s.
- No EKF lane switch, vibration compensation event or control degradation occurs.
- Removed M3 inspection findings are recorded, if disassembly is performed.

Until these criteria are met, Scout-15 remains restricted to the temporary 8 m/s
commanded limit and is not cleared for high-speed testing.

## Evidence and Limitations

The conclusions are based on DataFlash telemetry and comparative signal analysis.
DataFlash log files are identified above but are not embedded in this document.

The no-prop test can identify motor mechanical or commutation vibration. It cannot
reproduce propeller imbalance, aerodynamic loading, thrust-loaded bearings or the
complete in-flight frame resonance. Successful bench verification must therefore be
followed by staged flight verification.
