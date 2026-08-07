# Scout-15 Gyro Filter Sweep — `INS_GYRO_FILTER` 18 → 27 Hz

**Aircraft:** Scout-15 quadcopter
**Flight event:** 2026-08-07
**Flight log:** `2026-08-07 09-53-15.bin`
**Objective:** Recover phase margin at the 5.87 Hz airframe mode by raising the gyro low-pass
toward its documented rule value, without touching any gain
**Outcome:** **Unambiguous success.** Roll `ring` 2.0 → **0.0**, mode energy **−3.6×**, `|D|`
**−3.5×**. Zero clips
**Decision:** **Adopt `INS_GYRO_FILTER = 27`**
**Parameter snapshot:** `certification/parameters/0807.param` (`STAT_BOOTCNT` 334)

## Summary

The aircraft had been flying at `INS_GYRO_FILTER = 18` for months against a rule value of
`max(20, 400/D)` = **26.7 Hz** for its 15" props. `INS_GYRO_FILTER` is a **2-pole Butterworth**, so
that 9 Hz shortfall was costing **9.4° of phase** at the airframe mode for no compensating benefit.

An in-flight sweep — `18 → 21 → 24 → 27 → 24 → 21 → 18` — improved every roll and pitch metric
monotonically. The predicted noise cost appeared exactly as forecast but was **59× smaller** than
the resonance it removed.

This is the direct inverse of the 2026-08-06 PID notch event
([record](../2026-08-06_pid-notch-divergence-event/README.md)), which *spent* 17.7° of phase and
diverged the aircraft. **Same currency, opposite direction, opposite result.**

## Configuration proof — read from the log

| Item | Value |
| --- | --- |
| Firmware | ArduCopter 4.8.0-dev, commit `32e43699` |
| `INS_GYRO_FILTER` | **18 at boot**, swept in flight by script |
| Rate PIDs | **unchanged**: RLL 0.160/0.160/0.0052, PIT 0.160/0.160/0.0051 |
| `ATC_ANG_RLL_P` / `PIT_P` | 6.0 / 4.5 — unchanged |
| `ATC_RAT_*_FLTT` / `FLTD` | 10 / 10 — unchanged |
| `ATC_RAT_*_SMAX` | 12 |
| `INS_HNTCH_OPTS` / `HMNCS` / `BW` / `MODE` | 0 / 3 / 15 / 3 |
| `INS_RAW_LOG_OPT` | 9 — raw gyro, pre and post filter |
| `PSC_D_ACC_P` / `_I` | 0.017 / 0.034 |
| `FILT1_TYPE` | 0 — PID notch off |
| Armed | 156.7 s, one span, Loiter |

Script: `certification/scripts/gyro-filter-step.lua`, RC9 (301) +3 Hz, RC10 (302) −3 Hz,
clamped 18–27. The ladder is confirmed by the script's own GCS output, including the low clamp:

```
GYRF: at limit 18 Hz          <- clamp held
GYRF: 18 -> 21    GYRF: 21 -> 18    GYRF: 18 -> 21
GYRF: 21 -> 24    GYRF: 24 -> 27
GYRF: 27 -> 24    GYRF: 24 -> 21    GYRF: 21 -> 18
```

**Roll was exercised on the way up, pitch on the way down**, so the two axes are semi-independent
passes that happen to share the 27 Hz turnaround. Event counts confirm it:

| window | 18 | 21 | 24 | **27** | 24 | 18 |
| --- | --- | --- | --- | --- | --- | --- |
| roll events | 0 | 8 | 9 | **6** | 0 | 3 |
| pitch events | 2 | 0 | 0 | **8** | 6 | 6 |

## Results

All values normalised by `lean`, because manoeuvre intensity varied **1.2–10.2°** across windows —
the trap that invalidated three earlier sorties.

### Roll — ascending pass, the axis carrying the structural mode

| `gyroF` | ev | `ring` | `mode` | `lean` | **`mode/lean`** | `\|D\|` |
| --- | --- | --- | --- | --- | --- | --- |
| 18 | 3 | 2.0 | 1.49 | 4.8 | **0.310** | 0.00733 |
| 21 | 8 | 1.5 | 0.85 | 6.6 | **0.129** | 0.00386 |
| 24 | 9 | 1.0 | 0.84 | 9.8 | **0.086** | 0.00348 |
| **27** | 6 | **0.0** | 0.32 | 8.1 | **0.040** | **0.00209** |

`ring` falls monotonically to **zero**. Normalised mode amplitude falls **7.75×**.

### Pitch — descending pass, independent confirmation

| `gyroF` | ev | `ring` | **`mode/lean`** | rise ms |
| --- | --- | --- | --- | --- |
| 18 | 6 | 0.5 | **0.092** | 146 |
| 24 | 6 | 0.0 | **0.054** | 32 |
| **27** | 8 | 0.5 | **0.035** | **44** |

Same direction, smaller magnitude — consistent with roll being the axis that carries the mode
(pre-filter, roll's dominant peak is the 5.9 Hz mode while pitch and yaw are dominated by 106 Hz
motor noise).

### The benefit/cost trade, measured directly from raw gyro

Post-filter roll gyro, 1927 Hz sampling:

| `gyroF` | **5–7 Hz mode** | **30–200 Hz noise** | 200+ Hz |
| --- | --- | --- | --- |
| 18 | **0.03333** | 0.00036 | 0.00023 |
| 21 | 0.02377 | 0.00050 | 0.00045 |
| 24 | 0.01844 | 0.00073 | 0.00072 |
| **27** | **0.00919** | **0.00077** | 0.00064 |

**Mode −3.6×. Noise +2.1×.** Both predicted, both real. But in absolute terms the mode fell by
**0.0241** while noise rose by **0.0004** — a trade of roughly **59 : 1**.

### Two results that were counter-intuitive

**`|D|` fell 3.5×, it did not rise.** The expectation with less filtering is more high-frequency
noise into the D term. Instead `|D|` went 0.00733 → 0.00209. **D was being driven by the
resonance, not by noise** — remove the phase lag, the resonance shrinks, and there is less for D to
differentiate. On an airframe with a structural mode inside the control band, the mode dominates
the D term and noise is a rounding error.

**At 18 Hz the mode barely needs provoking.** In the 18 Hz window where roll was *not* being
commanded (pitch pass, 0 roll events), roll mode energy measured **0.03333** — higher than at
21 Hz *while roll was being actively driven* (0.02377). Excess phase lag does not merely amplify
commanded motion; it lets the airframe ring on its own.

## Decision

**Adopt `INS_GYRO_FILTER = 27`.** Now saved in `0807.param`.

**Do not go higher**, on three grounds:

1. 27 is the rule value. `400/15 = 26.67`, and `INS_GYRO_FILTER` is `AP_Int16` so only whole Hz are
   settable.
2. Diminishing returns. 18 → 27 buys 9.4°; 27 → 40 buys only 5.9° more.
3. The gyro LPF stops helping the harmonic notch. At 27 Hz it still gives −14.0 dB at the 60 Hz
   motor fundamental; at 40 Hz only −7.8 dB, handing 6 dB back to a notch that must then carry it
   alone.

`ring` had also reached its 0.0 floor, so the metric could not resolve further improvement.

## Confidence limits

- **The direction and mechanism are solid, ~95%.** Two semi-independent axis passes agree, the
  effect is monotonic across four settings, and the spectral measurement is taken directly from raw
  gyro rather than inferred.
- **Strong cross-flight check:** the return-to-18 window measured `mode` **1.49** against **1.48**
  on the previous flight — **0.7% agreement** on an independent sortie. The metric is repeatable.
- **Within-setting scatter is still large.** The two 24 Hz windows read `mode/lean` 0.086
  (ascending, roll-excited) and 0.030 (descending, pitch-excited) — a 2.9× spread. They are not
  like-for-like, but it bounds how finely this can be resolved: **~3 Hz steps are near the limit,
  1 Hz would be invisible.**
- **No clean 18 Hz roll window with ≥6 events exists on this flight.** The opening 18 Hz window had
  **zero** roll events (hover, lean 1.2°) and is unusable. The 18 Hz roll figures quoted come from
  the return window with only 3 events, cross-checked against the previous flight's 10-event
  baseline.
- **Nothing above 27 was tested.** The mode was still falling at the top of the ladder. This
  validates the rule value as *at least* right; it does not establish it as an optimum.

## Health and telemetry

| Metric | Value | Limit | Status |
| --- | --- | --- | --- |
| Clip count | **0** | 0 | pass |
| VIBE median | 2.35 / 2.51 / 3.00 | < 15 | pass |
| VIBE max | 9.45 / 11.40 / 8.53 | < 30 | pass |
| `PM.Load` max | 388 | — | nominal |
| `PM.NLon` max | 2 | — | nominal |
| `Dmod` | 1.000, `Dlim%` 0.0 | — | `SMAX` never engaged |

VIBE is higher than the 08-47-02 flight (median 1.58) but so was the flying — 27% of this flight
was above 10° lean against 7% before. No abort criterion was approached.

## Pilot observation — altitude dip on braking (indicative, not established)

Pilot reported that during lateral reversals the aircraft sags noticeably on the braking phase,
estimated at *"40 cm one way, 20 cm the other"*, felt across the last couple of flights.
**Recorded as a pilot impression, explicitly not a measured conclusion.**

What the data shows:

| flight | `PSC_D_ACC` | worst sag | rms err at lean>10° | % time lean>10° |
| --- | --- | --- | --- | --- |
| 2026-08-04 19-57-57 | 0.05 / 0.10 (old) | **32 cm** | 0.126 | 9% |
| 2026-08-07 08-47-02 | 0.017 / 0.034 | 21 cm | 0.043 | 7% |
| 2026-08-07 09-53-15 | 0.017 / 0.034 | **32 cm** | 0.102 | **27%** |

- **Magnitude is consistent with the pilot estimate** — measured worst sag 32 cm against a felt
  ~40 cm.
- **It is not a regression from the `PSC_D_ACC` correction.** The pre-change flight sagged the same
  32 cm and was slightly *worse* during high lean. That change is exonerated.
- **The likeliest reason it is newly noticeable is exposure.** This flight spent **27%** of its time
  above 10° lean, three to four times the previous flights.
- **The reported left/right asymmetry did not reproduce.** Measured minima were −7 cm rolling left
  and −11 cm rolling right — the opposite ordering, much smaller, and on only ~95 samples per side.
  Not established either way.

Ruled out as causes:

- **Angle boost is working.** `CTUN.ABst` peaks at 0.0154 against a theoretical requirement of
  `(1/cos 23.5° − 1) × 0.162` = **0.0146**. Lean compensation is being applied correctly.
- **Not throttle authority.** `ThO` peaks at 0.223 of 1.0; during lean > 15° it reaches only 0.188
  against a 0.162 hover. There is enormous headroom.

That leaves **transient response lag in the vertical loop** as the working hypothesis — the
controller reacts to altitude and acceleration error after the fact, so a rapid attitude change
produces a brief sag before recovery. No corrective action is proposed on this evidence.

## Next step

The gain-margin problem that motivated all of this is materially improved without a single gain
having changed. Before scaling roll P/I/D down, **re-fly a normal sortie at 27 Hz** and confirm the
new baseline holds outside a sweep. If roll `ring` stays at or near 0.0, the gain-reduction sortie
may no longer be necessary at all.

Open, unquantified: whether `ATC_RAT_*_FLTT/FLTD` should follow `INS_GYRO_FILTER / 2` to 13.5 Hz
now that the gyro filter has moved. It is left at 10 deliberately — raising it feeds *more* D to the
5.87 Hz mode, which every measurement to date says makes ring worse.
