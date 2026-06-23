# Scout 15 — Maintenance & Modification Log

> **Purpose:** Standing airframe maintenance and modification record for Transport Canada
> traceability (CARs Part IX — produce records on demand; retain for the life of the aircraft).
> One row per maintenance action or modification; significant events get a detailed entry below.
> This is the airworthiness paper trail behind the Standard 922 declaration.

**Aircraft:** Scout 15 (9IMOD 15" tube-frame quad)
**RPAS registration #:** _TBD_
**Firmware:** ArduCopter V4.8.0-dev (MatekH743, git `32e43699d2`)
**Motors:** 4× Angel X4108 380 KV (89 g, M5 shaft, 45 mΩ) · **Props:** T-Motor 15" CF, direct M3 bolt-on to bell

---

## Log (chronological)

| # | Date | Component | Action / Event | Disposition | By |
|---|---|---|---|---|---|
| 1 | 2026-06-23 | Motor (Angel X4108 380 KV) + M3 prop-mount bolt | Stripped M3 socket-head bolt → drilled head off → steel shavings entered bell/magnets & stator | Cleaned (bell off, tape/putty/air). Improved but not as smooth as the other 3; **visible magnet scoring.** **Disposition PENDING** bench A/B test + magnet inspection | Andrew Dewar |

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

**Disposition — PENDING (decision criteria)**
- **Inspect** for any cracked / chipped / lifting magnet → if found, **replace the motor (non-negotiable).**
- **Bench A/B test** (props OFF, Mission Planner Motor Test) vs a known-good motor at the same throttle:
  compare **current draw, temperature, vibration, sound.**
- **Decision rule:** identical-to-good within noise **and** no cracked magnets → may return to service (logged + ESC-temp watched). Any measurable degradation or magnet crack → **replace; demote this motor to a bench/test spare.**
- **Leaning replacement** (~70%) for an operational/BVLOS aircraft — a motor is cheap vs. an in-flight failure.

**Corrective / preventive action**
- Replace the stripped bolt; **swap all motor/prop M3 socket-head (hex) bolts to Torx or steel button-head** — the supplied hex screws are soft and cam out easily.
- When drilling/grinding near a motor in future, **remove the motor or fully mask the bell** to keep shavings out.

**References:** flight context in [operations/flight-logs/2026-06-23_okotoks_vibe-prop-ab.md](flight-logs/2026-06-23_okotoks_vibe-prop-ab.md)

---

*Maintenance & modification record. Add a row per action; keep for the life of the aircraft (CARs Part IX).*
