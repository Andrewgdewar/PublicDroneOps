# Scout-15 Roll P/I Sweep — Oscillation Boundary

**Aircraft:** Scout-15 quadcopter
**Flight:** 2026-08-04
**Flight log:** `2026-08-04 20-03-11.bin`
**Objective:** Locate the roll rate P/I oscillation boundary by in-flight sweep
**Outcome:** Boundary found at **0.170** (ring 5.0). Roll ceiling ~0.160
**Method:** `roll-pi-step.lua`, RC options 303 / 304, step 0.01

## Configuration proof

| Item | Value |
| --- | --- |
| Firmware | ArduCopter 4.8.0-dev, commit `32e43699` |
| `INS_HNTCH_OPTS` / `HMNCS` | 0 / 3 |
| `INS_GYRO_FILTER` | 18 |
| `ATC_RAT_RLL_D` | 0.0052, held |
| Pitch P/I / D | 0.150 / 0.0051, held |
| Boot roll P/I | 0.150 |

`ATC_RAT_RLL_P` and `_I` were written to the **same value** on every step, per the ArduPilot
manual tuning procedure — *"Each time the P term is changed set the I term equal to the P term."*

## Sweep sequence

```
t= 59.5s  0.150 -> 0.140
t= 63.7s  0.140 -> 0.150
t= 71.5s  0.150 -> 0.160
t= 86.0s  0.160 -> 0.170
t= 95.7s  0.170 -> 0.160
t= 96.1s  0.160 -> 0.150     returned to boot value before landing
```

## Result — `pidreview.py segments RLL`

| P/I | dur_s | ev | nerr | corr | rise ms | ring |
| --- | --- | --- | --- | --- | --- | --- |
| **0.150** | 32.4 | 9 | 0.54 | 0.91 | **67** | **1.0** |
| **0.160** | 13.7 | 6 | **0.29** | **0.97** | 89 | 1.5 |
| **0.170** | 8.8 | 3 | 0.47 | 0.92 | **150** | **5.0** |

## Decision

**0.170 is the oscillation onset.** Ring rose 5x and, decisively, rise time *increased* to 150 ms.
A healthy loop responds faster as P rises; rise time increasing while ringing grows is the
signature of the measurement being corrupted by oscillation.

Above 0.150 was closed pending a longer window. Roll ceiling recorded as ~0.160.

## Frequency-domain check

The oscillation at 0.170 peaks at **5.9 / 6.0 / 5.5 Hz** — the same frequency as the airframe mode
present at every other gain. Excess P excites the existing resonance rather than creating a new
loop instability. The frequency does not shift with gain.

## Confidence limits

The 0.170 reading rests on **3 events** and the 0.160 reading on 6. Magnitudes are therefore
indicative; the direction and the onset location are the reliable outputs.

## Health and telemetry

Zero clips. `Dmod` 1.00, `Dlim%` 0.0 throughout.

## Tooling

`roll-pi-step.lua` logs a custom `RPIS` stream carrying P, I and step direction, because a runtime
`Parameter:set()` emits no `PARM` record. `pidreview.py` was generalised from D-only to any
`(axis, term)` pair via `SCRIPTED_PARAM_STREAMS` so the sweep windows attribute correctly.

## Next step

Extend the sweep downward to establish whether the optimum lies below 0.150, and repeat 0.160 with
a longer window. See
[`../2026-08-04_roll-pitch-pi-sweep-review/README.md`](../2026-08-04_roll-pitch-pi-sweep-review/README.md).
