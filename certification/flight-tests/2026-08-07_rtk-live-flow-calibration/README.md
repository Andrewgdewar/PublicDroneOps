# Scout-15 RTK Fixed and Optical Flow Calibration

**Aircraft:** Scout-15 quadcopter
**Flight event:** 2026-08-07
**Flight log:** `2026-08-07 13-23-25.bin`
**Objective:** First flight with live RTK corrections, and run the built-in optical flow scale
calibration
**Outcome:** **Both succeeded.** `HAcc` **1.66 → 0.040 m** at 100% RTK Fixed. Flow calibration
converged with fitness 0.04 and revealed the sensor was reading **~25% low**
**Decision:** Adopt both. Position-hold assessment deferred — no valid window existed
**Parameter snapshot:** `certification/parameters/0807.param`

## Summary

Two objectives, both met, and they reinforced each other: RTK sharpens the EKF velocity and height
states that the flow calibration's `los_pred` term depends on, so running the calibration on the
first RTK flight produced a better fit than it would have otherwise.

The flight does **not** answer whether RTK improves position hold. That question needs a dedicated
sortie and is explained below.

## Configuration proof — read from the log

| Item | Value |
| --- | --- |
| Firmware | ArduCopter 4.8.0-dev, commit `32e43699` |
| GPS | UM982, `GPS1_TYPE` 25 (Unicore moving-baseline NMEA), R4.10Build11826 |
| `GPS_INJECT_TO` | 127 (all instances) |
| `EK3_FLOW_USE` | 1 (Navigation) |
| `EK3_SRC_OPTIONS` | 1 (FuseAllVelocities) |
| `EK3_SRC2_VELXY` | **0 — flow deliberately NOT fused during calibration** |
| `FLOW_TYPE` | 6 (DroneCAN) |
| `RC12_OPTION` | 158 (OPTFLOW_CAL) |
| Mode | PosHold 95.0 → 196.6 s, Loiter thereafter |
| Armed | 82.4 → 231.0 s |

## Result 1 — RTK Fixed

```
GPS Status : RTK Fixed — 100% of the log (809 samples)
RTCMFU     : 639 fragments used        RTCMFD : 62 discarded  (~9% loss)
HAcc       : median 0.040 m   min 0.010 m   max 0.070 m
SAcc       : 0.040 m/s
```

| | before | after | |
| --- | --- | --- | --- |
| `HAcc` | 1.66 m | **0.040 m** | **41×** |

Corrections were sourced from a public community NTRIP caster over a **37 km baseline** — near the
practical limit for holding a fixed solution, yet Fixed held for the entire flight with no
observed reversion to Float. The ~9% fragment loss is unremarkable link attrition and did not
interrupt the solution.

**Prior state, for contrast:** `RTCMFU` and `RTCMFD` were both **exactly zero** on every earlier
flight — not a single RTCM fragment had ever reached the flight controller. No aircraft-side
parameter was ever at fault; `GPS_INJECT_TO = 127` was already correct, `AP_GPS_NMEA` inherits the
base `inject_data()` which writes straight to the port, and `is_rtk_rover()` returns true only for
`UBLOX_RTK_ROVER` and `UAVCAN_RTK_ROVER`, so a moving-baseline Unicore is never skipped. The fault
was entirely upstream — most likely a client connected without selecting a mountpoint, which
returns the caster's sourcetable instead of a correction stream.

## Result 2 — optical flow calibration

Run with `RC12_OPTION = 158`, in PosHold at **3.06 m mean height** over grass.

```
FlowCal: Started
FlowCal: x:12% ... x:100%           (15 s)
FlowCal: y:14% ... y:78%            (after repositioning)
FlowCal: samples collected
FlowCal: scalarx:1.338 fit:0.04
FlowCal: scalary:1.312 fit:0.04
FlowCal: FLOW_FXSCALER=338, FLOW_FYSCALER=311
```

| | value | acceptance |
| --- | --- | --- |
| scalar X | **1.338** | 0.20 – 4.0 ✓ |
| scalar Y | **1.312** | 0.20 – 4.0 ✓ |
| fitness | **0.04** | ≤ 0.5 ✓ |

**The sensor was reading approximately 25% low** (1 ÷ 1.338 = 0.75). Had flow been enabled as a
velocity source before calibrating, the EKF would have been fused velocities a quarter too small.
Calibrating first was the correct order and it mattered.

### Flight profile during calibration

| | |
| --- | --- |
| height | 3.06 m mean (1.26 – 4.27) |
| roll | ±27°, peak rate **70 °/s** |
| pitch | ±27°, peak rate **66 °/s** |
| flow `Qual` | median 129, **min 111** |
| clips | **0** |
| VIBE max | 21.72 / 22.85 / 13.93 |

Elevated VIBE is expected during deliberate aggressive wobbling and produced no clipping.

### Method notes worth keeping

- Sample gate requires **≥ 20 °/s** on roll or pitch (gyro *and* flow) and **< 20 °/s** yaw.
- **Yaw between axes is harmless** — the yaw check discards individual samples only, it does not
  reset state. Cycling the switch LOW→HIGH *does* wipe both buffers.
- **Rotate in place, do not translate.** The residual is
  `body_rate + (flow_rate × scalar) − los_pred`, and `los_pred` comes from the EKF's own velocity
  and height states. Pure rotation drives `los_pred` toward zero and makes the fit self-contained.
- Only **50 samples per axis** are needed; 30 s per axis is generous.
- Calibration works with `EK3_SRC2_VELXY = 0`. `FuseOptFlow()` is called unconditionally and
  `really_fuse` gates only the state update — the calibration sample is stored outside that gate.

## Why position hold is not assessed here

The two longest hands-off windows after calibration were **8.7 s and 8.1 s**, and both are
**braking transient, not hold**:

| window | distance from start at t = 0, 2, 4, 6, 8 s |
| --- | --- |
| 163.8 – 171.9 | 0.00 → **2.81** → 1.33 → 1.31 → 1.23 |
| 203.5 – 212.2 | 0.00 → 1.28 → 1.29 → 1.52 → **1.67** |

The first travels 2.81 m out and returns; the second carries momentum then drifts. `PHLD_BRK_RATE`
8 °/s and `LOIT_BRK_ACC_M` 1 m/s² make PosHold deceleration deliberately gentle on an aircraft this
size, so the entire window is spent settling.

**No conclusion about RTK's effect on position hold is drawn from this flight.** The box and rms
figures those windows produce measure deceleration and must not be compared against hover data.

## Decision

1. **RTK adopted.** Corrections confirmed flowing and Fixed held throughout.
2. **`FLOW_FXSCALER = 338` / `FLOW_FYSCALER = 311` adopted**, saved automatically by the calibrator.
3. **`EK3_SRC2_VELXY` remains 0.** Flow fusion is the next flight's single variable.

## Confidence limits

- **RTK working: ~99%.** 100% Fixed, `HAcc` 0.040 m, corrections counted.
- **Calibration valid: ~90%.** Fitness 0.04 is an order of magnitude inside the threshold, and both
  axes agree to 2%.
- **Position hold: no claim.** Insufficient valid data.
- **Expected benefit of enabling flow: modest.** Flow supplies velocity only — the `SourceXY` enum
  has no OpticalFlow option for `POSXY` — and GPS velocity is already 0.040 m/s.

## Health

Zero clips across the flight. Flow quality never dropped below 111. No failsafes. Flight ended with
a normal motor-emergency-stop shutdown.

## Next step

**Dedicated hover, ≥ 60 s continuous hands-off**, current configuration, to establish the RTK
position-hold baseline. Then a second sortie with `EK3_SRC2_VELXY = 5` as the only change.

## Open item

Altitude sag reported during large, fast tilt movements. Steady hover is unaffected (0.025–0.044 m
rms). Rangefinder tilt compensation is confirmed present in source —
`alt = cos(tilt) × slant_range`, clamped at cos(45°) — but it assumes flat ground, and at 3 m with
27° of tilt the beam lands **1.5 m laterally** onto different terrain. The ground on this flight was
not flat. A softer vertical acceleration loop (`PSC_D_ACC` reduced 2.95× on 08-06) is the competing
explanation. The two separate by reflying over flat ground.
