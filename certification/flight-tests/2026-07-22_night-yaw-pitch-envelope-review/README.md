# Scout-15 Night Yaw, Pitch and Envelope Review

**Date:** 2026-07-22

**Log:** `2026-07-22 20-28-59.bin`

**Purpose:** Validate yaw input softening, extend the repaired-airframe speed envelope,
and characterize reported pitch aggressiveness/oscillation before changing pitch PID.

## Executive Summary

The `PILOT_Y_RATE_TC=0.10` yaw adjustment worked. Median detected yaw overshoot fell
from 45.0% to 29.5%, while whole-flight yaw ratio/correlation improved from
`0.86/0.92` to `0.88/0.94`. No yaw PID change is indicated.

The aircraft reached 17.86 m/s with zero accelerometer clips, no in-flight `ERR`
records, no EKF lane switch, no vibration compensation and no failsafe action. This
is a major improvement over the July 20 incident. However, vibration again rose
sharply above 12 m/s. The 15-18 m/s bins produced VIBE p95 of 65-72 m/s2 and a maximum
of 76.6 m/s2. The former catastrophic peaks and clipping did not recur, but speeds
above 12 m/s remain outside the qualified survey envelope.

The reported pitch oscillation is confirmed. Pitch tracking error has a repeatable
5.7-6.6 Hz component. It is present at low speed but becomes severe when high-speed
structural vibration increases. Pitch PID recommendations are intentionally deferred
until after this report.

## Flown Configuration

```text
INS_GYRO_FILTER=18
ATC_RAT_RLL_P/I/D=0.150/0.150/0.00465
ATC_RAT_PIT_P/I/D=0.170/0.170/0.0051
ATC_RAT_YAW_P/I/D=0.180/0.018/0
PILOT_Y_EXPO=0.4
PILOT_Y_RATE=202.5
PILOT_Y_RATE_TC=0.10
FLOW_POS_X/Y/Z=-0.10/0/0.02
SERVO_BLH_RVMASK=2
PRX1_TYPE=0
```

No tuning parameter changed in flight.

## Flight Profile

| Metric | Result |
| --- | ---: |
| Armed duration | 319.9 s |
| GPS-log duration | 328.8 s |
| Path distance | 1,531 m |
| Maximum radius | 316.1 m |
| Maximum relative altitude | 26.2 m |
| Maximum groundspeed | 17.86 m/s |
| Mean path groundspeed | 4.66 m/s |
| Flight modes | Loiter, PosHold, Loiter, PosHold, Loiter |
| Energy consumed | 18.88 Wh |
| Capacity consumed | 820 mAh |
| Battery remaining | 99% -> 88% |
| Average power | 206.7 W |
| Maximum current | 19.6 A |

## Alerts and Estimator Health

No in-flight `ERR` record, EKF lane switch, vibration-compensation event, failsafe
mode change or gain change occurred. Both EKF cores reported GPS use. The recurring
`PTDS: waiting for RC options 301 and 302` message came from the still-installed pitch
D script; it did not change gains.

GPS remained 3D with 24-28 satellites and HDOP no worse than 0.6. GPS yaw was valid
for 81.2% of GPS messages; no in-flight heading-source error was reported. Processor
load was approximately 30%, with zero scheduler, internal and I/O errors.

## Vibration Envelope

No accelerometer clipping occurred on either IMU.

| Speed band | IMU0 VIBE median | VIBE p95 | VIBE maximum | Clips |
| ---: | ---: | ---: | ---: | ---: |
| 0-1 m/s | 7.1 m/s2 | 12.8 m/s2 | 24.5 m/s2 | 0 |
| 1-2 m/s | 7.5 m/s2 | 16.2 m/s2 | 22.9 m/s2 | 0 |
| 2-3 m/s | 9.7 m/s2 | 21.1 m/s2 | 24.0 m/s2 | 0 |
| 3-4 m/s | 11.4 m/s2 | 16.7 m/s2 | 21.2 m/s2 | 0 |
| 4-5 m/s | 12.0 m/s2 | 16.2 m/s2 | 30.8 m/s2 | 0 |
| 5-6 m/s | 13.4 m/s2 | 18.1 m/s2 | 26.6 m/s2 | 0 |
| 6-7 m/s | 15.0 m/s2 | 21.3 m/s2 | 30.1 m/s2 | 0 |
| 7-8 m/s | 17.1 m/s2 | 29.0 m/s2 | 32.5 m/s2 | 0 |
| 8-9 m/s | 19.0 m/s2 | 31.5 m/s2 | 35.0 m/s2 | 0 |
| 9-10 m/s | 20.5 m/s2 | 35.2 m/s2 | 42.3 m/s2 | 0 |
| 10-11 m/s | 22.5 m/s2 | 32.4 m/s2 | 42.8 m/s2 | 0 |
| 11-12 m/s | 22.1 m/s2 | 38.6 m/s2 | 46.8 m/s2 | 0 |
| 12-13 m/s | 35.6 m/s2 | 48.1 m/s2 | 61.2 m/s2 | 0 |
| 13-14 m/s | 25.9 m/s2 | 53.3 m/s2 | 56.3 m/s2 | 0 |
| 14-15 m/s | 45.9 m/s2 | 57.5 m/s2 | 60.5 m/s2 | 0 |
| 15-16 m/s | 63.6 m/s2 | 68.0 m/s2 | 68.9 m/s2 | 0 |
| 16-17 m/s | 60.5 m/s2 | 65.1 m/s2 | 65.3 m/s2 | 0 |
| 17-18 m/s | 65.7 m/s2 | 72.2 m/s2 | 76.6 m/s2 | 0 |

Whole-flight VIBE was `11.9/35.8/76.6 m/s2` median/p95/maximum on IMU0 and
`8.2/24.9/52.7 m/s2` on IMU1.

The repaired aircraft is materially safer than the incident configuration: it reached
17.86 m/s without clipping or EKF degradation. It is not vibration-clean at that
speed. VIBE increases rapidly above 12 m/s and strongly excites the pitch loop. The
recommended survey band remains 6-8 m/s. Ten m/s is a current upper operating gate;
12+ m/s is not qualified for routine survey use.

## Yaw Input-Softening Result

The only yaw change was:

```text
PILOT_Y_RATE_TC: 0.00 -> 0.10 s
```

| Metric | Before | After |
| --- | ---: | ---: |
| Whole-flight yaw ratio | 0.86 | 0.88 |
| Whole-flight correlation | 0.92 | 0.94 |
| Median event overshoot | 45.0% | 29.5% |
| Median event rise time | 182 ms | 278 ms |
| Median ring | 1.0 | 1.0 |
| WebTools peak/tail | 0.75/0.74 | 0.71/0.69 |
| Synthetic overshoot/ring | 0/0 | 0/0 |

The new time constant softened pilot command entry as intended without destabilizing
heading hold. Keep `PILOT_Y_RATE_TC=0.10` if the subjective feel also improved. Do not
change yaw PID. If it remains too sharp, test 0.15 s separately. If smoothing is good
but maximum yaw speed remains excessive, reduce `PILOT_Y_RATE` in a later isolated
test.

## Pitch Oscillation Characterization

Pitch response is stable in aggregate but has a repeatable 5.7-6.6 Hz tracking-error
component. The issue is strongly load- and vibration-dependent.

| Flight phase | Speed median/max | VIBE median/p95 | Pitch ratio/correlation | Error RMS/p95 | D p95/max | Dominant pitch-error peaks |
| --- | ---: | ---: | ---: | ---: | ---: | --- |
| Loiter 1 | 3.1/12.5 m/s | 11.2/34.7 | 1.04/0.80 | 2.8/5.8 deg/s | 0.011/0.039 | 6.53, 5.86 Hz |
| PosHold 1 | 16.1/17.9 m/s | 59.4/68.7 | 1.82/0.25 | 17.8/43.7 deg/s | 0.112/0.239 | 6.63, 5.84 Hz |
| Loiter 2 | 1.0/10.0 m/s | 10.3/20.0 | 1.12/0.60 | 13.6/30.5 deg/s | 0.056/0.199 | 5.82, 6.75 Hz |
| PosHold 2 | 8.1/14.1 m/s | 17.9/32.8 | 0.95/0.76 | 9.4/20.4 deg/s | 0.030/0.170 | 5.66, 6.36 Hz |
| Loiter 3 | 1.3/5.5 m/s | 8.1/12.1 | 0.89/0.61 | 3.5/7.2 deg/s | 0.005/0.050 | 6.28, 5.43 Hz |

Across 22 detected pitch events, median overshoot was 22.7%, but median ring count was
3.5 and p75 was 6.8. Several small or overlapping transitions produced inflated
percentage overshoot, so ring count, spectral peaks and absolute error are more useful
than the maximum event percentage.

Dmod remained 1.0. ArduPilot did not limit the pitch controller. The issue is not a
Dmod/slew-protection event. High-speed vibration increased D-term activity by roughly
an order of magnitude relative to clean low-speed Loiter.

This report intentionally makes no pitch PID change. The next discussion should decide
whether to reduce pitch P/I authority, reduce pitch D, or adjust both while preserving
the accepted low-speed transfer response. Any pitch change should be tested below
8 m/s first.

## Roll Controller

Only three qualifying roll events were present. Median overshoot/ring was 26.0%/1.0.
Whole-flight ratio/correlation was `1.21/0.81`, with zero D limiting. No new roll fault
was identified.

## Motors and Thermal Health

| Motor | Position | Command med/p95/max | RPM med/p95/max | RPM per command | Temp med/max |
| --- | --- | ---: | ---: | ---: | ---: |
| M1 | front-right | 1370/1448/1539 | 2731/3414/3799 | 7.40 | 22/25 C |
| M2 | back-right | 1418/1478/1584 | 3031/3546/4073 | 7.32 | 32/34 C |
| M3 | back-left | 1388/1467/1577 | 2825/3515/3865 | 7.34 | 36/39 C |
| M4 | front-left | 1465/1511/1610 | 3448/3649/4182 | 7.35 | 28/31 C |

RPM-per-command values remained tightly grouped from 7.32 to 7.40. Raw/filtered RPM
correlation was at least 0.994. Bidirectional-DShot telemetry error-rate p95 was zero
for all ESCs and p99 was at or below 0.17. No DataFlash `ERR`, RPM dropout or motor
fault occurred. M3 remained warmest at 39 C, which is acceptable.

IMU0 and IMU1 reached 41.9 and 42.4 C. All gyro and accelerometer health flags remained
true. Battery temperature was not available.

## Battery and Efficiency

Battery health remained true:

| Metric | Result |
| --- | ---: |
| Loaded voltage start/end/min | 23.64/23.26/22.73 V |
| Resting voltage start/end/min | 23.68/23.27/23.15 V |
| Maximum current | 19.6 A |
| Capacity consumed | 820 mAh |
| Energy consumed | 18.88 Wh |
| Battery remaining | 99% -> 88% |
| Average power | 206.7 W |

Preliminary level-flight efficiency:

| Groundspeed | Median power | Energy per distance |
| ---: | ---: | ---: |
| 6 m/s | 208 W | 9.6 Wh/km |
| 8 m/s | 209 W | 7.3 Wh/km |
| 10 m/s | 208 W | 5.8 Wh/km |
| 11 m/s | 202 W | 5.1 Wh/km |
| 12 m/s | 230 W | 5.3 Wh/km |
| 13 m/s | 230 W | 4.9 Wh/km |

Eleven m/s had the lowest measured power. Twelve to thirteen m/s had the best measured
energy per ground-distance, but those speeds exceed the current vibration envelope and
are not recommended for routine use. The 160 Wh usable-energy assumption implies about
46 minutes at the measured average power. Efficiency values are short-duration,
groundspeed-based and wind-sensitive.

## Supporting Sensors

- GPS remained 3D with 24-28 satellites and HDOP no worse than 0.6.
- GPS yaw was available for 81% of GPS records; no heading-source error occurred.
- Optical-flow quality was 108-226, median 184.
- The measured flow offset `FLOW_POS_Z=0.02` was active.
- Rangefinder status varied normally as altitude exceeded its useful range; maximum
  reported distance was 29.2 m.
- Processor load was approximately 30%, with zero scheduler, internal and I/O errors.

## Qualification Status

- Hover: passed.
- 4/6/8 m/s: passed.
- 10 m/s: passed in prior flight; vibration was higher in this flight during maneuvers.
- 12+ m/s: not qualified for routine operation.
- Automated survey mission: pending.
- Payload/AI computer integration: pending.
- Yaw input smoothing at 0.10 s: objectively improved; subjective confirmation pending.
- Pitch PID damping: review required before payload qualification.
