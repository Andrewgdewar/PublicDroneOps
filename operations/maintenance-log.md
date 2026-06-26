# Scout 15 — Maintenance & Modification Log

> **Purpose:** Standing airframe maintenance and modification record for Transport Canada
> traceability (CARs Part IX — produce records on demand; retain for the life of the aircraft).
> One row per maintenance action or modification; significant events get a detailed entry below.
> This is the airworthiness paper trail behind the Standard 922 declaration.

**Aircraft:** Scout 15 (9IMOD 15" tube-frame quad)
**RPAS registration #:** _TBD_
**Firmware:** ArduCopter V4.8.0-dev (**MatekH743-bdshot**, git `32e43699d2`) — rebuilt to the bdshot variant for bidir DShot / ESC telemetry (see #3)
**Motors:** 4× Angel X4108 380 KV (89 g, M5 shaft, 45 mΩ) · **Props:** T-Motor 15" CF, direct M3 bolt-on to bell

---

## Log (chronological)

| # | Date | Component | Action / Event | Disposition | By |
|---|---|---|---|---|---|
| 1 | 2026-06-23 | Motor **rear-right** (Angel X4108) + M3 prop-mount bolt | Stripped M3 bolt → drilled head off → steel shavings entered bell/magnets & stator | **RESTORED** — props-off bench test: current matched all 4; magnet scoring cosmetic, no cracked magnets. Returned to service. | Andrew Dewar |
| 2 | 2026-06-26 | Motor **rear-left = output M1** (Angel X4108) | Higher-pitch whine + residue at bell screw + bell tight to remove | **KEPT (monitor)** — ESC-RPM bench test: RPM matched all 4 at 15%/25% (no drag); whine resolves >15% (low-RPM resonance); bell stiction axial, not rotational. Spare on hand. | Andrew Dewar |
| 3 | 2026-06-26 | Firmware (FC) | Rebuilt ArduCopter to **MatekH743-bdshot** (from plain MatekH743) to enable bidirectional DShot / ESC telemetry | **DONE** — `SERVO_BLH_BDMASK=15`, `SERVO_DSHOT_ESC=4` (EDT), `POLES=24`; per-motor RPM + temp now logging. Mods intact (git `32e43699d2`). | Andrew Dewar |

---

## Detailed entries

### #1 — 2026-06-23 — Motor damage from stripped prop-mount screw

**What happened**
- An **M3 socket-head (hex) prop-mount bolt** at one motor stripped — the hex socket cammed out completely smooth (no drive left).
- Removal escalated to the "nuclear" option: **drilled the head off** to release the part.
- During drilling, **steel shavings entered the motor** (bell magnets + stator windings); motor no longer spun smoothly.

**Recovery performed**
- Did **not** power the motor (short risk).
- Removed the bell, cleaned magnets + stator with **tape → poster putty → compressed air**.
- Result: spin is **much improved but still not as smooth** as the other three motors.
- **Visible scoring on the magnets** from the drill.

**Current state / risk**
- Motor is **compromised** (cosmetic-to-functional magnet damage; residual roughness).
- Risks if flown: slightly higher current/heat, more vibration, and an **unknown failure margin**
  (worst case: magnet crack/debond → in-flight motor lock).

**Disposition — RESTORED (updated 2026-06-26)**
- Props-off bench test (Mission Planner Motor Test): **current matched all 4 motors within noise**; no cracked/lifting magnets (scoring is cosmetic).
- **Returned to service** — performance indistinguishable from the other three. Monitor over early flights; spare on hand.

**Corrective / preventive action**
- Replace the stripped bolt; **swap all motor/prop M3 socket-head (hex) bolts to Torx or steel button-head** — the supplied hex screws are soft and cam out easily.
- When drilling/grinding near a motor in future, **remove the motor or fully mask the bell** to keep shavings out.

**References:** flight context in [operations/flight-logs/2026-06-23_okotoks_vibe-prop-ab.md](flight-logs/2026-06-23_okotoks_vibe-prop-ab.md)

---

### #2 — 2026-06-26 — Rear-left motor (M1) whine investigated → kept under monitoring

**What prompted it**
- Rear-left motor (output **M1**) had a slightly higher-pitch whine vs the other three; **residue** noted at the bell-retaining screw; the **bell would not release** even with firm pull (its identical-retention twin, rear-right, popped off with force) — initially suspected a failing bearing.

**Investigation — ESC-RPM bench test (props off, real battery, bidir DShot working)**
- Ran motors at **15% and 25%**, props off, with per-ESC RPM logging.
- **RPM matched across all 4 at the same output** (M1: 1233 @15% / 2216 @25%, vs pack 1225–1241 / 2208–2241) → **no mechanical drag.**
- Whine **resolves above 15%** → consistent with a **low-RPM tonal resonance, not a bearing fault.**
- Tight bell = **axial stiction** (tight shaft/bearing fit), distinct from rotational drag — squares with the matched RPM.
- ESC temperature telemetry **glitchy** (see open item); median temps normal (~25 °C), rear-left **not** elevated.

**Disposition — KEPT under monitoring**
- Performance indistinguishable from the other three; no measurable problem. Not pristine (whine/residue/stiction history), so **monitor**: listen for the whine worsening, check motor temp by touch after landing, **keep a spare X4108 on hand.**

**Open item:** EDT ESC-temperature telemetry **unreliable on 3 of 4 ESCs** (spurious spikes to 113–142 °C with sane ~25 °C medians). Investigate before relying on ESC temp for monitoring.

---

### #3 — 2026-06-26 — Firmware rebuilt to MatekH743-bdshot (enable ESC telemetry)

**What / why**
- Root cause of "ESC RPM always 0": firmware was the plain **MatekH743** build, which does **not** compile bidirectional DShot (`HAL_WITH_BIDIR_DSHOT` undefined → `SERVO_BLH_BDMASK` param absent).
- **Rebuilt** ArduCopter against the **`MatekH743-bdshot`** board target (Docker, ARM GCC 10.2.1); custom mods intact (CSPC lidar + GPS-for-yaw, git `32e43699d2`); reflashed via Mission Planner.

**Result**
- `SERVO_BLH_BDMASK=15` now available + set; `SERVO_DSHOT_ESC=4` (BlueJay+EDT); `SERVO_BLH_POLES=24`; `MOT_PWM_TYPE=5` (DShot300).
- **Per-motor RPM + temperature now logging** (`ESCn`); unlocks the ESC-RPM harmonic notch (`INS_HNTCH_MODE=3`) and EDT desync/demag event logging.
- Also resolved an SD-card **`ENOSPC`** logging failure (card reseat) — `.bin` logging now working.

---

*Maintenance & modification record. Add a row per action; keep for the life of the aircraft (CARs Part IX).*
