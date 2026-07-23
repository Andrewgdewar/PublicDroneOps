# Flight Record - 2026-07-22 - Motor Repair and Envelope Qualification

**Status:** Filed

**Type:** Bench investigation, corrective maintenance, and staged flight qualification

| Field | Value |
| --- | --- |
| Date | 2026-07-22 |
| Location | Okotoks, AB area (private test site) |
| Aircraft | Scout 15 quadcopter, registration C-2617791317 |
| Pilot-in-command | Andrew Dewar |
| Firmware | ArduCopter 4.8.0-dev, MatekH743-bdshot, commit `32e43699` |
| Outcome | Rear motors replaced; hover and 10 m/s gates passed; 12+ m/s not qualified |

## Purpose

This session investigated the July 20 vibration/EKF event, replaced suspect propulsion
components, and progressively requalified the aircraft through bench, hover, low-speed,
and extended-envelope testing.

## Evidence Set

| Test | DataFlash log | Result |
| --- | --- | --- |
| Original no-prop motor comparison | `2026-07-22 09-46-08.bin` | M3-position near-synchronous outlier found |
| Post-repair bench test 1 | `2026-07-22 12-47-57.bin` | Direct motor-frequency response improved through 1,833 RPM |
| Post-repair matched high-RPM test | `2026-07-18 17-59-46.bin` | FC timestamp stale; M3-position structural peak persisted |
| Hover shakedown | `2026-07-22 14-52-42.bin` | Passed |
| 4/6/8 m/s and PID flight | `2026-07-22 19-46-11.bin` | Passed through 7.76 m/s |
| 10-12 m/s, yaw assessment | `2026-07-22 20-01-33.bin` | 10 m/s passed; 12 m/s not qualified |
| Yaw-fix and 17.9 m/s characterization | `2026-07-22 20-28-59.bin` | No clips/EKF fault; high-speed vibration and pitch oscillation confirmed |

## Bench Investigation

All propellers were removed. Individual motors were tested at matched commands with
ESC RPM telemetry and approximately 2 kHz raw gyro logging on two IMUs.

At approximately 2,220 RPM, the original M3 back-left position produced the largest
near-synchronous response:

| Motor | Position | RPM | Raw gyro 0 | Raw gyro 2 |
| --- | --- | ---: | ---: | ---: |
| M1 | front-right | 2,225 | -45.7 dB | -62.0 dB |
| M2 | back-right | 2,216 | -44.9 dB | -61.8 dB |
| M3 | back-left | 2,214 | -32.3 dB | -49.0 dB |
| M4 | front-left | 2,233 | -41.0 dB | -57.9 dB |

M3 was 8.7-12.8 dB above the next motor depending on sensor and comparison. M2 had the
least stable RPM and a prior repair history. Rear motors M2 and M3 were replaced.

At approximately 1,240 RPM after replacement:

- M2 motor-frequency amplitude fell 83-85%.
- M3 motor-frequency amplitude fell 67-68%.
- All ESC errors were zero.

The matched 2,220 RPM retest showed a nearby position-specific peak remained at the
M3 back-left arm/mount path. At the FFT bin nearest exact motor rotation, replacement
M3 and M4 were comparable. The remaining peak was therefore assessed as structural
transmission rather than a second failed replacement motor. The motor mount, arm,
clamps, center plates, landing gear, wiring and FC isolation were inspected; no visible
defect requiring repair was found.

## Corrective Maintenance

- Replaced M2 back-right motor.
- Replaced M3 back-left motor.
- Verified all motor directions with propellers removed.
- Recorded working DShot direction mask `SERVO_BLH_RVMASK=2`.
- Returned gyro low-pass filter to 18 Hz.
- Disabled non-working PRX1 (`PRX1_TYPE=0`).
- Restored normal flight logging after bench tests.
- Recorded optical-flow sensor offset as 10 cm aft and 2 cm below the FC origin.

## Flight Qualification Results

### Hover Shakedown

`2026-07-22 14-52-42.bin`, 84.6 seconds armed, maximum speed 1.41 m/s.

| Metric | IMU0 | IMU1 |
| --- | ---: | ---: |
| VIBE median | 6.42 m/s2 | 4.35 m/s2 |
| VIBE p95 | 8.49 m/s2 | 5.85 m/s2 |
| VIBE maximum | 10.88 m/s2 | 7.93 m/s2 |
| Clips | 0 | 0 |

No EKF, ESC, failsafe, thermal, GPS, rangefinder, optical-flow or processor fault.

### 4/6/8 m/s Qualification

`2026-07-22 19-46-11.bin`, maximum speed 7.76 m/s.

| Speed | IMU0 median | p95 | Maximum | Clips |
| ---: | ---: | ---: | ---: | ---: |
| 0-2 m/s | 10.8 | 18.8 | 24.8 | 0 |
| 2-4 m/s | 13.3 | 20.7 | 25.7 | 0 |
| 4-6 m/s | 15.6 | 21.6 | 26.7 | 0 |
| 6-8 m/s | 14.3 | 19.8 | 30.4 | 0 |

The 30.4 m/s2 maximum was one 0.10-second sample during a pitch maneuver. No EKF or
ESC fault occurred. The 4/6/8 m/s gate passed.

### 10-12 m/s Qualification and Yaw Assessment

`2026-07-22 20-01-33.bin`, maximum speed 11.90 m/s.

| Speed | IMU0 median | p95 | Maximum | Clips |
| ---: | ---: | ---: | ---: | ---: |
| 6-8 m/s | 14-17 | 20 | 22 | 0 |
| 8-10 m/s | 19-21 | 22-23 | 23 | 0 |
| 10-11 m/s | 25.9 | 29.9 | 31.3 | 0 |
| 11-12 m/s | 27.4 | 31.3 | 35.1 | 0 |

Ten m/s passed the temporary p95 vibration gate. The 11-12 m/s p95 exceeded 30 m/s2,
so 12 m/s was not accepted for routine use.

### Extended Characterization

`2026-07-22 20-28-59.bin`, maximum speed 17.86 m/s.

The aircraft remained controllable with zero clips, no EKF lane switch, no vibration
compensation, no failsafe action and no motor fault. Vibration nevertheless increased
rapidly above 12 m/s:

| Speed | IMU0 median | p95 | Maximum | Clips |
| ---: | ---: | ---: | ---: | ---: |
| 10-12 m/s | 22 | 32-39 | 43-47 | 0 |
| 12-14 m/s | 26-36 | 48-53 | 56-61 | 0 |
| 14-15 m/s | 45.9 | 57.5 | 60.5 | 0 |
| 15-18 m/s | 61-66 | 65-72 | 65-77 | 0 |

This is materially better than the July 20 event, which produced clipping and an EKF
lane switch near 23 m/s. It does not qualify the aircraft for routine operation above
10 m/s.

## Controller Findings

### Yaw

Yaw felt abrupt with `PILOT_Y_RATE_TC=0`. The yaw rate loop and GPS-yaw source were
healthy; transmitter centering was also healthy. Setting `PILOT_Y_RATE_TC=0.10`
softened pilot yaw commands without changing yaw PID:

| Metric | Before | After |
| --- | ---: | ---: |
| Yaw ratio | 0.86 | 0.88 |
| Correlation | 0.92 | 0.94 |
| Median event overshoot | 45.0% | 29.5% |
| Median ring | 1.0 | 1.0 |

The 0.10-second input time constant is retained pending subjective confirmation.

### Pitch

The current pitch tune showed a repeatable 5.7-6.6 Hz tracking-error component.
At low speed it was modest. High-speed structural vibration amplified pitch response
and D-term activity substantially. During the 16-18 m/s PosHold burst:

```text
Pitch ratio/correlation: 1.82/0.25
Pitch error RMS/p95: 17.8/43.7 deg/s
VIBE median/p95: 59.4/68.7 m/s2
```

Pitch PID refinement remains open and must be tested below 8 m/s before payload
qualification. No pitch gain change is approved by this record.

## Power, Efficiency and Thermal Health

The final flight consumed 18.88 Wh / 820 mAh over 319.9 seconds armed. Average power
was 206.7 W and maximum current 19.6 A. Minimum loaded voltage was 22.73 V; battery
health remained true.

Preliminary still-air survey figures are wind-sensitive groundspeed measurements:

| Speed | Median power | Energy per distance |
| ---: | ---: | ---: |
| 6 m/s | 208 W | 9.6 Wh/km |
| 8 m/s | 209 W | 7.3 Wh/km |
| 10 m/s | 208 W | 5.8 Wh/km |
| 11 m/s | 202 W | 5.1 Wh/km |

Although higher speed improves energy per ground-distance, vibration controls the
operational envelope. Six to eight m/s remains the recommended survey band.

Maximum ESC temperatures were M1 25 C, M2 34 C, M3 39 C and M4 31 C. IMUs reached
approximately 42 C. All health flags remained true. Battery temperature was not
available.

## Current Qualification Status

| Capability | Status |
| --- | --- |
| Hover | Passed |
| Manual Loiter through 8 m/s | Passed |
| Manual Loiter at 10 m/s | Passed |
| Routine operation above 10 m/s | Not qualified |
| Automated survey grid | Pending |
| Ballast/payload testing | Pending |
| AI computer and camera integration | Pending |
| Expensive payload carriage | Not authorized by this test program yet |
| Yaw input smoothing | Objectively improved; subjective confirmation pending |
| Pitch damping | Further low-speed tuning required |

## Next Actions

1. Retain 6-8 m/s as the routine test/survey band.
2. Do not repeat the 18 m/s run during tuning.
3. Validate yaw feel with `PILOT_Y_RATE_TC=0.10` at low speed.
4. Select and test a pitch PID adjustment below 8 m/s.
5. Remove or disable the obsolete pitch-D stepping script if RC options remain off.
6. Complete two clean low-speed flights before short automated-grid qualification.
7. Use ballast before carrying high-value payloads.

*Filed public test and maintenance record. Town-level location only; precise test-site
details remain private.*
