# Scout-15 Evening Envelope and Yaw Review

**Date:** 2026-07-22

**Log:** `2026-07-22 20-01-33.bin`

**Configuration:** `INS_GYRO_FILTER=18`, roll P/I `0.150`, roll D `0.00465`,
pitch P/I `0.170`, pitch D `0.0051`, yaw P/I `0.180/0.018`, GPS yaw,
T-Motor MS1504 propellers

## Summary

The flight reached 11.90 m/s without accelerometer clipping, EKF errors, vibration
compensation, failsafe actions or motor faults. The post-repair airframe passes the
10 m/s gate. The 11-12 m/s bin reached a VIBE p95 of 31.3 m/s2, slightly above the
30 m/s2 qualification gate, so 12 m/s is not yet accepted as a routine survey speed.

The reported twitchy yaw feel is not caused by yaw PID instability, GPS-yaw noise,
stick-center jitter or vibration. The yaw loop is stable and slightly under-tracks
commands. The current `PILOT_Y_RATE_TC=0` provides no pilot-command smoothing, making
stick changes immediate. The next snapshot sets `PILOT_Y_RATE_TC=0.10`, ArduPilot's
"Crisp" setting, without changing yaw PID, maximum rate, expo, acceleration or RC
 deadzone.

## Flight Profile

| Metric | Result |
| --- | ---: |
| Armed duration | 270.8 s |
| GPS-log duration | 280.4 s |
| Path distance | 857 m |
| Maximum radius | 116.6 m |
| Maximum relative altitude | 21.3 m |
| Maximum groundspeed | 11.90 m/s |
| Mean path groundspeed | 3.06 m/s |
| Energy consumed | 16.63 Wh |
| Capacity consumed | 714 mAh |
| Battery remaining | 99% -> 90% |
| Average power | 213.5 W |
| Maximum current | 17.2 A |

Time by groundspeed band:

| Speed band | Time |
| ---: | ---: |
| 0-2 m/s | 132.1 s |
| 2-4 m/s | 58.1 s |
| 4-6 m/s | 49.6 s |
| 6-8 m/s | 11.6 s |
| 8-10 m/s | 7.4 s |
| 10-12 m/s | 21.6 s |

## Vibration Envelope

No accelerometer clips occurred on either IMU.

| Speed band | IMU0 VIBE median | VIBE p95 | VIBE maximum | Clips |
| ---: | ---: | ---: | ---: | ---: |
| 0-1 m/s | 7.0 m/s2 | 10.2 m/s2 | 19.7 m/s2 | 0 |
| 1-2 m/s | 8.8 m/s2 | 12.8 m/s2 | 26.1 m/s2 | 0 |
| 2-3 m/s | 9.0 m/s2 | 15.5 m/s2 | 29.4 m/s2 | 0 |
| 3-4 m/s | 10.6 m/s2 | 17.3 m/s2 | 25.9 m/s2 | 0 |
| 4-5 m/s | 11.8 m/s2 | 17.3 m/s2 | 34.9 m/s2 | 0 |
| 5-6 m/s | 14.7 m/s2 | 18.6 m/s2 | 31.4 m/s2 | 0 |
| 6-7 m/s | 14.2 m/s2 | 19.6 m/s2 | 22.4 m/s2 | 0 |
| 7-8 m/s | 16.9 m/s2 | 20.2 m/s2 | 21.2 m/s2 | 0 |
| 8-9 m/s | 19.9 m/s2 | 21.2 m/s2 | 21.4 m/s2 | 0 |
| 9-10 m/s | 20.6 m/s2 | 23.2 m/s2 | 23.4 m/s2 | 0 |
| 10-11 m/s | 25.9 m/s2 | 29.9 m/s2 | 31.3 m/s2 | 0 |
| 11-12 m/s | 27.4 m/s2 | 31.3 m/s2 | 35.1 m/s2 | 0 |

Whole-flight VIBE:

| Metric | IMU0 | IMU1 |
| --- | ---: | ---: |
| Median | 9.7 m/s2 | 6.7 m/s2 |
| p95 | 25.6 m/s2 | 17.6 m/s2 |
| Maximum | 35.1 m/s2 | 24.2 m/s2 |
| Clips | 0 | 0 |

The envelope remains controlled through 10 m/s. Vibration rises predictably above
8 m/s and the 11-12 m/s p95 exceeds the temporary 30 m/s2 gate. This is not a new
failure signature, but it supports retaining 6-8 m/s as the normal survey band and
10 m/s as the current qualified upper limit.

## Yaw Diagnosis

### Flown yaw configuration

```text
ATC_ANG_YAW_P=4.5
ATC_RAT_YAW_P=0.18
ATC_RAT_YAW_I=0.018
ATC_RAT_YAW_D=0
ATC_ACC_Y_MAX=270
PILOT_Y_RATE=202.5
PILOT_Y_EXPO=0.4
PILOT_Y_RATE_TC=0
RC4_DZ=20
EK3_SRC1_YAW=2
```

### Rate-loop behavior

The yaw rate loop is stable and slightly under-responsive:

| Metric | Result |
| --- | ---: |
| Whole-flight desired/actual ratio | 0.86 |
| Whole-flight correlation | 0.92 |
| Meaningful-command ratio | 0.85 |
| Meaningful-command correlation | 0.94 |
| Event overshoot median | 45.0% |
| Event ring median | 1.0 |
| WebTools peak/tail | 0.75 / 0.74 |
| WebTools synthetic overshoot/ring | 0 / 0 |
| Yaw output p95 / maximum | 0.102 / 0.172 |

During 54.1 seconds of meaningful yaw input, desired yaw rate p50/p75/p95/maximum was
21.9/39.9/87.5/190.5 deg/s. Actual rate was 19.1/35.6/78.1/151.4 deg/s. Reducing yaw
rate P would worsen this under-tracking and is not recommended.

### Stick-center behavior

The transmitter centered cleanly. Of 2,651 airborne RC samples, 2,023 were exactly
1501. During the longest 32.2-second centered-stick segment:

- RC4 remained between 1490 and 1501, entirely inside the configured deadzone.
- Heading error RMS/p95/maximum was 0.50/0.93/1.48 degrees.
- Actual yaw-rate RMS/p95 was 1.44/1.90 deg/s.
- Yaw output p95 was 0.042.

This rules against transmitter-center noise or an undersized deadzone as the primary
cause. `RC4_DZ=20` remains unchanged.

### GPS-yaw behavior

GPS yaw was healthy:

- 26-28 satellites, HDOP at or below 0.5.
- GPS-yaw versus AHRS-yaw difference during centered flight had 0.17-degree median,
  0.60-degree standard deviation, 1.33-degree p95 and 2.35-degree maximum.
- GPS-derived yaw-rate correlated 0.968 with gyro yaw-rate.

No independent GPS-yaw jump or estimator fault was found.

### Yaw adjustment

The first adjustment is input shaping only:

```text
PILOT_Y_RATE_TC=0.10
```

ArduPilot describes 0.10 seconds as "Crisp"; the flown value of zero is sharper than
its "Very Crisp" example. This time constant applies to pilot yaw commands in Loiter
and PosHold. It does not soften autonomous yaw behavior or centered-stick heading
hold.

Leave these unchanged for the first comparison:

```text
ATC_ANG_YAW_P=4.5
ATC_RAT_YAW_P=0.18
ATC_RAT_YAW_I=0.018
ATC_RAT_YAW_D=0
ATC_ACC_Y_MAX=270
PILOT_Y_RATE=202.5
PILOT_Y_EXPO=0.4
RC4_DZ=20
```

If 0.10 seconds remains too sharp, test 0.15 seconds next. If the feel becomes smooth
but full-stick turns remain too fast, reduce `PILOT_Y_RATE` separately in a later test.
Do not change both parameters at once.

## Roll and Pitch Controller Health

Controller tracking improved versus the preceding flight:

| Axis | Ratio | Correlation | RMS error | D limiting |
| --- | ---: | ---: | ---: | ---: |
| Roll | 1.14 | 0.91 | 3.10 deg/s | 0% |
| Pitch | 1.08 | 0.79 | 4.46 deg/s | 0% |
| Yaw | 0.86 | 0.92 | 7.45 deg/s | 0% |

No gain change is indicated by this flight.

## Motor and Thermal Health

| Motor | Position | Command med/p95/max | RPM med/p95/max | RPM per command | Temp med/max |
| --- | --- | ---: | ---: | ---: | ---: |
| M1 | front-right | 1369/1428/1523 | 2738/3236/3819 | 7.48 | 25/28 C |
| M2 | back-right | 1415/1448/1525 | 3028/3469/3695 | 7.37 | 34/36 C |
| M3 | back-left | 1398/1452/1517 | 2903/3477/3785 | 7.41 | 39/41 C |
| M4 | front-left | 1458/1486/1538 | 3436/3579/3778 | 7.46 | 31/33 C |

RPM-per-command values remain tightly grouped. Raw/filtered RPM correlation was at
least 0.991. Bidirectional-DShot telemetry error-rate p95 was zero on every ESC and
p99 remained at or below 0.11. There were no DataFlash `ERR` records or RPM dropouts.
M3 remained the warmest ESC at 41 C, which is acceptable.

IMU0 and IMU1 temperatures were 29.3-41.4 C and 30.3-41.4 C. Gyro and accelerometer
health flags remained true. Battery temperature was not available in telemetry.

## Battery and Efficiency

Battery health remained true throughout:

| Metric | Result |
| --- | ---: |
| Loaded voltage start/end/min | 23.85 / 23.50 / 23.10 V |
| Resting voltage start/end/min | 23.86 / 23.51 / 23.37 V |
| Maximum current | 17.2 A |
| Average current | 9.17 A |
| Average power | 213.5 W |
| Capacity consumed | 714 mAh |
| Energy consumed | 16.63 Wh |

Preliminary level-flight efficiency:

| Groundspeed | Median power | Energy per distance |
| ---: | ---: | ---: |
| 4 m/s | 216 W | 15.0 Wh/km |
| 5 m/s | 208 W | 11.6 Wh/km |
| 6 m/s | 200 W | 9.3 Wh/km |
| 8 m/s | 198 W | 6.9 Wh/km |
| 9 m/s | 195 W | 6.0 Wh/km |
| 10 m/s | 203 W | 5.6 Wh/km |
| 11 m/s | 214 W | 5.4 Wh/km |

Nine m/s had the lowest measured power and 11 m/s the best energy per ground-distance.
These are preliminary groundspeed results from a short flight and are affected by wind
direction. They do not override the vibration qualification limit. Six to eight m/s
remains the recommended survey band until repeated 10 m/s missions pass.

Using a 160 Wh usable-energy assumption, the measured average power implies roughly
45 minutes airborne. The 11 m/s energy figure extrapolates to about 29 km still-air
ground range, but this should not be treated as qualified range until repeated
bidirectional survey legs confirm it.

## Supporting Systems

- No EKF lane switch, vibration compensation, failsafe action or in-flight `ERR` record.
- Processor load approximately 29%; scheduler, internal and I/O errors zero.
- GPS remained 3D with 26-28 satellites and HDOP no worse than 0.5.
- Optical-flow quality was 74-225, median 185.
- Rangefinder operated through its useful altitude range; readings above configured
  range were rejected by status and did not affect high-altitude cruise.
- The post-disarm `PreArm: Gyros inconsistent` message occurred after motor shutdown,
  not in flight. Recheck normal pre-arm status after the next cold start.
- `pitch-d-step.lua` remained installed and printed a waiting message every five
  seconds because RC options 301/302 were disabled. It did not change gains.

## Next Test

Load `certification/parameters/0722night.param`, reboot, and verify:

```text
PILOT_Y_RATE_TC=0.10
FLOW_POS_Z=0.02
```

Perform a short Loiter and PosHold yaw-feel comparison at low speed. Use repeated
small, medium and one larger yaw command in each mode, with centered-stick holds
between commands. Do not change yaw PID. If yaw feel is improved and centered heading
hold remains clean, retain 0.10 seconds.
