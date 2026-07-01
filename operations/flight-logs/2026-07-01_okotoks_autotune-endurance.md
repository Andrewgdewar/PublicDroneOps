# Flight Record — 2026-07-01 — Scout 15

> **RPAS:** Scout 15 quadcopter — registration **C-2617791317**, serial **SCOUT15-001**
> **Location:** Okotoks, AB area (private test site)
> **Pilot:** Andrew Dewar — RPAS Pilot Certificate (Basic Operations)
> **Purpose:** AutoTune (roll/pitch) + endurance and handling assessment in elevated wind
> **Conditions:** Daytime VMC; **gusty / elevated wind** (wind picked up during Flight 2)
> **Firmware:** ArduCopter V4.8.0-dev (MatekH743-bdshot)

---

## 1. Summary

Two flights totalling **23.7 minutes** — the **longest and most demanding sorties to date** for this airframe, flown at altitude in gusty wind with AutoTune engaged. All systems were nominal thermally, electrically, and structurally. The aircraft held attitude well on **factory (untuned) PIDs** despite the wind. Post-flight, the motors and airframe were **confirmed cool to the touch**, consistent with the logged temperatures.

Two operational findings (neither an airframe-health issue):
1. A **GCS-telemetry failsafe** ended Flight 1 early via RTL.
2. **AutoTune could not converge the pitch Angle-P** term in the wind and saved no gains.

No mechanical, thermal, or power anomalies were observed.

---

## 2. Flights at a glance

| | Flight 1 | Flight 2 |
|---|---|---|
| Duration | 4.2 min | 19.5 min |
| Max altitude (AGL) | 13.2 m | 9.7 m |
| Max climb / descent | +2.0 / −1.3 m/s | +2.4 / −1.0 m/s |
| Max groundspeed | 5.3 m/s (19 km/h) | 3.7 m/s (13 km/h) |
| Hover throttle | 22 % | 22 % |
| Flight modes | Loiter, RTL, AutoTune | Loiter, AutoTune |
| Outcome | RTL (GCS failsafe) | Landed; AutoTune pitch not converged |

> The **22 % hover throttle** confirms a high thrust-to-weight ratio (over-powered for weight) — the primary reason the motors run cool and the aircraft feels responsive.

---

## 3. Systems assessment

### 3.1 Thermal — excellent (large margin)

| Sensor | Flight 1 max | Flight 2 max |
|---|---|---|
| Hottest ESC (M2, front-right) | 43 °C | 38 °C |
| Coolest ESC (M4, rear-right) | 29 °C | 24 °C |
| FC / IMU | 38 °C | 37 °C |

Peak ESC temperature **43 °C** — far below any thermal concern (BLHeli_S/Bluejay MCUs tolerate ~100 °C+). This **matches the pilot's post-flight observation** that the motors and airframe were cool. The high thrust-to-weight (22 % hover) keeps the motors loafing. No battery temperature sensor is fitted (logs 0 °C).

### 3.2 Motors / RPM balance — nominal, no fault signature

| Motor | Flight 1 median RPM | F1 spread | Flight 2 median RPM | F2 spread |
|---|---|---|---|---|
| M1 rear-left | 3225 | −3.8 % | 3500 | +6.4 % |
| M2 front-right | 3480 | +3.8 % | 3475 | +5.7 % |
| M3 front-left | 3116 | −7.1 % | 3102 | −5.7 % |
| M4 rear-right | 3506 | +4.6 % | 3008 | −8.5 % |

Hover RPM ~3000–3500. The spread widened modestly in Flight 2 (to −8.5 %), **consistent with differential thrust commanded to hold attitude against gusts** — normal closed-loop control behaviour, not a fault. Critically, **no single motor shows a consistent divergence** across both flights (the "low" motor identity shifts with wind direction), so there is no degradation signature. The **rear-left (M1) — the bearing-watch motor — ran normally**, even slightly high in Flight 2, showing **no drag**.

### 3.3 Vibration — clean, within limits

| Axis | Flight 1 median / max | Flight 2 median / max |
|---|---|---|
| VibeX | 8.3 / 35.7 | 14.2 / 39.1 |
| VibeY | 11.3 / 48.6 | 18.4 / 51.8 |
| VibeZ | 5.0 / 19.3 | 8.3 / 30.8 |
| Clipping | **0 / 0 / 0** | **0 / 0 / 0** |

**Zero accelerometer clipping on all axes, both flights.** Medians sit comfortably in the healthy band (<20). Transient peaks on the **Y-axis reached ~49–52** during aggressive AutoTune twitches and gusts, with X/Z lower. Y is the axis to keep monitoring, but with **no clipping and low medians** the harmonic-notch filter is clearly effective and structural vibration is well controlled. Higher Flight 2 medians reflect the AutoTune twitching plus wind.

### 3.4 Battery / power — healthy

| Metric | Flight 1 | Flight 2 |
|---|---|---|
| Voltage (start → min) | 24.14 → 22.96 V | 23.64 → 21.31 V |
| Sag (max) | 1.19 V | 2.34 V |
| Min cell (6S) | 3.83 V/cell | 3.55 V/cell |
| Current (median / peak) | 10.4 / 23.2 A | 10.5 / 22.5 A |
| Consumed (calibrated) | 708 mAh | 3423 mAh |

Hover current was a **consistent ~10.5 A** (corrected — see calibration note below), with peaks ~22–23 A during climbs and AutoTune twitches. Flight 2's larger sag (2.34 V) is expected — the pack was more depleted, raising internal resistance — yet the **minimum 21.31 V (3.55 V/cell) stayed healthy** and above the low-voltage failsafe (19.2 V). **No failsafe voltage trips.** Total consumed across both flights: **~4130 mAh over 23.7 min** (charger-verified).

> **Current-sensor calibration (from this flight):** the pack was fully charged before flight and recharged immediately after. The charger returned **~4130 mAh** against the FC's logged **5721 mAh** — the sensor was **over-reading ~38 %**. `BATT_AMP_PERVLT` was corrected **40 → 28.9**. All current and consumed figures in this record are the **corrected** values. This makes true hover current **~10.5 A** (not ~14.5 A) and still-air endurance **~40 min** (not ~30 min). Earlier records referencing ~14–16 A have been updated accordingly.

### 3.5 Navigation — excellent

**28 satellites, HDOP 0.50** on both flights (UM982 dual-antenna GNSS). A strong position solution that supported solid Loiter station-keeping in wind.

### 3.6 Attitude control / wind handling — good (on factory PIDs)

| Tracking error (desired − actual) | Flight 1 | Flight 2 |
|---|---|---|
| Roll RMS | 1.65° | 1.41° |
| Pitch RMS | 1.13° | 2.63° |
| Yaw RMS | 0.80° | 0.85° |
| Max transient error | 12.0° | 14.7° |

Attitude tracking held to **~1.4–2.6° RMS** despite gusty conditions and **untuned factory PIDs** — a good result. The higher Flight 2 pitch RMS (2.63°) reflects both the wind and the incomplete pitch tune. Peak transient errors (12–15°) coincide with gusts and AutoTune twitches. A **completed AutoTune is expected to tighten this further.**

---

## 4. Findings & actions

1. **GCS telemetry failsafe (Flight 1)** — error subsystem 6 / code 1: GCS heartbeat lost for >5 s → automatic RTL. This was **not** an RC-link or battery event (RC channels were valid throughout). **Action:** review GCS/telemetry link stability and `FS_GCS_TIMEOUT` before the next session.
2. **AutoTune incomplete** — roll and pitch were both attempted; pitch **Angle-P determination failed** in the wind (log: *"AutoTune: Angle P Gain Determination Failed"*). No gains were saved. **Action:** retune in calm conditions, **one axis at a time** (`AUTOTUNE_AXES=1`, roll first).
3. **No mechanical, thermal, or electrical anomalies.**

---

## 5. Conclusion

The Scout 15 completed its most demanding sorties to date — **23.7 minutes in gusty wind at altitude** — with **excellent thermal margins, clean vibration (zero clipping), healthy power delivery, strong GNSS, and good attitude tracking on factory gains**. The two open items (GCS link stability and AutoTune convergence) are **operational / configuration matters, not airframe health.** The aircraft is structurally and electrically sound and performed well under load.
