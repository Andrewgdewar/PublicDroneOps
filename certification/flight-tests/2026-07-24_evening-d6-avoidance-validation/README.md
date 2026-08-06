# Scout-15 Evening D6 Avoidance Validation

**Date:** 2026-07-24

**Log:** `2026-07-24 20-53-19.bin`

**Firmware:** ArduCopter V4.8.0-dev (`32e43699`), MatekH743-bdshot

**Purpose:** Validate RC11 proximity-avoidance switching, `PRX_FILT=1`, raw D6
logging, Loiter slide behavior, and the aircraft's low-speed health after the earlier
short-hop review.

## Summary

This flight is a **functional pass** for the D6, the RC11 switch, the 1 Hz boundary
filter, and low-speed simple avoidance.

- RC11 switched cleanly between 1051 and 1951 with no middle state.
- The FC acknowledged `AvoidProximity HIGH` and `LOW` at the expected times.
- The D6 delivered raw and filtered sectors at 10 Hz with 100% `Good` health.
- UART4 received approximately 13.8 kB/s with zero dropped bytes.
- Simple avoidance became active for one 6.45-second interval while RC11 was high.
- Avoidance modified horizontal velocity and briefly requested backing away.
- The intervention was smooth, with a maximum velocity-vector change of 0.034 m/s.
- No EKF, motor, logging, scheduler, CAN, clipping, or vibration fault occurred.

The test does **not** validate head-on stopping or high-speed deflection. Groundspeed
never exceeded 1.06 m/s, and the obstacle was around 275 degrees relative to the
aircraft, so this was mainly a side/glancing interaction.

The embedded log shows `AVOID_MARGIN=1.5`; the parameter snapshot is synchronized to
that tested value.

## Flight Profile

| Metric | Result |
| --- | ---: |
| Armed duration | 73.5 s |
| Approx. airborne duration | 68.7 s |
| Mode | Loiter |
| Maximum relative altitude | 2.99 m |
| Maximum groundspeed | 1.06 m/s |
| Maximum climb / descent | +1.09 / -0.99 m/s |
| Mean / maximum current | 10.35 / 19.23 A |
| Mean / maximum power | 242 / 451 W |
| Energy consumed | 4.61 Wh |

## Flown Configuration

```text
AVOID_ENABLE = 3
AVOID_BEHAVE = 0
AVOID_MARGIN = 1.5
AVOID_ALT_MIN = 1
AVOID_ACCEL_MAX = 3
OA_TYPE = 0

RC11_OPTION = 40
RC11_MIN = 1051
RC11_MAX = 1951

PRX1_TYPE = 19
PRX1_CSUM = 1
PRX1_INT_THR = 20
PRX1_MIN = 0.8
PRX1_MAX = 12
PRX_FILT = 1
PRX_LOG_RAW = 1

LOIT_OPTIONS = 0
```

## RC11 Runtime Gate

| Event | Time | Result |
| --- | ---: | --- |
| RC11 high | 92.242 s | `AvoidProximity HIGH`, accepted |
| RC11 low | 129.041 s | `AvoidProximity LOW`, accepted |

RC11 distribution across the log:

- Low: 57.2%
- High: 42.8%
- Middle: 0%

This confirms a proper two-position mapping. Low disables runtime proximity authority;
high enables it. The D6 remains powered and logged in both states.

## Avoidance Intervention

The log contains 65 `SA` records: 64 active records plus one inactive marker.

| Metric | Result |
| --- | ---: |
| Active interval | 105.826-112.281 s |
| Active duration | 6.45 s |
| Back-away flag | 12 records |
| Desired velocity median / maximum | 0.148 / 0.535 m/s |
| Modified velocity median / maximum | 0.148 / 0.532 m/s |
| Velocity change median / p95 / maximum | 0.001 / 0.008 / 0.034 m/s |
| Closest filtered distance median / minimum | 1.66 / 1.25 m |
| Obstacle direction | approximately 267-286 degrees |
| Median altitude | 2.43 m |

The intervention was small because the approach was slow and mostly glancing. The
aircraft briefly entered the 1.5 m configured margin, then the avoidance controller
limited the obstacle-directed velocity component and requested backup where needed.
No abrupt command, oscillation, or large displacement was observed in the log.

## One-Hertz Boundary Filter

`PRX` and `PRXR` each logged 857 frames at exactly 10 Hz.

| Metric | Result |
| --- | ---: |
| Proximity health | 100% `Good` |
| Valid paired sector samples | 4,116 |
| Absolute raw/filtered difference median | 0.015 m |
| Absolute raw/filtered difference p95 | 0.253 m |
| Absolute raw/filtered difference maximum | 2.102 m |
| Observed stable-sector lag | approximately 0.1-0.2 s |

The filter roughly halved frame-to-frame distance noise in most sectors. During the
approach in sector 270, filtered distance generally trailed a steadily closing raw
distance by approximately 5-10 cm. Larger differences occurred on abrupt target or
sector changes, as expected from a low-pass filter.

`PRX_FILT=1` is accepted for continued low-speed testing. This flight does not prove
that 1 Hz is sufficient for high-speed avoidance.

## Flight-Control and Vibration Health

| Metric | Result |
| --- | ---: |
| Roll rate ratio / correlation | 1.17 / 0.86 |
| Pitch rate ratio / correlation | 1.09 / 0.74 |
| Roll / pitch D limiting | 0% / 0% |
| Roll attitude error RMS / p95 | 0.265 / 0.522 deg |
| Pitch attitude error RMS / p95 | 0.213 / 0.467 deg |
| Desired tilt median / p95 | 1.73 / 4.95 deg |

These results are materially cleaner than the afternoon windy flight and do not
support a PID change.

| Vibration | Median / p95 / maximum | Clips |
| --- | ---: | ---: |
| IMU0 vector | 6.32 / 7.62 / 8.55 m/s2 | 0 |
| IMU1 vector | 4.20 / 5.08 / 5.65 m/s2 | 0 |

## Motor, Navigation, and System Health

All four motors had continuous RPM telemetry, low error rates, and normal
temperatures:

| Motor | Position | RPM median / maximum | ESC error p95 / max | Max temp |
| --- | --- | ---: | ---: | ---: |
| M1 | front-right | 3000 / 3658 | 0.20 / 1.19% | 31 C |
| M2 | back-right | 2908 / 3600 | 0.04 / 0.24% | 40 C |
| M3 | back-left | 3475 / 3750 | 0.08 / 0.48% | 45 C |
| M4 | front-left | 3233 / 3700 | 0.04 / 0.24% | 36 C |

Additional health:

- GPS: 27-28 satellites, HDop 0.60.
- Optical-flow quality: 81 minimum, 113 median.
- IMU maximum temperature: 38 C.
- No DataFlash drops or `ERR` records.
- No UART receive drops.

## Decision and Limits

Keep:

```text
PRX_FILT = 1
PRX_LOG_RAW = 1
AVOID_MARGIN = 1.5
AVOID_BEHAVE = 0
LOIT_OPTIONS = 0
```

Use RC11 low by default and enable avoidance deliberately for low-speed Loiter tests.
This flight validates switching and gentle glancing interaction only. It does not
grant operational credit for:

- PosHold horizontal avoidance;
- Auto mission path planning (`OA_TYPE=0`);
- head-on stops;
- trees or obstacles outside the D6 scan plane;
- closing speeds above the tested 1.06 m/s.

The next staged test should use a broad, soft target with clear escape space:

1. RC11 high in Loiter at 2-3 m AGL.
2. Approach approximately head-on at 1 m/s and release the stick.
3. Repeat with a 30-degree glancing approach at 1 m/s.
4. Review `SA`, `PRX`, `PRXR`, `PSCN`, `PSCE`, `ATT`, and `RCIN` before increasing
   speed.