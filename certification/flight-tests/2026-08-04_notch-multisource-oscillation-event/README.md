# Scout-15 Harmonic Notch MultiSource Oscillation Event

**Aircraft:** Scout-15 quadcopter
**Flight event:** 2026-08-04
**Flight log:** `2026-08-04 19-12-23.bin`
**Event classification:** Control oscillation induced by filter configuration
**Aircraft outcome:** Two short hops, both aborted. No crash, no damage, zero clips
**Corrective action:** `INS_HNTCH_OPTS` reverted 6 → 0; `INS_HNTCH_HMNCS` restored 1 → 3
**Root cause:** Confirmed from log — see below
**Verification:** [`../2026-08-04_filter-review-raw-imu/README.md`](../2026-08-04_filter-review-raw-imu/README.md)

## Summary

`INS_HNTCH_OPTS` was set to 6 — bit 1 (MultiSource, one notch per motor) plus bit 2 (update notch
coefficients at loop rate). The aircraft oscillated violently at **5.2 Hz** on both hops and was
landed immediately.

Roll rate error rose from a nominal ~4.9 deg/s to **26.53 deg/s**, a 5.4x increase. Peak roll rate
reached 90.5 deg/s. The oscillation was not a new mode: it was the airframe's existing ~5.5 Hz
resonance driven far harder than normal.

CPU was **not** the cause. The filter chain itself did not degrade.

## Configuration during event

| Item | Value |
| --- | --- |
| Firmware | ArduCopter 4.8.0-dev, commit `32e43699` |
| `INS_HNTCH_OPTS` | **6** (bit 1 MultiSource + bit 2 loop-rate update) |
| `INS_HNTCH_HMNCS` | **1** — firmware reduced it from 3, see below |
| `INS_HNTCH_MODE` | 3 (ESC telemetry) |
| `INS_HNTCH_BW` / `ATT` / `FREQ` | 15 / 40 / 25 |
| `INS_GYRO_FILTER` | 18 |
| `INS_RAW_LOG_OPT` | 9 |
| `SCHED_LOOP_RATE` | 400 Hz |
| Rate PIDs | unchanged: RLL 0.150/0.150/0.0052, PIT 0.150/0.150/0.0051 |

## Root cause — notch centre frequency collapse

ESC telemetry arrives at **100 Hz per motor** (`SERVO_BLH_TRATE=10`, measured from `ESC` message
intervals: 10.00 ms median on all four instances). Bit 2 requests notch coefficient updates at the
**400 Hz** loop rate. Three of every four loop iterations therefore had no fresh RPM, and the notch
centre frequency collapsed to zero and back:

```
FTN.NDn = 4                 4 notches, one per motor — MultiSource active
FTN.NF1..NF4  median 0.00   p95 55-58 Hz   std 23-24 Hz
```

A notch whose centre slams between 0 and 57 Hz hundreds of times per second is not a linear filter.
It injects broadband energy and destroys phase coherence across the control bandwidth, which drove
the 5.5 Hz airframe mode.

The ArduPilot documentation precondition for bit 2 is explicit — *"Only valid if frequency source
updates at loop rate, ie Bi-Directional DShot telemetry."* The aircraft does use bidirectional
DShot, but its telemetry rate is 100 Hz, not 400 Hz. **The precondition was assumed satisfied from
the hardware type rather than checked against the measured rate.**

## Secondary finding — MultiSource cost the 2nd harmonic

`HAL_HNF_MAX_FILTERS` caps total notch count. With four motors under MultiSource the firmware
silently reduced `INS_HNTCH_HMNCS` from 3 to 1, giving `NDn = 4` (4 motors x 1 harmonic). Second
harmonic coverage was lost entirely in exchange for marginally better centring.

The reduced value was **written back to the saved parameter file** and had to be manually restored.

## Not the cause

| Signal | Value | Interpretation |
| --- | --- | --- |
| `PM.Load` | 350 / 1000 median, 374 max | 65% headroom |
| `PM.NLon` | max 1 | scheduler effectively never overran |
| `PM.Mem` | 271 KB free | ample |
| Clips | 0 | no accelerometer saturation |

## Measured effect

| Flight | Roll peak | Roll rate err rms |
| --- | --- | --- |
| 08-03 15-03 baseline | 6.07 Hz | 4.66 deg/s |
| 08-03 15-58 baseline | 6.02 Hz | 4.92 deg/s |
| **08-04 19-12 event** | **5.2 Hz** | **26.53 deg/s** |
| 08-04 19-57 recovery | 5.56 Hz | 4.09 deg/s |

Spectral band power on the roll axis, 4–7 Hz: **1.483** on the following good flight versus
**427.9** during the event — a 289x increase, accounting for **98.25%** of all gyro energy.

Critically, the filter chain performed **identically** in both flights: 25–50 Hz at −23.3 dB,
motor fundamental at −35.0 dB, 140+ Hz at −46.3 dB. The filtering never degraded; the energy was
injected into the airframe mode.

## Other log observations

- `PreArm: Gyros inconsistent` — consistent with the two IMUs seeing different motion under
  violent oscillation
- Flight controller rebooted mid-session
- `RC13: MotorEStop HIGH` and `PreArm: Motors Emergency Stopped` — deliberate pilot safety layer

## Corrective action taken

```
INS_HNTCH_OPTS    6 -> 0
INS_HNTCH_HMNCS   1 -> 3    (restore after firmware reduction)
```

Verified against the last known-good flight across all 1,339 parameters: no remaining differences
in the control or filter path.

## Limits adopted

- **`INS_HNTCH_OPTS` stays at 0.** Bit 2 is unusable at this ESC telemetry rate. Bit 1 is not worth
  the lost harmonic.
- **Change one thing per flight.** This flight changed three variables at once; the cause had to be
  inferred rather than read directly.
- **Check preconditions against measured data.** The 100 Hz ESC rate was present in a log that had
  already been analysed before this configuration was flown.

## Analysis methods

`tools/logtools/pidreview.py` — `filterreview` for pre/post filter spectra from raw IMU logging,
`segments` for gain attribution. `FTN` notch frequency and `ESC` telemetry intervals read directly
via pymavlink.
