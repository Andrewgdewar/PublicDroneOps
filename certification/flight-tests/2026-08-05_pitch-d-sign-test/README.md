# Pitch D Sign Test — D Feeds the 6.2 Hz Mode

| | |
|---|---|
| **Aircraft** | Scout-15 · Matek H743-SLIM · ArduCopter 4.8.0-dev `32e43699` |
| **Date** | 2026-08-05 |
| **Log** | `2026-08-05 15-18-14.bin` |
| **Objective** | Determine the **sign** of `ATC_RAT_PIT_D`'s effect on the ~6.2 Hz airframe mode at pitch P/I 0.170 |
| **Outcome** | **Direction answered — more D made the mode worse.** Magnitude of that claim was later revised down; see *Correction*. Pilot aborted the sweep on feel, and the direction of the data agreed |

> **CORRECTION, same day.** This record originally headlined a **+106%** rise in the 6.2 Hz mode.
> That figure did not account for manoeuvre intensity, which was **32% higher** in the second
> window. Normalised, the rise is **+49%**, and the attitude-error metric moved the *other* way
> (`attRMS` 1.66 -> 1.61). The direction of the result stands; the magnitude does not, and the
> attitude metric does not support it at all. `pidreview.py segments` now prints `lean` and
> `attRMS` per window so this cannot recur silently. See *Correction* below.

## Why this test existed

Flight `2026-08-05 14-06-45` adopted pitch P/I **0.160** because above it the ~6.2 Hz airframe mode
became the dominant error source — err rms rose 6.16 → 10.58 going 0.160 → 0.170. But that whole
sweep was flown at **D = 0.0051 only**: a one-dimensional slice, not an optimum. If D were simply
too small to damp the mode, pitch 0.170+ might have become usable with more D.

The prior reasoning for testing D upward was: the D/P ratio **fell** 0.177 → 0.123 exactly where
the mode appeared, `Dmod` 1.000 / `Dlim%` 0.0 showed D had full headroom, and the modelled phase
budget at 6.2 Hz was +39° — still leading, so still damping *in principle*.

**That phase model was wrong, or incomplete.** See Decision.

## Configuration proof

Read from the log, not assumed.

| Parameter | Value |
|---|---|
| `ATC_RAT_PIT_P` / `_I` | 0.160 (boot) |
| `ATC_RAT_PIT_D` | 0.0051 (boot) |
| `ATC_RAT_RLL_P` / `_I` | 0.150 |
| `ATC_RAT_RLL_D` | 0.0052 |
| `INS_GYRO_FILTER` | 18 |
| `INS_HNTCH_OPTS` | 0 |
| `INS_HNTCH_HMNCS` | 3 |
| `INS_RAW_LOG_OPT` | 0 |

Stepper activity confirmed in `MSG`:

```text
 72.9  PPIS: Pitch P/I 0.160 -> 0.170
129.0  PTDS: ready D=0.0051 step=0.0004
156.1  PTDS: Pitch D 0.0051 -> 0.0056
184.1  PTDS: Pitch D 0.0056 -> 0.0052
```

Roll was not touched. `segments RLL` returns two windows both at 0.150 / 0.0052.

## Result — the measurement that answers the question

Pitch **rate**-error spectrum per window. This is the criterion; `ring` was known in advance to be
blind to a continuously excited resonance.

| Window | err rms | **6.2 Hz amp** | 2.2 Hz amp | top peaks |
|---|---|---|---|---|
| P/I 0.160, D 0.0051 | 10.47 | 0.834 | 0.811 | 6.0, 5.3, 6.0 Hz |
| P/I 0.170, D 0.0051 | 8.37 | **0.575** | 0.519 | 6.3, 2.4, 5.9 Hz |
| P/I 0.170, **D 0.0056** | 8.87 | **1.185** | 0.597 | 6.1, 5.8, 5.7 Hz |

**The 6.2 Hz component more than doubled: 0.575 → 1.185, +106%.** Overall err rms also rose,
8.37 → 8.87.

> **This +106% is superseded.** Manoeuvre intensity was not equal between the two windows —
> mean demanded lean **6.5° vs 8.6°**, 32% higher in the D 0.0056 window. Normalised the rise is
> **+49%**. See *Correction*.

D-term contribution over the same windows:

| Window | \|P out\| | \|D out\| | D/P | `Dmod` min |
|---|---|---|---|---|
| P/I 0.160, D 0.0051 | 0.0218 | 0.0042 | 0.194 | 1.000 |
| P/I 0.170, D 0.0051 | 0.0167 | 0.0033 | 0.200 | 1.000 |
| P/I 0.170, D 0.0056 | 0.0205 | 0.0057 | **0.281** | 1.000 |

D's relative contribution rose 40% and the mode doubled. `Dmod` never left 1.000 and `Dlim%`
stayed 0.0, so this is not D-term saturation — it is D actively driving the resonance.

## The metrics disagreed with each other, and that matters

`segments PIT` on the same two windows:

| | nerr | ratio | corr | ev | over% | rise | ring |
|---|---|---|---|---|---|---|---|
| P/I 0.170, D 0.0051 | 0.96 | 1.27 | 0.67 | 7 | 87.0 | 60 | 0.0 |
| P/I 0.170, **D 0.0056** | **0.58** | 1.19 | **0.87** | 15 | **55.4** | 80 | 1.0 |

Every transient-tracking metric **improved** — which is what D is supposed to do. Meanwhile the
resonance doubled and the pilot's immediate judgement was that it felt worse.

Both readings are correct. `nerr`, `corr` and `over%` are dominated by the response to commanded
manoeuvres; the 6.2 Hz mode is a separate, continuously excited contribution that those statistics
average over. **The pilot feels the resonance, not the tracking statistic.**

This is the second time a summary metric has pointed the wrong way on this airframe. `ring`
returned 0.0 at pitch 0.170 and 0.190 while 6.2 Hz dominated. The standing rule stands and is
now stronger: **judge this airframe on the rate-error spectrum.**

## Decision

**`ATC_RAT_PIT_D` does not go up.** The next test sweeps it *down*.

The mechanism is phase, not magnitude. The pre-flight model put D at +39° of net lead at 6.2 Hz —
nominally still damping — by accounting only for `INS_GYRO_FILTER` and `ATC_RAT_PIT_FLTD`. The
measurement says the total loop lag at 6.2 Hz is larger than that: ESC/motor response and airframe
structural phase are not in the model. Enough additional lag and D's contribution flips from
damping the mode to reinforcing it, which is what was recorded.

**This also promotes evidence previously logged as weak.** The 2026-08-03 roll D sweep found
D 0.0052 → 0.0053 moving `ring` 1.0 → 2.5, which was set aside because a ±6% sweep could not be
resolved against its own scatter. Both axes now point the same way on independent flights.

Superseded by this result: the earlier recommendation that `ATC_RAT_PIT_FLTD` 10 → 15 would be the
lever if the mode rose with D. FLTD 10 → 15 removes ~9.3° of lag at 6.2 Hz but adds ~8.7% D
magnitude, and magnitude is currently working against the aircraft. It is a weaker bet than simply
reducing D. Not ruled out — deferred until the downward D sweep has bounded the effect.

## Correction — manoeuvre intensity was not controlled

Re-analysed the same log after adding `attRMS`, `mode` and `lean` columns to
`pidreview.py segments`. `lean` is mean demanded lean angle, a proxy for how hard the aircraft
was being flown.

| Window | mode (5–7 Hz) | lean | **mode/lean** | **attRMS** |
|---|---|---|---|---|
| P/I 0.170, D 0.0051 | 0.55 | 6.5° | 0.085 | 1.66 |
| P/I 0.170, **D 0.0056** | 1.09 | 8.6° | **0.127** | **1.61** |

Two things follow:

- The rise is **+49%**, not +106%. Roughly half the original figure was the pilot flying harder.
- **`attRMS` improved slightly** (1.66 → 1.61). On the attitude loop, more D was not worse.

The conclusion that D should not go up survives, but on weaker evidence than first reported, and
with one metric dissenting. Subsequent flight `2026-08-05 15-40-23` swept D 0.0047–0.0053 and
found `attRMS/lean` **flat** at 0.165–0.195 with within-gain scatter exceeding between-gain
spread — so the standing conclusion is now **"D has no measurable effect across ±8%"** rather
than "D feeds the mode".

## Confidence limits — read these before acting on the above

- **The D comparison rests on two windows, one step apart.** 15 and 7 events respectively. The
  direction is clear at +106%, well outside the ±20% fixed-gain scatter established for this
  airframe, but the *magnitude* of the effect is a single measurement.
- **Wind was higher than the earlier sortie**, pilot-reported and confirmed in the data: the
  identical 0.160 / 0.0051 configuration read err rms **10.47** here versus **6.16** on flight
  `14-06-45` about an hour before. Absolute numbers from this flight are **not** comparable to
  that one.
  The D conclusion is unaffected because both D windows are *within this flight* at fixed
  P/I 0.170 — which is precisely why gains are swept in flight rather than across flights.
- **Linearity is assumed, not measured.** That more D worsens the mode does not guarantee less D
  improves it by a proportional amount. The downward sweep tests exactly this and may return a
  much smaller effect.
- **The sweep was aborted after one step**, so windows at 0.0059 and 0.0063 do not exist. No claim
  is made about behaviour above 0.0056.
- The 0.0052 and the second 0.170 / 0.0051 windows recorded **0 events** — near-pure hover. They
  are unusable for tracking metrics and are excluded from every comparison above.

## Tooling defect found — grid anchoring in `pitch-d-step.lua`

The commanded sequence was 0.0051 → 0.0055 → 0.0059. The log shows 0.0051 → **0.0056** → 0.0052.

`round_step()` snapped values to integer multiples of `STEP`, a grid anchored on **zero**. The boot
value 0.0051 is not on the 0.0004 grid, so the first step overshot to 0.0056 (+9.8% rather than the
intended +7.8%) and the step back down could not return to 0.0051.

Fixed — the grid is now anchored on the boot value:

```lua
local BOOT_D = pitch_d:get()
local function round_step(value)
   return BOOT_D + math.floor(((value - BOOT_D) / STEP) + 0.5) * STEP
end
```

Verified: `0.0051 → 0.0049 → 0.0047 → 0.0045 → 0.0047 → 0.0049 → 0.0051`, returning exactly to the
boot value. `STEP` reduced to **0.0002** for the downward sweep.

**Step size was not the cause of the bad result.** A +7.8% step instead of +9.8% would have
produced a smaller worsening, not an improvement — the mechanism is phase and does not reverse
with step size.

## Health and telemetry

| | |
|---|---|
| Clips | **0** |
| VIBE | median 5.4 · p95 11.5 · max 21.6 m/s² |
| `PM.Load` | max 353 / 1000 |
| `PM.NLon` | max 2 |
| `PM.Mem` | min 271 192 B free |
| `Dmod` | 1.000 throughout |
| `Dlim%` | 0.0 throughout |

No EKF messages, no failsafes, no PreArm faults other than the expected
`PreArm: Motors Emergency Stopped` at t=110.9 and t=200.9 — RC13 MotorEStop working as designed.

The gap between t≈110 and t≈141 is a deliberate ground stop: E-stop engaged, RC option assignments
remapped from the GCS, E-stop released. `PTDS: ignored - aircraft not flying` at t=132.7 and 134.8
confirms the stepper's armed-and-flying guard rejecting switch flicks on the ground. Both scripts
correctly re-acquired their options after the remap — the runtime re-query fix from 2026-08-04 is
working.

## Next step

Sweep `ATC_RAT_PIT_D` downward at pitch P/I 0.170, one bracketing step up first so the local
gradient is measured on both sides of the saved value:

| Window | Pitch P/I | Pitch D |
|---|---|---|
| 1 | 0.170 | 0.0053 |
| 2 | 0.170 | 0.0051 |
| 3 | 0.170 | 0.0049 |
| 4 | 0.170 | 0.0047 |

≥ 15 s per window, continuous pitch reversals, target ≥ 6 events. Return to 0.160 / 0.0051 before
landing — swept values persist through disarm and are cleared only by a reboot.

Judge on the 6.2 Hz component of pitch rate error. **Not** `ring`, and **not** `nerr` — on this
flight `nerr` moved opposite to the effect the pilot could feel.

Expectation is deliberately low. The ~5.5–6.2 Hz mode is a permanent airframe property that has
only ever been minimised, and D scales one contribution to it. A partial reduction is the realistic
best case. If window 3 shows no improvement over window 2, the question is closed and the next
lever is `ATC_ANG_PIT_P` — firmware default 4.5 in a 3.0–12.0 range, never moved.
