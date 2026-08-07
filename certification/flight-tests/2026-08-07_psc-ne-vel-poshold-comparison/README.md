# Scout-15 Stock Velocity Gains, `EK3_RNG_M_NSE`, and Loiter vs PosHold

**Aircraft:** Scout-15 quadcopter
**Flight event:** 2026-08-07
**Flight log:** `2026-08-07 12-17-52.bin`
**Objective:** Verify `PSC_NE_VEL_*` returned to firmware defaults and `EK3_RNG_M_NSE = 0.1`, and
compare Loiter against PosHold within one flight
**Outcome:** **`EK3_RNG_M_NSE` worked — altitude hold 2–3× tighter.** The horizontal gain change
produced **no resolvable effect**. The real limit was identified: **GPS position accuracy of 1.66 m**
**Decision:** Keep the stock gains and `EK3_RNG_M_NSE = 0.1`. Pursue RTK
**Parameter snapshot:** `certification/parameters/0807.param`

## Summary

All four intended parameters loaded this time, verified from the log before any interpretation.

The flight did three things. It confirmed the altitude change worked. It failed to resolve the
horizontal change. And it found that neither was ever going to be the answer, because **position
hold was limited by a 1.66 m GPS position estimate** the whole time.

## Configuration proof — read from the log

| Item | Value |
| --- | --- |
| Firmware | ArduCopter 4.8.0-dev, commit `32e43699` |
| `PSC_NE_VEL_P` / `_I` / `_D` | **1.0 / 0.5 / 0** — firmware defaults |
| `PSC_NE_POS_P` | 1.0 |
| `EK3_RNG_M_NSE` | **0.1** — loaded this time |
| `ATC_ANG_PIT_P` | 7.0 |
| `INS_GYRO_FILTER` | 27 |
| Rate PIDs | RLL 0.160/0.160/0.0052, PIT 0.160/0.160/0.0051 |
| Armed | 167.5 s |
| Modes | Loiter / PosHold alternated **seven times** |

Interleaving the modes within one flight makes the Loiter-vs-PosHold comparison valid — both
experience the same wind and the same GPS conditions.

## Result 1 — `EK3_RNG_M_NSE = 0.1` works

Measured on fully hands-off windows, throttle included in the stick-centred test:

| | EKF alt err rms | p2p |
| --- | --- | --- |
| 0.5 (`11-36-20`) | 0.079 m | 0.314 m |
| **0.1 (this flight)** | **0.025 – 0.044 m** | **0.130 – 0.181 m** |

**Two to three times tighter, consistent across both windows.** Rangefinder-measured sag also
improved, −42.5 → **−28.1 cm per 30 s**.

The later window (starting 126 s into the flight) sagged only **−4 cm over 53.7 s** — essentially
zero — while the earlier one (89 s) sagged −77 cm over 33.5 s. Barometer drift measured **−15 cm
per 30 s on both flights** and settles as the sensor equilibrates, which is consistent with the
thermal mechanism identified on `11-36-20`.

## Result 2 — the horizontal gain change is not resolvable

| | ampR | ampP | box | rms |
| --- | --- | --- | --- | --- |
| before, Loiter (85 s) | 0.329 | 0.253 | — | 0.163 m |
| after, Loiter (41 s) | 0.753 | 0.373 | 1.05 m | 0.163 m |

Roll wobble amplitude reads *higher* after the change. **This is not a result.**

**Two windows of identical config in the same flight differ by up to 3.8×** (PosHold windows at
0.166 and 0.623). That scatter swamps everything. Position rms is unchanged to three decimal places
(0.163 vs 0.163) and box sizes overlap completely.

The honest verdict is **"no resolvable change"**, not "worse". The gains are kept because stock is
a better-defined baseline than an unexplained elevation above it, not because they were shown to
help.

## Result 3 — PosHold measures better than Loiter

Same flight, interleaved, so conditions match:

| | ampR | ampP | box | rms |
| --- | --- | --- | --- | --- |
| Loiter (41 s) | 0.753 | 0.373 | 1.05 m | 0.163 m |
| **PosHold (36 s)** | **0.623** | **0.159** | **0.70 m** | 0.159 m |

PosHold is better on every metric, and pitch wobble is **57% lower**. This matches the pilot's
in-flight impression.

**Mechanism:** PosHold carries a wind-compensation estimator that Loiter does not, gated at
**0.10 m/s** (`POSHOLD_WIND_COMP_ESTIMATE_SPEED_MAX_MS`). PosHold's median speed was **0.093 m/s** —
just under the gate, so the estimator was updating. Loiter's was 0.155 m/s.

⚠ **Held loosely.** The 3.8× within-mode scatter applies here too. This is *consistent with* the
mechanism, not proof of it.

## Result 4 — the actual limit was GPS all along

```
GPS Status : 3D Fix — 100% of the flight, never RTK
RTCMFU     : 0 fragments      RTCMFD : 0 fragments
NSats      : 25               HDop   : 0.60
HAcc       : median 1.66 m, best 1.46 m
SAcc       : 0.040 m/s
```

Twenty-five satellites and HDop 0.60 look excellent, but **horizontal position accuracy was
1.66 m** — and the aircraft was being asked to hold to 0.16 m. Velocity accuracy at 0.040 m/s is
**41× better than position**, which is why the velocity loop behaves and the position loop wanders.

The aircraft held **0.16 m rms against a 1.66 m position estimate** — it was already outperforming
its own sensor. No gain change can beat that.

`RTCMFU` and `RTCMFD` both at zero proves no RTCM ever reached the flight controller. The
correction path was configured but not delivering.

### Optical flow — present, healthy, ignored

| | |
| --- | --- |
| `FLOW_TYPE` | 6 (DroneCAN), 17,634 records |
| `Qual` | median **131**, 100% of samples ≥ 50, **zero dropouts** |
| `EK3_SRC1_VELXY` | **3 (GPS)** — flow not fused |

A fully functional sensor doing nothing. Noted but **not** proposed as the fix: the firmware enum
`SourceXY` has no OpticalFlow option for `POSXY`, so flow can supply velocity only — and GPS
velocity was already excellent.

## Decision

1. **Keep `PSC_NE_VEL_* = 1.0 / 0.5 / 0`** and `EK3_RNG_M_NSE = 0.1`.
2. **Pursue RTK.** `HAcc` 1.66 m is the binding constraint and nothing in the tuning domain
   competes with fixing it.
3. Optical flow deferred — calibrate first, then evaluate as a secondary velocity source.

## Confidence limits

- **`EK3_RNG_M_NSE` result: ~85%.** Consistent across two independent windows, effect 2–3×.
- **Horizontal gain change: no claim.** Scatter exceeds effect.
- **PosHold better than Loiter: ~70%.** Right direction, right mechanism, insufficient windows.
- **GPS as the limit: ~95%.** `HAcc` and the zero RTCM counters are unambiguous.

## Health

Zero clips. Baro drift −15 cm/30 s. No failsafes, no EKF messages.

## Next step

Establish RTCM correction delivery and confirm `RTCMFU` climbs, then refly with a **≥ 60 s
continuous hands-off window** — every attempt so far has been too short to separate settling from
steady-state hold.
