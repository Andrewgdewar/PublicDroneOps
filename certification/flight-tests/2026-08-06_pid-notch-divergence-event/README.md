# Scout-15 PID Error Notch — Roll Divergence Event

**Aircraft:** Scout-15 quadcopter
**Flight event:** 2026-08-06
**Flight log:** `2026-08-06 17-20-42.bin`
**Event classification:** Divergent closed-loop oscillation induced by control-path filter phase lag
**Aircraft outcome:** Two short hops, both diverged, both landed immediately. No crash, no damage, zero clips
**Corrective action:** `ATC_RAT_RLL_NEF` reverted 1 → 0 **and rebooted**; `FILT1_TYPE` 1 → 0
**Root cause:** Confirmed from log — measured, see below
**Objective:** Test a 6 Hz PID error notch on roll against the airframe's ~6.1 Hz mode, with pitch
left un-notched as an in-flight control axis

## Summary

The notch worked exactly as designed on the frequency it was aimed at, and destabilised the
aircraft at a different one.

A PID error notch was placed at 6.0 Hz on the roll axis to attenuate the airframe's dominant
6.12 Hz rate-error mode. Pitch was deliberately left clean as a same-flight control. Both hops
developed a **divergent 4.65 Hz roll oscillation** and were landed within about 7 seconds.

The oscillation frequency was **not** the notched frequency. It was a pre-existing, previously
harmless 4.7 Hz mode that the notch's lower skirt destabilised by removing phase margin the
aircraft did not have to spare.

**This was a control-loop stability failure, not a mechanical or noise failure.** Zero clips,
vibration well inside limits, no EKF or failsafe events.

## Configuration proof — as flown, read from the log

| Item | Value |
| --- | --- |
| Firmware | ArduCopter 4.8.0-dev, commit `32e43699` |
| `FILT1_TYPE` | **1** (notch) |
| `FILT1_NOTCH_FREQ` / `Q` / `ATT` | **6.0 / 5 / 15** |
| `ATC_RAT_RLL_NEF` | **1** — armed, verified pre-flight and confirmed in log |
| `ATC_RAT_PIT_NEF` | **0** — control axis, left clean |
| `ATC_RAT_RLL_NTF` / `PIT_NTF` | 0 / 0 — error path only, not target |
| Rate PIDs | **unchanged from baseline**: RLL 0.160/0.160/0.0052, PIT 0.160/0.160/0.0051 |
| `ATC_ANG_RLL_P` / `PIT_P` | 6.0 / 4.5 — unchanged |
| `ATC_RAT_*_FLTT` / `FLTD` / `FLTE` | 10 / 10 / **0** — unchanged |
| `INS_GYRO_FILTER` | 18 — unchanged |
| `INS_HNTCH_OPTS` / `HMNCS` / `BW` / `MODE` | 0 / 3 / 15 / 3 — unchanged |
| Armed time | 15.2 s across two spans (7.1 s, 8.1 s) |
| Scripted param streams | all empty — no stepper script ran |

`FLTE = 0` matters for the analysis below: it makes the logged `PIDR.Err` the **direct output of
the notch**, with no subsequent low-pass to confound it.

Every value in the table above was read back **from the log itself**, not from a saved parameter
file, and compared against the 2026-08-04 baseline log the same way. Rate gains, angle gains,
filters and harmonic-notch settings are all identical to that baseline. **The PID notch was the
only deliberate change. Single variable confirmed.**

> Note on provenance: `certification/parameters/0806.param` was the as-flown snapshot at the time
> of this flight, but it has since been re-downloaded after the corrective action and no longer
> reflects this configuration. The log is the authoritative record.

## Results

### Cross-axis comparison — the design that made this readable

| axis | notch | rms_err deg/s | ratio | corr |
| --- | --- | --- | --- | --- |
| **RLL** | **ON** | **42.654** | 3.39 | **−0.33** |
| PIT | off | 11.650 | 1.06 | 0.28 |
| *RLL, baseline 08-04 19-57-57* | *off* | *3.652* | *1.23* | *0.87* |

Roll was **11.7× worse than its own baseline** and **3.7× worse than the un-notched axis in the
same air, same battery, same pilot, same flight**. A negative correlation of −0.33 means the roll
controller output was anti-phase with demand — it was driving the oscillation, not correcting it.

### Divergence, not ring

RMS roll rate error, each hop split into thirds:

| | 1st third | 2nd third | 3rd third |
| --- | --- | --- | --- |
| **Hop 1** | 0.04 | 9.88 | **79.55** |
| **Hop 2** | 0.19 | 5.39 | **66.83** |

Monotonic growth of roughly 250× and 350×. This is a diverging closed-loop instability, not a
decaying ring and not turbulence. Both hops behaved identically — the result is repeatable, n = 2.

Peak roll rate error **173.9 deg/s**, yet attitude stayed within −9.5° to +4.7°. Those are
consistent: a 4.65 Hz oscillation at 174 deg/s peak implies an amplitude of
174 / (2π × 4.65) = **5.96°**. A violent buzz about a near-level attitude, which is why it remained
landable.

### Frequency — the notch missed the frequency that mattered

Roll rate-error spectrum, dominant peak per band:

| band | **baseline 08-04** (no notch) | **this flight, hop 1** | change |
| --- | --- | --- | --- |
| 3.5–4.2 Hz | 3.67 Hz, amp 0.078 | 4.08 Hz, amp 4.772 | 61× |
| **4.2–5.0 Hz** | **4.70 Hz, amp 0.083** | **4.65 Hz, amp 10.521** | **127×** |
| 5.0–5.8 Hz | 5.72 Hz, amp 0.140 | 5.07 Hz, amp 2.545 | 18× |
| **5.8–6.6 Hz** | **6.12 Hz, amp 0.169 ← dominant** | 6.20 Hz, amp 0.421 | **2.5×** |
| 6.6–7.5 Hz | 6.65 Hz, amp 0.056 | 7.46 Hz, amp 0.276 | 5× |

Two things are visible at once:

1. **The notch suppressed its target.** The 6.12 Hz mode was the dominant peak at baseline. In a
   flight where broadband error grew 127×, it grew only 2.5×. Relative to everything around it, the
   notch attenuated the mode it was aimed at very effectively.
2. **The 4.7 Hz mode already existed and was harmless** — amplitude 0.083, about half the 6.12 Hz
   peak, well damped, never a problem across the entire tuning campaign. Under the notch it became
   the dominant peak at 10.521 and diverged.

## Root cause — notch skirt phase lag spent a margin that was not there

Because `FLTE = 0`, `PIDR.Err` is the notch output and `RATE.RDes − RATE.R` is its input. Their
ratio is the **notch's own transfer function, measured in flight**. Values below are corrected for
the ~1.3 dB wideband offset seen at 1.1 Hz, where notch gain should be unity:

| frequency | attenuation | **phase** |
| --- | --- | --- |
| 2.25 Hz | −0.3 dB | −6.1° |
| 3.80 Hz | −0.0 dB | −12.0° |
| 4.23 Hz | −0.1 dB | −14.1° |
| **4.65 Hz ← oscillation** | **−0.4 dB** | **−17.7°** |
| 5.07 Hz | −0.9 dB | −25.7° |
| **6.20 Hz ← notch centre** | **−6.8 dB** | +9.3° |
| 6.62 Hz | −3.4 dB | +22.4° |

At the frequency that went unstable the notch delivered **essentially no attenuation and 17.7° of
phase lag**. That is the worst available trade: all of the cost, none of the benefit.

This is inherent to the filter, not a misconfiguration. A notch produces phase **lag below** its
centre and phase **lead above** it. For an ideal biquad at $f_0 = 6$, $Q = 5$:

$$\angle H(j\omega) = -\arctan\!\left(\frac{\omega_0\omega/Q}{\omega_0^2-\omega^2}\right) = -22.1° \text{ at } 4.65 \text{ Hz}$$

The measured −17.7° is smaller because attenuation was limited to 15 dB rather than infinite. Model
and measurement agree, so the effect is predictable in advance for any future notch placement.

**Why that 17.7° was fatal:** the roll axis was flying at 0.160 against a measured oscillation onset
of 0.170 — a gain margin of $20\log_{10}(0.170/0.160) = 0.53$ dB, roughly 94% of the way to the
stability boundary. An aircraft in that state has no phase budget to spend on anything. The notch
spent about 18° of it at 4.65 Hz, and the pre-existing lightly-damped mode there crossed into
instability. Adding lag also lowers the frequency at which total open-loop phase reaches −180°,
which is why the oscillation appeared at 4.65 Hz rather than at the 6.12 Hz mode.

**The notch was not the wrong tool. It was the right tool applied to an aircraft with no margin to
pay for it.** This is precisely the failure that Leonard Hall's documented filter procedure exists
to prevent: when filtering is increased, PID gains must be reduced and the aircraft retuned. Only
the first half of that was done. See `tuning.md` §5.4 and §10.2.

## What was *not* the cause

- **Not vibration or noise.** Zero clips across the whole log. VIBE median 0.19 / 0.18 / 0.14,
  p95 4.8 / 6.0 / 5.8, max 10.7 / 12.6 / 12.7 — all far below the 30 abort threshold.
- **Not a configuration error.** All six notch parameters verified before flight and re-read from
  the log. `ATC_RAT_RLL_NEF` did not snap back to 0.
- **Not the notch failing to engage.** It measurably attenuated 6.8 dB at its centre.
- **Not a wrong notch frequency for the stated target.** 6.12 Hz genuinely was the dominant
  baseline mode; 6.0 Hz was a correct choice *for that mode*.
- **Not CPU.** `PM.Load` max 345, `PM.NLon` max 4.
- **Not gyro health.** A `PreArm: Gyros inconsistent` message appears at boot but cleared before
  arming. Gyros were calibrated cold at 23.8 °C versus 39.5 °C previously; this affects run-to-run
  repeatability but cannot produce a 127× coherent single-frequency divergence.

## Secondary finding — `SMAX = 50` is decorative and provided zero protection

`ATC_RAT_RLL_SMAX = 50` was in force throughout. During a genuinely divergent oscillation reaching
174 deg/s of rate error:

```
PIDR.Dmod   min 1.000    0.00% of samples below 1.0
PIDR.SRate  whole hop: median 0.53  p95 18.44  max 20.03
PIDR.SRate  last 2 s : median 13.27 p95 19.84  max 20.03
```

**The slew limiter never activated.** Peak slew rate was 20.03, so `SMAX = 50` sits 2.5× above the
worst case this aircraft has ever produced. It cannot engage under any condition short of a crash.

Modelled engagement at candidate values, hop 1:

| `SMAX` | % of hop | % of final 2 s |
| --- | --- | --- |
| 50 (current) | 0.0 | 0.0 |
| 25 | 0.0 | 0.0 |
| 20 | 0.5 | 1.9 |
| 15 | 10.0 | 35.6 |
| **12** | **16.9** | **60.1** |
| 10 | 19.7 | 69.9 |

`SMAX = 12` would have been actively suppressing P+D through 60% of the divergence. This upgrades
the pending `SMAX` 50 → 12 change from a theoretical tidy-up to an evidenced safety improvement,
now supported by data from a real oscillation event rather than from hover statistics.

## Decision

1. **`ATC_RAT_RLL_NEF` → 0, and reboot.** The notch does not detach from the PID until reboot;
   clearing the parameter alone is insufficient. `FILT1_TYPE` → 0 to remove it fully.
2. **Do not retry a PID notch until the gain margin exists to pay for it.** The correct order is
   gain reduction first, notch second. Attempting the notch first has now been tested and failed.
   **When it is retried, `Q` is the lever — not depth, and not a different centre frequency.**
   See the design trade below.
3. **Adopt `SMAX = 12`** on roll and pitch, on the evidence above.
4. The next roll mission is the one already deferred twice: **scale P, I and D down together**
   (×1.00 / ×0.85 / ×0.70) to open real gain margin. Every documented method prescribes it and it
   has still never been flown.

## Notch design trade — computed from the firmware biquad, model validated above

Benefit is attenuation at the two real modes. Cost is phase lag at 4.65 Hz, the mode that diverged.

| config | 6.12 Hz | 5.72 Hz | **lag @ 4.65 Hz** | lag @ 3.67 Hz | dB per degree |
| --- | --- | --- | --- | --- | --- |
| **as flown** `f0=6 Q=5 ATT=15` | −11.9 dB | −7.1 | **−18.1°** | −9.5° | 0.66 |
| **move onto 4.7 Hz** | **−0.6** | −1.1 | **−23.9°** | **−18.6°** | ~0 |
| `ATT=6` | −5.6 | −4.2 | −10.7° | −5.7° | 0.52 |
| `ATT=3` | −2.9 | −2.3 | −6.1° | −3.3° | 0.47 |
| **`Q=12`** | −6.9 | −2.4 | **−7.7°** | −3.9° | **0.90** |
| `Q=20` | −4.0 | −1.0 | −4.6° | −2.3° | 0.87 |

- **Re-centring the notch on 4.65 Hz is strictly worse in every column.** It lands on its own steep
  skirt so the lag there *increases*, it abandons the 6.12 Hz mode, and it loads 18.6° onto the
  3.67 Hz mode beneath. It would walk the instability down the ladder. The 4.65 Hz oscillation was
  also iatrogenic — un-notched that mode is amplitude 0.083 and harmless.
- **Reducing depth is the weakest lever** — efficiency gets *worse*, from 0.66 to 0.47 dB per
  degree. It buys less benefit than it saves cost.
- **Raising `Q` is the only option that improves the trade** rather than sliding along it. `Q=12`
  buys 58% of the attenuation for 43% of the lag.
- **The catch on high `Q`:** the mode wanders 5.5–6.2 Hz between flights, and at `Q=12` the notch
  gives only 2.4 dB at 5.72 Hz. A narrow notch that misses costs phase and returns nothing, so it
  must be centred on a peak measured from the flight immediately preceding.

Note `setup_notch_filter()` passes bandwidth as `FREQ/Q`, and `calculate_A_and_Q()` then re-derives
Q from that bandwidth via the octaves formula. `FILT1_NOTCH_Q = 5` produced an **effective Q of
4.737**. The parameter is not the filter's actual Q.

## Confidence limits

- **Root cause: ~90%.** The phase lag is measured in flight, not modelled, and it agrees with the
  analytic biquad prediction within 4.4°. The 4.7 Hz mode is confirmed present in the baseline. The
  divergence is repeatable across two hops. The cross-axis control was clean and same-flight.
- **The residual 10%** is that gain margin at 4.7 Hz was never independently measured — the 0.53 dB
  figure comes from a sweep that found the onset at 5.5–6.2 Hz. The 4.7 Hz margin is inferred to
  have been small, not proven.
- **Hops were short (7.1 s, 8.1 s) and the aircraft never reached settled hover.** Absolute rms
  values are therefore not comparable to a full-duration flight. The cross-axis ratio and the
  spectral peaks are the trustworthy quantities; the 11.7× baseline multiple is indicative only.
- **`SMAX` engagement percentages are modelled** from logged `SRate` against candidate thresholds.
  They do not account for the limiter's own 25 Hz internal low-pass or its 10% gain floor, so real
  engagement would differ somewhat in duty cycle, though not in direction.

## Health and telemetry

| Metric | Value | Limit | Status |
| --- | --- | --- | --- |
| Clip count | **0** | 0 | pass |
| VIBE median | 0.19 / 0.18 / 0.14 | < 15 | pass |
| VIBE max | 10.74 / 12.61 / 12.70 | < 30 | pass |
| `PM.Load` max | 345 | — | nominal |
| `PM.NLon` max | 4 | — | nominal |
| EKF / failsafe | none | — | pass |
| Attitude excursion | −9.5° to +4.7° roll | 60° | pass |

Boot messages confirmed `RCOut: DS300:1-4`, `RC Protocol: SBUS`, UM982 GPS at 230400,
`Frame: QUAD/X`. `RC13: MotorEStop HIGH` and the associated `PreArm: Motors Emergency Stopped`
are the expected pre-flight safety interlock.

## Next step

Revert and reboot, then fly the gain-reduction sortie. The PID notch remains a valid technique for
this airframe and should be revisited **after** margin exists — at which point it should be centred
by measurement and its skirt checked against every other lightly-damped mode in the 3–8 Hz band,
not just the target.
