# Scout‑15 Tuning Review — QuikTune & AutoTune Episode

**Dates:** 2026‑07‑02 (QuikTune) and 2026‑07‑03 (AutoTune)
**Aircraft:** Scout‑15 quad · ArduCopter V4.8.0‑dev (commit `32e43699`, branch `scout15-flight`)
**Author of record:** flight/tuning log
**Status at close:** Reverted to stock tune (verified) + `ATC_THR_MIX_MAX=0.9` applied. Craft flies well ("effortless, smooth") on stock.

---

## TL;DR

- **Stock rate tune is the best tune the craft has had.** Both auto‑tuners *degraded* it, by different mechanisms.
- **QuikTune (2026‑07‑02):** pegged `YAW_P` to its ceiling unvalidated and left an outer‑loop (LOITER) seesaw. Reverted.
- **AutoTune pitch (2026‑07‑03):** "Success" but gutted **rate P 0.135 → 0.038 (−72%)** while only cutting **angle P −31%**, creating an unbalanced cascade that **hunts at ~0.8 Hz** (touchy / seesaw). Caused by the tune never getting clean twitches — constant "pilot overrides" from wind drift in AltHold. Reverted.
- **Descent instability = aerodynamic (settling‑with‑power / VRS‑adjacent)**, not tune. Low disk loading makes it easy to trigger at gentle descent rates. Mitigated operationally + `THR_MIX_MAX=0.9`.
- **Path forward:** keep stock (+ input shaping) or manual‑tune; treat AutoTune as optional polish only, PosHold‑entry, calm air.

---

## 1. Aircraft configuration (verified)

| Item | Value | Notes |
|---|---|---|
| AUW | ~2.2 kg | ⚠️ estimate — verify on scale |
| Props | 4 × 15″ | |
| Motors | Angel X4108 380 KV | |
| FC | Matek H743‑SLIM | no Ethernet PHY |
| Firmware | ArduCopter V4.8.0‑dev `32e43699` | branch `scout15-flight` |
| Battery | 6S Li‑ion, `BATT_CAPACITY=7200` mAh | ~158 Wh usable |
| Disk loading | ~4.8 kg/m² (~47 N/m²) | **low** — see §6 |
| Thrust‑to‑weight | ~6 : 1 | hovers at ~17 % throttle |
| GPS | UM982 (NMEA) | 28 sats, HDOP 0.50 |
| ESC / notch | BLHeli32 bdshot; dynamic harmonic notch, ESC‑telem mode | **tracking 54–70 Hz** (verified) |

**Classification:** ultralight / low‑disk‑loading / over‑propped / high‑authority — *not* a heavy‑lift craft. This drives both the tuning behaviour (§4–5) and the descent behaviour (§6).

---

## 2. Baseline (stock) tune — the reference

| Param | Roll | Pitch | Yaw |
|---|---|---|---|
| `ATC_RAT_*_P` | 0.135 | 0.135 | 0.18 |
| `ATC_RAT_*_I` | 0.135 | 0.135 | 0.018 |
| `ATC_RAT_*_D` | 0.0036 | 0.0036 | 0 |
| `ATC_ANG_*_P` | 4.5 | 4.5 | 4.5 |
| `ATC_RAT_*_FLTT/FLTD` | 10 / 10 | 10 / 10 | 20 / 20 |

Feel shaping: `MOT_THST_EXPO=0.7`, `ATC_INPUT_TC=0.25`, `MOT_SPIN_MIN=0.13`.
PID voltage scaling on: `MOT_BAT_VOLT_MAX/MIN = 25.2 / 19.2`.

**Subjective:** takeoff and hover "effortless, smooth." This is the tune we keep.

### Hover performance (measured, 2026‑07‑02 log)

| | Current | Power | Throttle | Voltage |
|---|---|---|---|---|
| Hover | ~10 A | **~230 W** | ~17 % | 22.8 V |

Power held ~230 W across the pack (amps rise late only from voltage sag). The `MOT_THST_EXPO=0.7` change improved **throttle linearity / feel**, not power — hover power is set by weight, as expected.

---

## 3. Episode 1 — QuikTune (2026‑07‑02)

**Log:** `logs/2026-07-02_quicktune_19-22-22.bin` · **Result file:** `certification/parameters/quicktune/afterQuicktune.param`

**What happened:** QuikTune run three times (one reverted mid‑run). Final saved gains:

| | Roll | Pitch | Yaw |
|---|---|---|---|
| Rate P | 0.121 | 0.160 | **0.50** |
| Rate D | 0.0086 | 0.0075 | 0.010 |
| FLTT/FLTD | 9 | 9 | 9 |

**Findings (from log):**
- **Attitude tracking stayed good (<1° error).** The rate loop was fine.
- **`YAW_P` ramped to the `QUIK_YAW_P_MAX=0.5` ceiling without ever detecting oscillation** (slew‑rate metric stayed 0.03–0.11 through the whole yaw sweep) — i.e. **0.5 is an unvalidated guess**, and the likely cause of "turns not smooth."
- The "seesaw / neutered / self‑stops" feel was the **LOITER position controller** (outer loop) braking against held stick — `DesPitch` reversed ~0.6×/s under a held‑forward stick. Not the rate tune.
- Running QuikTune repeatedly compounded the degradation.

**Outcome:** reverted to stock.

---

## 4. Episode 2 — AutoTune pitch (2026‑07‑03)

**Log:** `logs/2026-07-03_autotune_10-16-18.bin` · entered from **AltHold**, `AUTOTUNE_AXES=2` (pitch only), two sessions.

**Result (reported "Success"):**

| Param | Stock | AutoTune | Change |
|---|---|---|---|
| `ATC_RAT_PIT_P` | 0.135 | **0.038** | **−72 %** |
| `ATC_RAT_PIT_I` | 0.135 | 0.038 | −72 % |
| `ATC_RAT_PIT_D` | 0.0036 | 0.0040 | ≈ stock |
| `ATC_ANG_PIT_P` | 4.5 | 3.122 | −31 % |
| Max pitch accel | — | ~362 °/s² | reduced |

**Why the tune is bad (measured):**
- AltHold A/B — pitch **attitude error 7.35°** on the AutoTune gains vs **1.17°** on stock; a sustained **~0.8 Hz oscillation**.
- It is **not "soft."** AutoTune cut **rate P far more than angle P** (angle‑to‑rate balance skewed from ~33:1 → ~82:1). An inner loop too weak for its outer loop **limit‑cycles** — the ~0.8 Hz seesaw — and the **15″ prop lag** (slow spin‑up = extra phase lag) makes it hunt even more readily. "Touchy" = marginally stable, so any stick input/gust excites the oscillation.

**Root cause of the failed tune (from AutoTune messages):**
- Stuck at **"Pitch Rate D Up 0%" for ~2 minutes** with near‑continuous **"AutoTune: pilot overrides active."**
- The ultralight, high‑authority craft **drifts fast in wind**, and AltHold has **no position hold** → constant repositioning → every stick touch pauses/corrupts the twitch test → the algorithm converged to garbage and still reported "Success."

**Outcome:** reverted to stock — verified `ATC_RAT_PIT_P=0.135`, `ATC_ANG_PIT_P=4.5` in `current.param`.

---

## 5. Cross‑episode conclusion

The through‑line ("every tune makes it worse") is real and has **two different root causes**:
- **QuikTune:** unvalidated yaw‑P peg + outer‑loop (position‑controller) seesaw.
- **AutoTune:** gutted rate P → unbalanced‑cascade hunting.

Both moved the craft off a **stock tune that is genuinely well‑matched** to a low‑disk‑loading, big‑prop, high‑authority airframe. This is consistent with:
- **ArduPilot's documented order** — manual tune / QuikTune are done *before* AutoTune; AutoTune is the *optional last polish*, not the primary tool.
- **Community experience** on large‑prop / ultralight builds — AutoTune frequently "softens everything," and builders revert to manual.

---

## 6. Descent instability (priority item)

**Symptom:** significant instability descending from altitude (~30 m).

**Findings:**
- **Worst instability was on the AutoTune pitch gains** (7° / 0.8 Hz oscillation on descent) — removed by reverting to stock.
- **On stock gains, descents are mild:** pitch attitude error ~1.0–1.65° with occasional ±8–9° nudges, **but VibeZ rises from ~7–9 to 16–20** — the signature of descending into disturbed air.

**Root cause — aerodynamic (settling‑with‑power / VRS‑adjacent), not tune.** First‑order estimate:
- Hover induced velocity `vh = √(DL / 2ρ) ≈ √(47 / 2.45) ≈ 4.4 m/s`.
- Observed descent rates 1.5–2.5 m/s ⇒ `Vz/vh ≈ 0.34–0.57` — the **incipient‑settling** band.
- **Low disk loading lowers `vh`, so settling begins at *gentle* descent rates** — an ultralight over‑propped craft is especially prone to this. *(First‑order calc; treat as indicative.)*

**Mitigations:**
- **Operational (primary):** descend **with forward/lateral airspeed** (fly out of the wake), or keep vertical descent **very gentle (<1 m/s)**. **Avoid moderate (1.5–3 m/s) straight‑down descent** — more sink rate drives *deeper* into the ring, it does not escape it.
- **Tune (secondary, applied):** `ATC_THR_MIX_MAX = 0.5 → 0.9` — prioritise attitude over throttle during descent disturbances.

---

## 7. Verified‑healthy infrastructure (not implicated)

| Check | Result |
|---|---|
| Harmonic notch | tracking motor fundamental (FTNS 54–70 Hz ≈ ESC RPM 3333/60) ✅ |
| ESC RPM telemetry | live, sane (median 3333 RPM) ✅ |
| Vibration | 12–20 (X/Y/Z), <30 limit, 0 clipping in cruise ✅ |
| GPS / EKF | 28 sats, HDOP 0.50 ✅ |
| Battery cal | `BATT_AMP_PERVLT=28.9` verified ✅ |

---

## 8. Current configuration & flight‑mode plan

- **Tune:** stock (reverted + verified). `MOT_THST_HOVER` now learned ≈ 0.171.
- **Applied change:** `ATC_THR_MIX_MAX=0.9` (staged in `current.param`; load to FC to activate).
- **Flight modes:**
  - **PosHold = default** for manual flying (direct stick feel, holds position when centred, none of LOITER's "fights‑me" braking) **and** for AutoTune‑in‑wind entry (weak position hold prevents drift/overrides).
  - **AltHold on a switch** = GPS‑independent fallback + cleanest tune A/B.
- **A/B switch:** `RCx_OPTION=180` (AUTOTUNE_TEST_GAINS) — LOW = original, HIGH = new tune, saves on disarm by switch position.

---

## 9. Recommended next steps

| Option | What | When |
|---|---|---|
| **A (recommended)** | Keep stock; refine *feel* only via `ATC_INPUT_TC` / expo | Now — it flies well |
| **B** | Manual tune roll+pitch: nudge `RAT_*_P` up from 0.135 in ~10 % steps, watch log for oscillation onset, back off 20–30 %. Consider the **Methodic Configurator** | If chasing better disturbance rejection |
| **C** | QuikTune **roll+pitch only** (`QUIK_AXES=3`), **PosHold entry, calm air** | Middle ground; avoids the yaw peg |

**If AutoTune is attempted again:** PosHold entry (never AltHold at this site), calm air, hands‑off, `AXES` one axis at a time, and **watch the progress %** actually climb (0 % = not getting clean data → abort).

---

## 10. File & log record

**Params** (`certification/parameters/quicktune/` unless noted):
- `current.param` — stock baseline **+ `THR_MIX_MAX=0.9`** (current state of record)
- `afterQuicktune.param` — 2026‑07‑02 QuikTune result (bad)
- `afterAddingQuicktune.param` — pre‑QuikTune reset baseline
- `../bad_tuning_params.param.disabled` — earlier bad set (retained, disabled)

**Logs** (`logs/`):
- `2026-07-02_quicktune_19-22-22.bin` — QuikTune episode
- `2026-07-03_autotune_10-16-18.bin` — AutoTune pitch episode

> Note: FC parameter download overwrites `current.param`. `.param` files must keep **CRLF** line endings (bare‑LF crashes the Mission Planner loader). Verify after any FC save.
