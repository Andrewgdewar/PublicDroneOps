# Scout-15 Pitch P/I Sweep — 0.150 to 0.190

**Aircraft:** Scout-15 quadcopter
**Flight:** 2026-08-05
**Flight log:** `2026-08-05 14-06-45.bin`
**Objective:** Locate the pitch rate P/I optimum and oscillation boundary
**Outcome:** **`ATC_RAT_PIT_P` / `_I` 0.150 → 0.160 adopted.** No oscillation boundary found to 0.190
**Method:** `pitch-pi-step.lua`, RC options 305 / 306, step 0.01

## Configuration proof

| Item | Value |
| --- | --- |
| Firmware | ArduCopter 4.8.0-dev, commit `32e43699` |
| `INS_HNTCH_OPTS` / `HMNCS` / `BW` / `MODE` | 0 / 3 / 15 / 3 (ESC telem) |
| `INS_GYRO_FILTER` | 18 |
| `ATC_RAT_PIT_D` | 0.0051, held |
| Roll P/I / D | **0.150 / 0.0052, held — verified single segment across all 126.1 s** |
| Boot pitch P/I | 0.150 |
| Armed | 127.1 s |

Roll was confirmed untouched by `pidreview.py segments RLL`, which returned one window for the
entire flight. The sweep is uncontaminated.

## Result — `pidreview.py segments PIT`

| P/I | dur_s | ev | nerr | ratio | corr | over% | rise ms | ring |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0.150 | 13.3 | 1 | 0.79 | 0.87 | 0.70 | 48.6 | 90 | 1.0 |
| **0.160** | 38.5 | 11 | **0.61** | 1.25 | **0.88** | **46.1** | 58 | 0.0 |
| 0.170 | 19.6 | 9 | 0.77 | 1.31 | 0.81 | 55.5 | **28** | 0.0 |
| 0.180 | 12.8 | 8 | 0.61 | 1.27 | 0.88 | 61.0 | 66 | 1.0 |
| 0.190 | 31.8 | 11 | 0.65 | 1.29 | 0.87 | 53.8 | 45 | 0.0 |

The 0.150 window returned a single event and is not usable as a baseline.

## Frequency-domain result — the deciding evidence

Pitch rate error spectrum per gain window:

| P/I | pitch err rms | Dominant peaks |
| --- | --- | --- |
| **0.160** | **6.16** | **2.2, 2.1, 2.5 Hz** |
| 0.170 | **10.58** | **6.3, 6.2**, 2.7 Hz |
| 0.180 | 9.73 | **6.2**, 2.2, **6.3** Hz |
| 0.190 | 7.53 | **6.1, 6.1, 6.1 Hz** |

At 0.160 the pitch error is dominated by ~2.2 Hz — pilot input and wind, i.e. external disturbance.
**At 0.170 and above the ~6.2 Hz airframe mode becomes the dominant error source**, and pitch error
rms rises 72% (6.16 → 10.58).

## Decision

`ATC_RAT_PIT_P` and `ATC_RAT_PIT_I` set to **0.160**.

Every metric agrees: lowest error rms, lowest overshoot, best `nerr` and `corr`, ring 0.0, and the
residual error stays low-frequency rather than resonant. The result reproduces the previous
flight's 0.160 reading independently (`nerr` 0.73 / `corr` 0.86 on 20-47-54 versus 0.61 / 0.88 here).

## Pilot report versus measurement

The pilot reported the aircraft *"felt better as I went"* through to 0.190. Rise time supports that
directly — **58 ms at 0.160 falling to 28 ms at 0.170** is a genuinely crisper response and is what
a pilot notices. Over the same interval, however, overshoot rose 46.1% → 55.5% and the 6.2 Hz mode
took over the error spectrum.

Crisper *response*, worse *tracking*. The subjective report was reading a real effect; the log shows
the cost that accompanied it.

## Tooling limitation found

**The `ring` metric alone was not sufficient on this airframe.** It returned 0.0 at 0.170 and 0.190
despite the 6.2 Hz mode clearly dominating the error spectrum at both. `ring` counts oscillation
cycles following a detected step event, so a continuously excited resonance between events does not
register.

Rate-error spectra must be checked alongside `ring` when evaluating a sweep on this aircraft.

## Confidence limits

Windows carry 8–11 events each except 0.150 (1 event, discarded). Error rms and spectral dominance
are consistent and large; `nerr` scatters between 0.61 and 0.77 with no clean trend above 0.160,
which is why the plateau rather than any single value above 0.160 is the reported finding.

## Comparison with prior prediction

A pre-flight estimate placed the pitch optimum near 0.172, extrapolated from the trend below 0.160
and from pitch showing more ring margin than roll. **That estimate was too high.** The improvement
plateaus at 0.160. Pitch does not exhibit the sharp ring cliff roll showed at 0.170; it degrades
through progressive resonance excitation instead, which the extrapolation did not anticipate.

## Health and telemetry

| Signal | Value |
| --- | --- |
| `VIBE` median X / Y / Z | 3.23 / 3.57 / 4.07 |
| `VIBE` p95 | 5.36 / 5.62 / 5.97 |
| `VIBE` max | 7.92 / 8.77 / 8.36 |
| Clips | 0 |
| `PM.Load` | 349 median, 369 max |
| `PM.NLon` | 1 max |
| `Dmod` / `Dlim%` | 1.00 / 0.0 |

## Next step

Roll remains at 0.150 with a 0.160 reading that rests on only 4 events. Repeat roll 0.160 with a
longer window before adopting it. Roll's ceiling is established at 0.170 (ring 5.0).
