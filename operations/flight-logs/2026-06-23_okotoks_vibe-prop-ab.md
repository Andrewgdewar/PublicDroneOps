<!--
PRIVACY: The precise street address is intentionally kept out of the public-facing body
below (this file is on the public allowlist). Town-level location only is published.
Full site address is recorded in the PRIVATE block at the very bottom of this file —
if you do NOT want it public, keep that block commented or remove before publishing.
-->

# Flight Record — 2026-06-23 — Vibration Check & Prop A/B (1550 vs 1555)

**Status:** ☐ Draft ☐ In progress ☑ Filed
**Type:** Tuning / test session (NOT counted as TC-logged hours — telemetry log only, no FC `.bin`)

---

## 1. Operation
| Field | Value |
|---|---|
| Date | 2026-06-23 |
| Flights (local) | **1555:** ~12:06→~12:09 PM (~3 min airborne) · **1550:** ~1:19→~1:20 PM (~49 s airborne) |
| Location | Okotoks, AB (private residence — gated backyard) |
| Surface | Grass |
| Aircraft | Scout 15 (9IMOD 15" quad) |
| Pilot-in-command | Andrew Dewar |
| Flight profile | **~2 m hover (OGE)**, gated backyard, **kill switch at the ready** |
| Logging | None this session (SD card pending) — live telemetry only |

> Flight times are approximate (the GCS log's absolute clock was offset; times anchored to wall-clock
> with the log's reliable relative offsets). Durations and the ~1 h 13 m gap between flights are from the log.

## 2. Goal
Live-telemetry comparison of the two prop sets to inform the endurance/smoothness choice:
- **Vibration** (live `VIBRATION` in QGC) — confirm low vibe / decide if balancing needed.
- **Prop A/B (1550 vs 1555)** — hover **current, voltage/sag, throttle, vibe**.

## 3. Pre-flight
| Check | Result |
|---|---|
| Both prop sets balanced | ☐ |
| Same battery, same full charge for both runs | ☐ |
| Props/nuts tight | ☐ |
| Area clear, gated, on grass | ☑ |
| Kill switch (`RCx_OPTION=31`) verified | ☐ |
| GPS NARROW_INT / EKF green / no prearm errors | ☐ |

## 4. Method (each prop set)
Take off in Loiter → steady hands-off hover ~2 m, 60–90 s → after ~20 s settle, record
**steady-state** values (not the climb spike) → land → swap → repeat. Order: **1555 first**
(installed), then **1550**.

## 5. Results (from QGC telemetry CSV)

### 2026-07-13 follow-up — T-Motor MS1504 15×5.6 polymer+CF

MS1504 maiden (`2026-07-13 10-27-23.bin`) combined the prior props' strengths:

| Prop | Hover current | Hover power | Vibe X/Y/Z median |
|---|---:|---:|---:|
| 1555 CF | ~11.8 A | ~275 W | 6.7 / 11.4 / 6.3 |
| 1550 CF | ~10.5 A | ~244 W | 8.3 / 13.8 / 6.6 |
| **MS1504 polymer+CF** | **9.67 A** | **227 W** | **2.5 / 3.6 / 2.3** |

MS1504 also showed zero clipping, normal ESC temperature (40°C max), ~3031 RPM hover and
excellent ESC-notch attenuation. Cross-date power comparisons are indicative, but MS1504 is
the clear vibration winner and the lowest measured hover-power prop to date. Ideal hover
endurance is ~45 min from 7.2 Ah usable capacity (~42 min using a 160 Wh energy budget).

> **⚠️ Current-sensor calibration (updated 2026-07-01):** the FC current sensor was later found to **over-read by ~38 %** (calibrated by a full-charge → fly → recharge cycle; `BATT_AMP_PERVLT` corrected 40 → 28.9). **The hover-current and endurance figures below are the corrected true values.** Raw logged currents were 1555 ~16.3 A / 1550 ~14.5 A. The relative prop comparison (~11 %) is unaffected.

| Prop | Vibe X/Y/Z (steady) | New clips in hover | Hover current (A) | Voltage (sag) | Throttle % |
|---|---|---|---|---|---|
| **1555** (5.5 pitch) | **~6.7 / 11.4 / 6.3** | 0 (the 33 clips were from the start-of-flight anomaly) | **~11.8** | ~23.3 (minimal sag) | **~19%** |
| **1550** (5.0 pitch) | **~8.3 / 13.8 / 6.6** | 0 | **~10.5** | ~23.2 (minimal sag) | **~21%** |

> Both hovers at **~2 m, out of ground effect** (15" props ≈ 5 rotor diameters — IGE gone by ~2–3).
> Steady values from the settled Loiter hover segments (1555: telemetry samples 115–184; 1550: rows
> 28–55). Same single 9 Ah pack, similar charge (~23.2–23.3 V). Rangefinder not in telem stream;
> baro altitude unreliable (un-zeroed) — height confirmed visually at 2 m.

### Prop A/B verdict (both OGE @ 2 m — confounds removed)
- **1550 = better endurance:** ~10.5 A vs ~11.8 A — **~11% less hover current** (trustworthy now that height is matched).
- **1555 = smoother:** lower vibration on all three axes (esp. Y: 11.4 vs 13.8).
- Both: **0 clips, all axes < 30 m/s/s — acceptable.**

### Flight-time estimate (single 9 Ah pack, OGE hover @ 2 m)
- Battery: **9 Ah single pack (6S2P)** → usable ≈ **7.2 Ah**. *(Not flown in parallel.)*
- **1550 @ 10.5 A:** 7.2 ÷ 10.5 × 60 ≈ **~41 min**.
- **1555 @ 11.8 A:** 7.2 ÷ 11.8 × 60 ≈ **~37 min**.
- **Still-air hover only** — wind, maneuvering, payload will derate. But this **supersedes the earlier
  ~18–22 min projection** (which assumed 20–25 A): real OGE hover draw is lower than feared.
- **Note:** the build doc's ~42 min came from a motor **g/W** calc; measured hover draw is still higher
  than that peak-efficiency figure (off-peak operating point + ESC/wiring losses). **Plan on ~26–30 min
  still-air, derate for conditions.**

## 6. Observations / anomalies
- **ANOMALY — uncommanded violent climb at start of flight.** Took off in the wrong mode
  (manual/Stabilize-type, where the spring-centred throttle reads ~50% = leap up). Brief violent
  hop: peak current **124 A**, vibration spikes **~48 / 53 / 44 m/s/s (X/Y/Z)**, **33 accelerometer
  clips**, voltage sagged to **21.1 V**. Recovered by switching to **Loiter**; remainder of flight
  stable. No damage. **Corrective action: only ever arm/fly in Loiter or AltHold — never Stabilize
  (spring throttle makes it unsafe).**
- Altitude telemetry unreliable this session (read ~12 m while disarmed on the ground — EKF/baro
  height reference not zeroed). Did not affect the hover; noted for awareness.
- Steady Loiter hover was smooth and controllable on 1555.

## 7. Outcome
- **Vibration:** steady hover **~6–14 m/s/s — acceptable** (<30) on both props. The "33" seen in flight was the
  clip *counter* from the anomaly, not steady vibe. **No balancing strictly required** (a balance pass won't hurt).
- **Endurance (single 9 Ah pack, OGE @ 2 m):** **~41 min (1550) / ~37 min (1555)** still-air hover (corrected for the calibrated current sensor — see §5 note); derate for
  wind/maneuvering.
- **Prop choice — real tradeoff (both confirmed OGE):** **1550 = ~11% better endurance**, **1555 = smoother**.
  Leaning **1555** unless endurance is the priority; revisit with `.bin` logs once SD card is in.
- **Follow-ups:** SD card → notch filter (Mission 3) → PID/Autotune (Mission 4) → re-confirm prop choice with
  logged data; optional prop balance.

---

<!-- NOTE: This file is on the PUBLIC allowlist. The precise street address is deliberately
NOT stored anywhere in this file (HTML comments are still visible in the raw source on GitHub).
The full operating-site address, if needed for TC, lives only in PRIVATE records that are
NOT on the publish allowlist. -->

*Tuning/test session record. Not TC-logged hours (no dataflash log). Town-level location only is public.*
