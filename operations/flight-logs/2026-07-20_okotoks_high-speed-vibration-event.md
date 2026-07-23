# Flight Record - 2026-07-20 - High-Speed Vibration and EKF Event

**Status:** Filed

**Type:** Flight-test safety event / mechanical investigation

| Field | Value |
| --- | --- |
| Date | 2026-07-20 |
| Location | Okotoks, AB area (private test site) |
| Aircraft | Scout 15 quadcopter, registration C-2617791317 |
| Pilot-in-command | Andrew Dewar |
| Firmware | ArduCopter 4.8.0-dev, MatekH743-bdshot, commit `32e43699` |
| DataFlash log | `2026-07-20 19-14-34.bin` |
| Armed duration | 419.6 seconds |
| Outcome | Aircraft recovered and landed; no crash or injury |

## Purpose

The flight began as a controlled pitch-rate damping test and continued into a
high-speed handling check. The purpose of this record is to document the resulting
vibration/EKF event, the immediate operating restrictions, and the evidence used for
the mechanical investigation.

## Configuration

```text
INS_GYRO_FILTER=25
ATC_RAT_RLL_P/I/D=0.150/0.150/0.00465
ATC_RAT_PIT_P/I=0.170/0.170
ATC_RAT_PIT_D=0.0053 during the final flight segment
FS_VIBE_ENABLE=1
FS_EKF_ACTION=1
Flight mode during event: PosHold
```

The pitch-D value had been active for approximately 161 seconds before the EKF event.
No gain change immediately preceded the vibration escalation.

## Event Chronology

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

The absolute vibration maximum occurred during the subsequent PosHold braking
maneuver. It was not the initiating event. Before braking, vibration had already
reached 144.7 m/s2 and the primary IMU had accumulated 494 clips.

No failsafe flight-mode change occurred. The aircraft remained in PosHold. The pilot
reduced speed, regained normal control response, returned to Loiter, and landed.

## Vibration Versus Speed

| Groundspeed | IMU0 VIBE median | VIBE p95 | Clip increments |
| ---: | ---: | ---: | ---: |
| 6-8 m/s | 13.6 m/s2 | 24.6 m/s2 | 0 |
| 8-10 m/s | 16.6 m/s2 | 28.9 m/s2 | 0 |
| 10-12 m/s | 23.4 m/s2 | 37.9 m/s2 | 1 |
| 12-14 m/s | 34.2 m/s2 | 47.9 m/s2 | 3 |
| 14-16 m/s | 42.6 m/s2 | 67.7 m/s2 | 4 |
| 16-18 m/s | 47.8 m/s2 | 71.6 m/s2 | 93 |
| 18-20 m/s | 48.8 m/s2 | 82.8 m/s2 | 138 |
| 20-24 m/s | 109.2 m/s2 | 133.4 m/s2 | 510 |

Vibration rose with both groundspeed and propulsion load. Both IMUs measured the
disturbance, supporting a physical airframe/propulsion source rather than a single
failed sensor.

## Controller and Estimator Findings

- Severe vibration preceded the EKF lane switch.
- Pitch tracking degraded after vibration became extreme.
- Pitch rate error reached approximately 85 deg/s during recovery.
- No D-term slew limiting occurred (`Dmod=1.0`).
- ESC telemetry reported no desynchronization or discrete motor fault.
- GPS remained 3D with 28 satellites and 0.6 HDOP.
- Battery and processor health remained normal.
- The 25 Hz gyro filter did not create the physical accelerometer vibration, but the
  higher filter cutoff and elevated gains provided less control margin than the prior
  18 Hz/stock-gain configuration.

## Immediate Actions

1. Aircraft removed from high-speed service pending investigation.
2. Top-speed and PID expansion flights suspended.
3. Temporary operating target limited to low-speed Loiter validation.
4. Gyro low-pass filter returned to 18 Hz for subsequent flights.
5. No-prop individual motor testing scheduled with raw gyro and ESC RPM logging.
6. Props, motor mounts, arm clamps, center plates, landing gear, battery mount, FC
   isolation and wiring paths placed under inspection.

## Initial Assessment

The event was initiated by a speed- and load-dependent physical vibration. Pitch-D
stepping was not the initiating cause. The vibration source was not identified from
this flight alone; likely contributors included propulsion imbalance and a structural
mode involving the lightweight frame, motor mounts, landing gear/battery assembly, or
FC vibration-transmission path.

## Operating Status After Flight

- Aircraft not cleared for high-speed testing.
- Expensive payload carriage prohibited pending repair and qualification.
- Follow-up bench and flight evidence recorded in
  `2026-07-22_okotoks_motor-repair-qualification.md`.

*Filed public safety-event record. Town-level location only; precise test-site details
are retained privately.*
