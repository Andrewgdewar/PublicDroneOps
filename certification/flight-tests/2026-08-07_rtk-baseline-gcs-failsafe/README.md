# Scout-15 RTK Position-Hold Baseline — and a GCS Failsafe Incident

**Aircraft:** Scout-15 quadcopter
**Flight event:** 2026-08-07
**Flight log:** `2026-08-07 14-28-49.bin`
**Objective:** Establish the position-hold baseline with RTK Fixed, using long continuous
hands-off windows
**Outcome:** **RTK made no measurable difference to position hold.** The prediction is falsified.
The cause was instead identified as a **position-controller cascade oscillation at 0.19–0.20 Hz**.
An unrelated **GCS failsafe** triggered an unplanned RTL climb
**Decision:** `PSC_NE_POS_P` 1.0 → **0.5**. `FS_OPTIONS` 0 → **16**
**Parameter snapshot:** `certification/parameters/0807.param`

## Summary

Two armed periods in one log. The first ended when a telemetry dropout triggered RTL; the second
ran to a battery failsafe.

The flight achieved its measurement objective and produced a **negative result on the main
hypothesis** — RTK did not improve position hold — which then made the real cause measurable for
the first time.

## Configuration proof — read from the log

| Item | Value |
| --- | --- |
| Firmware | ArduCopter 4.8.0-dev, commit `32e43699` |
| GPS | **RTK Fixed, 100%** · `HAcc` median **0.030 m** · RTCMFU 1338 |
| `PSC_NE_POS_P` | 1.0 |
| `PSC_NE_VEL_P` / `_I` / `_D` | 1.0 / 0.5 / 0 |
| `EK3_SRC2_VELXY` | 0 — flow not fused, as intended |
| `FLOW_FXSCALER` / `FYSCALER` | **338 / 311 — intact**, calibration not disturbed |
| `RC12_OPTION` | 0 — flow-cal switch cleared, no calibration ran |
| `FS_GCS_ENABLE` / `FS_GCS_TIMEOUT` | 1 (RTL) / 5 s |
| `FS_OPTIONS` | **0** |
| `RTL_ALT_M` | **30** |
| `MOT_THST_HOVER` | 0.1797 |

## Result 1 — RTK did not improve position hold

Hands-off windows ≥ 20 s, single mode, airborne, excluding the failsafe period:

| | pre-RTK (`HAcc` 1.66 m) | **RTK Fixed (`HAcc` 0.030 m)** |
| --- | --- | --- |
| Loiter box | 1.05 m | 1.22 m |
| Loiter rms | 0.163 m | 0.219 m |
| PosHold box | 0.70 m | 0.71 m |
| PosHold rms | 0.159 m | 0.195 m |

**A 41× improvement in position accuracy produced no improvement in position hold.** The apparent
small worsening lies inside the within-flight scatter — the two Loiter windows on this flight differ
by 87% on rms — so the correct reading is **no resolvable change in either direction**.

### Prediction falsified, and recorded as such

The prior record predicted: *"If the wander was GPS position noise being chased, box and rms should
fall substantially."* They did not. **GPS position noise was never the limiting factor.**

## Result 2 — the wander is a control-loop oscillation

With estimate noise eliminated by RTK, the remaining causes are wind or the controller. Wind was
measured and excluded — mean lean angle in the hands-off windows was **0.74–1.14°**, essentially
calm.

Measured on the position controller directly:

| window | pos err rms | osc freq | vel loop gain | vel loop phase | vel_err / vel_tgt |
| --- | --- | --- | --- | --- | --- |
| 222–258 s | 0.129 m | **0.20 Hz** | 1.18 | **−92°** | 1.50 |
| 258–284 s | 0.157 m | **0.19 Hz** | 1.13 | **−91°** | 1.56 |

**Gain above unity at −90° phase, with the outer loop contributing a further −90° from integrating
velocity into position — total ≈ −181°.** That is the oscillation condition, measured twice with
close agreement.

Velocity target-to-actual correlation was **−0.03 and −0.05**. ArduPilot's own Loiter documentation
states the pass criterion as *"the actual velocities will track the desired velocities"*. By that
criterion this tune fails.

### Mechanism — the two loops share a bandwidth

```
PSC_NE_VEL_P = 1.0  ->  P acting on an integrator, crossover 1.0 rad/s = 0.159 Hz, -90 deg there
PSC_NE_POS_P = 1.0  ->  outer crossover also ~0.159 Hz
```

Predicted oscillation ~0.16 Hz; **measured 0.19–0.20 Hz**, agreeing within 20%. A cascade requires
the inner loop to be roughly 3–5× faster than the outer. These are 1:1.

`tuning.md` §8.3 already prescribes the remedy for this signature — *"< 1 Hz → position / outer loop
→ `PSC_POSXY_*`, `PSC_VELXY_*` −50%"* — and quotes issue #128228 describing the identical
behaviour: *"the FC requests a desired roll which changes quickly and out of phase with the real
roll."* **The velocity gains were already at the −50% values from that worked example. The position
gain had never been touched.**

### Circular or linear?

| window | N/E phase | N:E amplitude |
| --- | --- | --- |
| 222–258 s | +99° | 1.09 |
| 258–284 s | +166° | 0.67 |

A heading-error orbit ("toilet-bowl") would show ~90° with equal amplitudes **consistently**. It
does not — the phase wanders between windows, which is the signature of a lightly damped 2D
oscillation excited by small disturbances. Yaw error is retained as a secondary candidate only.
rms radius 0.175–0.283 m, consistent with the pilot's report of a ~1.5 m circle.

## Incident — GCS failsafe caused an unplanned 4 m climb

```
144.5s  GCS Failsafe            Subsys 8 ECode 1
144.5s  MODE -> RTL (rsn 5)
145.5s  GCS Failsafe Cleared
147.8s  MODE -> Loiter          (pilot intervention)
```

Altitude went **1.78 → 5.67 m in ~3 s**, throttle stepping 0.18 → 0.23. `RTL_ALT_M = 30`, so the
aircraft was climbing toward **30 m** and was stopped by pilot action at 5.67 m.

### Root cause

The MAVLink channel log field `mgs` records the timestamp of the last GCS heartbeat received:

| t (s) | `mgs` |
| --- | --- |
| 135.6 | 135511 |
| 138.6 | 135511 — 3 s gap, under timeout |
| 139.6 | 139515 — one heartbeat |
| 144.6 | 139515 — **5.0 s gap, `FS_GCS_TIMEOUT` reached** |
| 145.6 | 145520 — link returns |

`rxdp` (dropped/corrupt packets) was **0** throughout — nothing was garbled, data simply stopped
arriving. RTCM stalled in the same window. A third 5 s gap occurred at 155.6–160.6 s, after disarm.

**The telemetry uplink was intermittently unavailable.** This is a ground-station link problem, not
an aircraft problem.

### Root cause refined — RTCM congestion, measured across four flights

An initial reading attributed the stall to ground-station software. **That was incomplete.** The
heartbeat stalls scale monotonically with correction traffic:

| flight | RTCM | rx pkt/s | gaps > 2 s | gaps > 4 s | max gap |
| --- | --- | --- | --- | --- | --- |
| 11-36-20 | 0 | 1.1 | 3 | 1 | 4.46 s |
| 12-17-52 | 0 | 1.0 | **1** | 0 | 2.17 s |
| 13-23-25 | 639 | 10.6 | **16** | 2 | 4.95 s |
| 14-28-49 | 1338 | 14.2 | **20** | **4** | **5.10 s** |

Gaps over 2 s rose from 1–3 to 16–20 the moment RTCM injection began, and the flight carrying the
most correction traffic produced both the most stalls and the only gap to exceed the 5 s timeout.

`rxdp` = **0** on every flight — no packet was corrupted or lost in transit. Traffic is arriving
**late**, which is queuing in a congested link rather than RF failure. The FC↔radio serial
(`SERIAL2_BAUD` 115200) runs at roughly 63% utilisation outbound at 121 packets/s, so the binding
constraint is the **radio air rate**, which is typically 10–50 kbps on RC-integrated MAVLink links —
well below what 1.1 KB/s of RTCM can comfortably share.

### What was NOT the cause

**GNSS quality was irrelevant.** The aircraft held **RTK Fixed, 100% of the flight, `HAcc` 0.030 m,
33 satellites**. The GCS failsafe monitors the MAVLink heartbeat over the telemetry radio; the GNSS
receiver is a separate subsystem. Satellite count and RTK status can neither cause nor prevent a
GCS failsafe.

**RC control was never lost.** Control is on SBUS, a separate link, and remained available
throughout — which is why pilot intervention worked immediately.

### Configuration was more aggressive than firmware default

| param | firmware default | as flown |
| --- | --- | --- |
| `FS_GCS_ENABLE` | 0 (disabled) | **1 (RTL)** |
| `FS_OPTIONS` | **16** — `GCS_CONTINUE_IF_PILOT_CONTROL` | **0** — all options disabled |

`FS_OPTIONS` bit 4 is *"Continue if in pilot controlled modes on GCS failsafe."* ArduPilot enables
it by default. The aircraft was in **PosHold — a pilot-controlled mode** — so with the stock value
the failsafe would have taken no action.

**Adopted: `FS_OPTIONS = 16`.** This restores the firmware default. GCS failsafe remains active for
autonomous and BVLOS operation, but will not seize control while the pilot is actively flying.

**Adopted: `FS_GCS_TIMEOUT` 5 → 10 s.** Covers the measured 5.10 s worst case with margin.
Range is 2–120, default 5.

**`FS_THR_ENABLE = 1` deliberately unchanged** — true RC loss (transmitter off, out of range, power
failure) still triggers RTL in every mode. `FS_OPTIONS` bit 4 applies only to the GCS failsafe; the
RC-related bits are 0 and 2, and neither is set.

**Outstanding: reduce telemetry stream rates.** Outbound traffic of 121 packets/s competes with
RTCM for the same air link. `SR2_EXTRA1` (attitude) and `SR2_EXTRA3` / `SR2_POSITION` are the usual
candidates. **`SERIAL2_BAUD` should not be raised** — it is not the limiting element, and a
mismatch with the radio costs telemetry entirely.

### Hazard note

The launch point has a partial overhang. An uninterrupted RTL climb to `RTL_ALT_M = 30` from 1.8 m
would have passed through it. `RTL_ALT_M` is retained at 30 for general obstacle clearance, but
site-specific overhead clearance is now an operational check.

## Mechanical observation — motor asymmetry

| | M1 | M2 | M3 | M4 |
| --- | --- | --- | --- | --- |
| ESC temp °C | 33 | **39** | 30 | **25** |
| RPM median | 3008 | **3191** | 3041 | **2871** |

CW pair (M1+M2) 6199 RPM vs CCW pair (M3+M4) 5912 — the **CW pair works 4.9% harder**, indicating a
persistent yaw torque being opposed. `MOT_THST_HOVER` has drifted 0.162 → 0.169 → **0.180** across
the day.

Motor-mount screws on M2 were torqued a few flights earlier, which is one candidate; motor mount
squareness, arm alignment and prop condition are others. **Not established as causal.** ESC
telemetry reports temperature and RPM only — `MotTemp`, `Curr` and `Volt` all read zero, so these
are ESC temperatures, not windings.

## Decision

1. **`PSC_NE_POS_P` 1.0 → 0.5.** Halves outer-loop crossover to ~0.08 Hz, giving 2× separation from
   the velocity loop. Supported by the measured phase, by `tuning.md` §8.3, and by ArduPilot's own
   velocity-tracking criterion.
2. **`FS_OPTIONS` 0 → 16.** Restores the firmware default.
3. Motor asymmetry to be inspected physically before the next flight.

**Falsifiable prediction:** the 0.19–0.20 Hz peak weakens or disappears, desired-angle band
amplitude falls, and position rms drops **below** 0.13 m. The oscillation is currently the dominant
error term, so damping it should *reduce* total error. If rms rises instead, the premise is wrong.

## Confidence limits

- **"RTK did not improve hold": ~90%.** Two configurations, matched metric, effect far inside scatter.
- **"Cascade oscillation": ~85%.** Two windows agree; theory and measurement within 20%.
- **"`PSC_NE_POS_P` = 0.5 is sufficient": ~65%.** Mechanism sound, magnitude untested.
- **GCS failsafe root cause: ~95%.** `mgs` freeze and `rxdp` = 0 are unambiguous.
- **Motor asymmetry cause: unestablished.** Correlation with the M2 work is noted, not concluded.

## Health

Zero clips. RTK Fixed throughout. Flight 1 ended on GCS failsafe; flight 2 on battery failsafe at
21.26 V / 6067 mAh, handled normally.

## Next step

Refly the ≥ 60 s hands-off profile with `PSC_NE_POS_P = 0.5` as the single variable, then evaluate
`EK3_SRC2_VELXY = 5`.
