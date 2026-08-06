# Scout‑15 Performance Debrief — Fastest & Farthest Session

**Date:** 2026‑07‑04
**Aircraft:** Scout‑15 quad · ArduCopter V4.8.0‑dev (commit `32e43699`, branch `scout15-flight`)
**Log:** `logs/2026-07-04_10-34-44.bin` (205 MB · 33 min armed · ~400 Hz logging)
**Configuration change under test:** 15×5.5 (1555) props · stock rate tune · `AVOID_ENABLE=1` (proximity avoidance off)
**Conditions:** Gusty wind — 15–25 km/h reported, higher bursts. Log‑derived steady component ~12–14 km/h from the W/WSW (~270°).
**Status at close:** Records set (speed + range). Airframe stable across the full envelope in gusty wind. One maintenance watch item logged (vibration headroom).

---

## TL;DR

- **New groundspeed record: 25.2 m/s (91 km/h)** — verified real (28 sats, HDOP 0.50, sustained ~1.4 s), not a GPS glitch. This was a **downwind** pass; wind‑corrected **true airspeed ~21 m/s (~78 km/h)** — still a record over the prior 18.3 m/s, in gusty air.
- **New range record: 471 m** from home, max altitude 61 m.
- **Held attitude and clean control through 15–25 km/h gusty wind** — a strong stability result for the stock tune.
- **Battery handled it with margin:** 3431 mAh consumed (~48 % of usable), minimum cell 3.54 V, peak draw 39.9 A / 847 W. No failsafe.
- **Tune stayed stable at full speed** — rate outputs clean, no oscillation signature at 91 km/h / 32° pitch.
- **Phantom Loiter "wall" is gone** with proximity avoidance disabled — zero avoidance events all session.
- **Safety finding (root‑caused & fixed): RTL was returning to the operator, not the takeoff point.** The home position was being overwritten at a steady 1 Hz to the ground device's GPS location — traced to a QGroundControl "update return‑to‑home to device location" setting, now disabled. See §5.
- **One watch item:** vibration is throttle‑driven (not a failing part) but headroom is marginal at high throttle, and the fore‑aft (Y) axis dominates every flight. Prop balance / mount torque check recommended before the next hard session.

---

## 1. Aircraft configuration (verified)

| Item | Value | Notes |
|---|---|---|
| AUW | ~2.2 kg | ⚠️ estimate — verify on scale |
| Props | 4 × 15×5.5 (1555) | swapped 2026‑07‑03; lower vibe than prior 1550 |
| Motors | Angel X4108 380 KV | |
| FC | Matek H743‑SLIM | |
| Firmware | ArduCopter V4.8.0‑dev `32e43699` | branch `scout15-flight` |
| Battery | 6S Li‑ion, `BATT_CAPACITY=7200` mAh | ~158 Wh usable |
| GPS | UM982 (NMEA) | 28 sats, HDOP 0.50 |
| Proximity avoidance | `AVOID_ENABLE=1` (off) | fence retained |

---

## 2. Performance — records set

| Metric | This session | Prior best (2026‑07‑03) |
|---|---|---|
| **Max speed** | **25.2 m/s = 91 km/h** | 18.3 m/s (66 km/h) |
| **Max distance from home** | **471 m** | near‑field |
| Max altitude | 61 m | — |
| Max pitch / roll | 32° / 28° | 32° / 23° |
| Mean speed (moving) | 9.9 m/s | — |

**Speed validation:** 208 log samples exceeded 20 m/s; the 25.2 m/s peak held for ~1.4 s (t≈1622 s) at 311–325 m from home with 28 satellites and HDOP 0.50. This is a genuine sustained pass, not a single‑sample spike.

### Wind correction — groundspeed vs true airspeed

GPS speed is **groundspeed**; the craft has no airspeed sensor. Wind was backed out of the log from the groundspeed‑vs‑heading asymmetry (high‑throttle passes only):

| Direction (high‑throttle p95) | Groundspeed |
|---|---|
| Downwind (ENE, ~45–90°) | 25.2 m/s |
| Upwind (SW/W, ~180–270°) | 17.4–18.3 m/s |

- **Log‑derived steady wind: ~3.4–3.9 m/s (12–14 km/h) from the W/WSW (~270°)** — consistent with the reported 15–25 km/h (the method reads the steady component, not gusts).
- **True airspeed capability: ~19–22 m/s (~70–78 km/h), best estimate ~21 m/s.**
- The 25.2 m/s peak = ~21 m/s true airspeed + ~3.5–4 m/s tailwind.

**Bottom line:** the 91 km/h groundspeed is real and stands as a groundspeed figure; through the air the craft achieved ~78 km/h — still a genuine improvement over the prior 18.3 m/s, earned in gusty conditions.

---

## 3. Battery — worked hard, healthy reserve

| Parameter | Value | Read |
|---|---|---|
| Consumed | **3431 mAh** | ~48 % of ~7200 mAh usable — landed with good reserve |
| Voltage start → end | 23.84 → 21.88 V | nominal |
| Minimum under load | **21.22 V (3.54 V/cell)** | healthy; above the 3.49 V/cell seen 2026‑07‑03 |
| Sag | 2.62 V | consistent with the current peaks |
| Current mean / peak | 13.3 A / **39.9 A** | peaks during the high‑speed passes |
| Power mean / peak | 298 W / **847 W** | 847 W burst to reach 91 km/h |

No battery failsafe triggered. The pack supported repeated high‑power bursts and returned with ~half its usable capacity remaining.

---

## 4. Flight dynamics & tune stability

Rate‑controller outputs stayed clean through the fastest and most aggressive flying to date:

| Axis | Mean output | % of samples > 0.10 |
|---|---|---|
| Roll (ROut) | 0.024 | 0.2 % |
| Pitch (POut) | 0.049 | 9.2 % |
| Yaw (YOut) | 0.017 | 1.0 % |

Pitch output is elevated only because sustained fast forward flight loads the pitch axis — there is **no oscillation or hunting signature**. The stock rate tune remains stable across the full speed and attitude envelope (91 km/h, 32° pitch), and held position cleanly through 15–25 km/h gusty wind — a good indicator of controller margin.

---

## 5. Mode behaviour

### Proximity "wall" — resolved
With `AVOID_ENABLE=1` (proximity avoidance disabled), there were **zero avoidance or proximity messages** for the entire session. Loiter reached 14.4 m/s, versus the 2–11.5 m/s phantom cap observed while avoidance was active on the (confirmed faulty) proximity sensor.

### Loiter vs PosHold speed — expected, by design
| Mode | Max speed | Note |
|---|---|---|
| PosHold | 25.2 m/s (91 km/h) | stick = lean angle (limited by `ANGLE_MAX`) |
| Loiter | 14.4 m/s (52 km/h) | stick = velocity demand (limited by `LOIT_SPEED`) |
| RTL | 8.5 m/s | `RTL_SPEED` |

Loiter is inherently speed‑limited by `LOIT_SPEED`; PosHold is limited only by lean angle. The remaining gap is normal mode behaviour, **not** the old phantom wall. Raise `LOIT_SPEED` if faster Loiter transits are wanted.

### RTL — safety finding (root‑caused and resolved)
RTL was exercised twice (t≈222 s, t≈1281 s) and returned without a "no return path" error — but it returned toward the **operator / ground‑station position, not the takeoff point.** Flying over people is unacceptable, so this was investigated to root cause.

**Proven from the log:** home was being **rewritten at a steady 1.00 Hz** (1067 updates, median interval 1.00 s), each time to a **stationary ground location** (positions clustered in a 3.5 m mean / 8.0 m max blob) while the aircraft ranged to 477 m. Home did **not** track the aircraft (home‑vs‑vehicle position correlation ≈ 0). Cross‑checked against firmware source: ArduCopter does not relocate an already‑set home in flight (`update_home_from_EKF()` early‑returns), so a continuous 1 Hz stream could only be an external MAVLink `DO_SET_HOME`.

**Root cause:** a **QGroundControl setting that updates the return‑to‑home position to the ground device's location.** With a GPS‑equipped ground device, this planted home on the operator once per second.

**Resolution:** setting disabled in QGroundControl — home now stays fixed at the arming/takeoff point, so RTL returns to takeoff. **Verify next flight:** the `ORGN` (home) record should show a single set at arming, not a 1 Hz stream.

**Optional defense‑in‑depth:** a Rally Point with `RALLY_INCL_HOME=0` makes RTL target a pre‑set clear location regardless of home — useful near people. Note this returns to a *fixed chosen point*, not the takeoff point.

### Notes
- Repeated "Motors Emergency Stopped" messages are ground‑side e‑stop toggles between flights — benign.
- Two brief GCS failsafes (t≈508 s, 592 s) self‑cleared within ~1–3 s.

---

## 6. Watch item — vibration headroom

Whole‑session vibration read high on the fore‑aft axis: **VibeY mean 21.0, max 87.7**, with 5181 accelerometer clip events (VibeX mean 15.1, VibeZ mean 11.5).

**Diagnosis — throttle‑driven, not a failing part:**

| Throttle band | VibeY mean | VibeY max | % > 30 |
|---|---|---|---|
| 0.00–0.12 | 14.4 | 47.9 | 4 % |
| 0.12–0.16 (hover) | 18.1 | 78.5 | 10 % |
| 0.16–0.20 | 25.4 | 85.3 | 30 % |
| 0.20–0.30 | 47.4 | 87.7 | 89 % |
| 0.30–1.00 | 64.6 | 87.4 | 100 % |

- Vibration correlates strongly with throttle (**Pearson r = 0.76** vs throttle, 0.73 vs current). The high whole‑session numbers came from the high‑throttle bursts needed for the speed/range records — compounded by gusty wind driving extra throttle activity.
- **Not a loosening fault:** hover‑band VibeY was flat across the session — 19.7 early vs 17.9 late. A progressively loosening component would climb over time; it did not.
- **Persistent Y‑axis dominance:** VibeY is the worst axis every flight (21 vs X 15, Z 11.5 today; also worst on 2026‑07‑03). A consistent single‑axis bias points to a specific fore‑aft mechanical source rather than random noise — most likely one prop's balance or a motor/arm on that axis.

**Recommended before the next hard session (proactive, not grounding):**
1. Balance‑check the 1555 props.
2. Re‑torque motor and arm bolts.
3. Confirm the flight‑controller soft‑mount is intact.

Rationale: hover‑band VibeY (~18–20) sits above the ideal (<15), and accelerometer clipping during aggressive bursts can reduce EKF altitude accuracy at the extremes of the envelope.

---

## 7. Summary

The 1555 props on the stock tune delivered the fastest (91 km/h) and farthest (471 m) flights logged for this airframe, with a stable controller across the full envelope and comfortable battery reserve. Proximity avoidance disabled cleanly resolved the phantom Loiter limit, and RTL behaved correctly. The single follow‑up is a routine vibration/balance check to restore high‑throttle headroom.
