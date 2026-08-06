# Scout-15 Fixed-Tune PID Reference Review

**Date:** 2026-07-19  
**Log:** `2026-07-19 11-52-51.bin`  
**Armed time:** 215.2 s  
**Configuration:** 25 Hz gyro LPF, stock roll/pitch P/I `0.135`, D `0.0036`

## Configuration proof

- `INS_GYRO_FILTER=25`
- `INS_RAW_LOG_OPT=0`
- harmonic notch enabled
- roll/pitch angle P `4.5`
- roll/pitch rate P/I `0.135`, D `0.0036`, FLTT/FLTD `10`, SMAX `50`
- `SID_AXIS=0`, `RC16_OPTION=0`
- no AutoTune messages or gain changes

The flight used Loiter for 103.7 s of maneuvers, PosHold for 49.7 s, then RTL.
It did not enter AltHold. Loiter and PosHold are analyzed separately below, and RTL
is excluded from the maneuver assessment.

## SmartTune CLI result

SmartTune CLI 3.0.3 produced no PID recommendations:

- quality score: 65/100 (`MARGINAL`)
- detected pitch steps: 0
- detected roll steps: 0
- ARX fit quality: 0.0% on all axes
- hidden frequency-response path: 0 valid windows out of 1,404

The aircraft was sufficiently excited. Desired rates reached 42 deg/s roll and
57 deg/s pitch, and the local ramp-aware analyzer found 17 roll and 28 pitch events.
SmartTune instead requires a one-sample jump of 30% of the maximum desired rate:
12.6 deg/s for roll and 17.1 deg/s for pitch. ArduPilot input shaping limited the
largest one-sample changes to 1.01 and 0.92 deg/s respectively.

A non-persistent ramp-aware SmartTune experiment was also rejected. Results changed
drastically with the arbitrary detector horizon, including 61% to more than 1000%
reported overshoot and false yaw events. SmartTune must not generate gains for this
flight. This is a tool-model mismatch, not a reason to fly more aggressively.

## Ramp-aware response

Metrics measure actual-minus-desired tracking error around desired-rate level
transitions. They exclude RTL.

| Mode | Axis | Events | Rate ratio | Rise | Median overshoot | Median ring |
| --- | --- | ---: | ---: | ---: | ---: | ---: |
| Loiter | Pitch | 16 | 0.94 | 80 ms | 39.6% | 1.0 |
| Loiter | Roll | 10 | 0.96 | 88 ms | 33.6% | 2.5 |
| PosHold | Pitch | 11 | 1.00 | 110 ms | 30.9% | 2.0 |
| PosHold | Roll | 7 | 0.92 | 68 ms | 21.2% | 2.0 |

The rate amplitude is already close to unity, so reducing P is not supported. Rise
time is also adequate. Overshoot and repeated sign changes show remaining damping
room, especially on pitch.

## Control headroom and health

- Dmod minimum: 1.0 on both axes; no D limiting
- maneuver output maximum: 0.139 or less in Loiter/PosHold
- maneuver D-term maximum: 0.036 or less
- IMU clipping: zero
- vibration vector median/p95: 10.92 / 20.37 m/s2
- battery current median/p95/max: 9.50 / 12.91 / 19.31 A
- minimum voltage: 23.08 V
- maximum IMU temperature: 30.3 C
- maximum ESC temperature: 35 C
- ESC telemetry error p95: 0.04% or less

## Decision

Keep roll and pitch P/I at `0.135`. Do not apply SmartTune's prior P reductions or I
increase. Use this log as the fixed baseline for a conservative, pitch-only damping
candidate:

```text
ATC_RAT_PIT_D: 0.0036 -> 0.0040
```

This is an 11.1% increase with substantial measured D/output headroom. It is a test
candidate, not a locked tune. Fly the same separated pitch sequence and accept it
only if overshoot/ringing improve without new high-frequency oscillation, D limiting,
temperature increase or degraded feel. Roll remains unchanged until pitch is decided.