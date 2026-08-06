# Scout-15 AutoTune and Speed Review

**Date:** 2026-07-18  
**Aircraft:** Scout-15 quad, T-Motor MS1504 15x5.6 props  
**Log:** `2026-07-18 10-31-38.bin` (507,281,408 bytes)  
**Firmware:** ArduCopter V4.8.0-dev, MatekH743-bdshot  
**Purpose:** pitch and roll AutoTune evaluation followed by a 7/10/13 m/s automated speed sweep

## Verdict

- **Keep the original pitch and roll tune:** rate P/I `0.135`, D `0.0036`, angle P `4.5`.
- Pitch AutoTune completed and felt acceptable, but it did not outperform the nearby original-gain window and was not saved.
- The first roll AutoTune attempt failed Angle-P determination. It had no completed gains to compare.
- The second roll attempt completed, but the stronger test window reproduced the pilot's bad feel: **14.63 deg roll attitude-error RMS, 26.16 deg p95 and 76.7% event overshoot**. Reverting was correct.
- Neither AutoTune result persisted. The log begins and ends with the original gains.
- `INS_GYRO_FILTER` remained **18 Hz**; this flight did not test the planned 25 Hz setting.
- The automated 7/10/13 m/s mission was exceptionally smooth. **10 m/s remains the conservative baseline; 13 m/s is the better measured transit/range candidate.**

## Flight sequence

### Flight 1 - pitch

| Event | Log time |
|---|---:|
| AutoTune started | 02:24.4 |
| Pitch complete / Success | 06:18.4 |
| Tuned gains selected | 06:27.7 |
| AutoTune stopped, PosHold selected | 07:36.5 |
| Disarmed in Loiter | 08:15.9 |

AutoTune proposed:

| Parameter | Original | AutoTune |
|---|---:|---:|
| Pitch rate P/I | 0.135 | 0.027 |
| Pitch rate D | 0.0036 | 0.0026 |
| Pitch angle P | 4.5 | 2.295 |
| Pitch max acceleration | existing | 27,829 centideg/s2 |

There were **zero pilot-override messages**. This was a valid completion, unlike the wind-contaminated July 3 attempt, but its gains are still much lower than stock.

### Flight 2 - roll

| Event | Log time |
|---|---:|
| First AutoTune started | 09:25.1 |
| Angle-P determination failed | 15:22.4 |
| AutoTune stopped | 16:22.0 |
| Test request rejected as incomplete | 16:48.4 |
| Second AutoTune started | 17:35.1 |
| Roll complete / Success | 21:30.1 |
| Original gains restored | 21:35.7 |
| Tuned gains selected | 21:37.5 |
| AutoTune stopped | 22:21.0 |
| Original/tuned toggles and mode exits | 22:26.3-22:50.7 |
| Disarmed in PosHold | 23:55.8 |

The first attempt did **not** finish before the switch to Loiter. The second attempt restarted AutoTune and proposed:

| Parameter | Original | AutoTune |
|---|---:|---:|
| Roll rate P/I | 0.135 | 0.030 |
| Roll rate D | 0.0036 | 0.0017 |
| Roll angle P | 4.5 | 1.869 |
| Roll max acceleration | existing | 25,899 centideg/s2 |

## Tune A/B evidence

AutoTune test gains may be loaded internally without a `PARM` write. Windows below are keyed to the explicit `Pilot Testing gains` and `original gains restored` messages.

| Window | Rate RMS | Normalized error | Attitude RMS | Attitude p95 | Overshoot | Ring |
|---|---:|---:|---:|---:|---:|---:|
| Pitch tuned | 9.37 deg/s | 0.71 | 1.87 deg | 4.59 deg | 30.5% | 1.5 |
| Pitch original after stop | 6.16 deg/s | 0.85 | **1.23 deg** | **2.77 deg** | 32.4% | 0.0 |
| Roll tuned, gentle window | 8.82 deg/s | 0.56 | 1.68 deg | 4.18 deg | 22.1% | 2.0 |
| Roll tuned, rejected window | **37.25 deg/s** | **1.19** | **14.63 deg** | **26.16 deg** | **76.7%** | 1.0 |
| Roll original after stop | 7.95 deg/s | 0.82 | **1.56 deg** | **4.08 deg** | 42.7% | 2.0 |

Pitch was flyable, but the data does not justify replacing stock. The rejected roll window is unequivocally poor. The low AutoTune rate and angle gains weakened the control cascade enough to produce large tracking excursions when exercised more strongly.

### Save-state proof

Firmware saves a completed AutoTune at disarm only when:

1. the aircraft is still in AutoTune and no gain-test command was used; or
2. the gain-test switch is selecting tuned gains at disarm.

Both flights exited AutoTune before disarm. Final embedded parameters remained:

- `ATC_RAT_PIT_P/I/D = 0.135 / 0.135 / 0.0036`
- `ATC_ANG_PIT_P = 4.5`
- `ATC_RAT_RLL_P/I/D = 0.135 / 0.135 / 0.0036`
- `ATC_ANG_RLL_P = 4.5`

No parameter recovery is required.

## Automated speed sweep

Mission sequence was 7 m/s southeast, 10 m/s northwest, and 13 m/s southeast over the same line. A later 18 m/s command immediately preceded RTL and did not produce a valid steady leg.

The table excludes the first and last five seconds of each leg.

| Command | Actual | Direction | Power | Current | Energy | Velocity RMSE | Cross-track p95 | Roll/Pitch error RMS | Vibe XYZ median | Clips |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 7 m/s | 7.01 m/s | 125 deg | 199.5 W | 9.28 A | 7.91 Wh/km | 0.20 m/s | **0.09 m** | 0.12 / 0.29 deg | 7.2 / 10.1 / 4.6 | 0 |
| 10 m/s | 10.00 m/s | 303 deg | 203.6 W | 9.54 A | 5.66 Wh/km | 0.27 m/s | **0.21 m** | 0.11 / 0.13 deg | 9.3 / 11.5 / 6.1 | 0 |
| 13 m/s | 12.80 m/s | 121 deg | 210.4 W | 9.88 A | 4.57 Wh/km | 0.60 m/s | **0.22 m** | 0.17 / 0.29 deg | 13.3 / 16.6 / 8.3 | 0 |

All three legs were smooth. Actual speed matched target closely, position-controller cross-track stayed below 0.22 m p95, attitude error stayed below 0.3 deg RMS, and no accelerometer clipping occurred.

The exact Wh/km values are **not wind-corrected**: 7 and 13 m/s were flown southeast while 10 m/s was flown northwest. They prove that 13 m/s remained efficient and controlled, but not a precise still-air range curve.

### Operational speeds

| Use | Recommendation | Basis |
|---|---:|---|
| Station keeping / maximum endurance | Hover or low speed | Cruise legs do not determine the minimum-power hover point. Today's level low-speed power was approximately 237-243 W outside the mission. |
| Survey baseline | **10 m/s** | Best balance of low vibration, 0.13 deg pitch/roll RMS and 0.21 m cross-track p95. Existing `WP_SPD=10`. |
| Efficient transit | **13 m/s** | Only 3.3% more power than 10 m/s while moving 28% faster; still smooth with zero clipping. |
| Loiter maximum | Keep `LOIT_SPEED_MS=12.5` | Consistent with the validated 13 m/s envelope; this is a limit, not the normal station-keeping speed. |
| RTL | Current default (`RTL_SPEED_MS=0`) | Inherits waypoint speed. Keep the conservative 10 m/s behavior until paired-direction testing is complete. |

Using a 160 Wh planning budget, the directional measurements imply approximately 47.2 minutes / 28.3 km at 10 m/s and 45.6 minutes / 35.0 km at 12.8 m/s. These are idealized electrical projections, not dispatch limits.

## Aircraft health

| Check | Result |
|---|---|
| Armed time | 7.38 + 15.15 + 3.81 = **26.35 min** |
| Energy consumed | **103.4 Wh**, approximately 4,613 mAh across three flights |
| Battery minimum | **21.11 V**, 3.52 V/cell under load |
| Peak current | **21.05 A** |
| GPS | 27-28 satellites, HDOP 0.5-0.6, 3D fix throughout |
| IMU clipping | **0 on both IMUs** |
| ESC temperature | **50 C maximum** (ESC 2), acceptable |
| FC/IMU temperature | 43.36 C maximum |
| Scheduler/internal errors | 0 |
| GCS failsafe | One occurrence 13 s after final disarm; no flight effect |

Primary IMU vibration increased with speed but remained controlled. At 13 m/s, p95 X/Y/Z was 20.0/26.2/12.7, below the 30 m/s2 guideline on all axes. The higher ESC `Err` values at final spin-up are bdshot eRPM telemetry error percentages; they decayed to approximately 0.1-0.5% and are not motor-fault counters.

## Actions

1. Keep the original pitch and roll gains. Do not load either AutoTune result.
2. Keep `WP_SPD=10` as the survey/default autonomous speed.
3. Use 13 m/s for optional clear-area transit after one repeat mission confirms it in both directions.
4. Repeat each candidate speed in both directions on the same battery and average paired legs before claiming a still-air range optimum.
5. Treat the 25 Hz gyro-filter check as still outstanding; the aircraft remained on 18 Hz throughout this session.
6. No mechanical, propulsion or battery corrective action is indicated by this log.