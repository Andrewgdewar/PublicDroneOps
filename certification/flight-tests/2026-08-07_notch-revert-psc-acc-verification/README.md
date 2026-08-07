# Scout-15 Notch Revert + Vertical Controller Correction — Verification Flight

**Aircraft:** Scout-15 quadcopter
**Flight event:** 2026-08-07
**Flight log:** `2026-08-07 08-47-02.bin`
**Objective:** Confirm the PID notch is gone, and verify four setup corrections applied after the
2026-08-06 divergence event
**Outcome:** Clean flight. **Best tracking metrics on record on both axes.** Zero clips
**Parameter snapshot:** `certification/parameters/0807.param` (`STAT_BOOTCNT` 332, `STAT_FLTCNT` 122)

## Summary

Following the 2026-08-06 PID notch divergence
([record](../2026-08-06_pid-notch-divergence-event/README.md)), the notch was removed and four
setup parameters corrected. This flight verifies all of it.

Everything checked out. The aircraft returned to normal handling, recorded its best tracking
numbers to date, and the vertical-controller correction produced a large, unambiguous, directly
attributable improvement.

## Configuration proof — read from the log

| Item | Value |
| --- | --- |
| Firmware | ArduCopter 4.8.0-dev, commit `32e43699` |
| `FILT1_TYPE` | **0** — notch removed, `FILT1_NOTCH_*` deallocated |
| `ATC_RAT_RLL_NEF` | 1 — **inert**, see below |
| `ATC_RAT_*_SMAX` | **12** (was 50) |
| `PSC_D_ACC_P` / `_I` | **0.017 / 0.034** (were 0.05 / 0.10) |
| `INS_LOG_BAT_MASK` / `_OPT` | **0 / 0** |
| `INS_RAW_LOG_OPT` | **9** — primary gyro, pre *and* post filter |
| Rate PIDs | **unchanged**: RLL 0.160/0.160/0.0052, PIT 0.160/0.160/0.0051 |
| `ATC_ANG_RLL_P` / `PIT_P` | 6.0 / 4.5 — unchanged |
| `INS_GYRO_FILTER` | 18 — unchanged |
| `INS_HNTCH_OPTS` / `HMNCS` / `BW` / `MODE` | 0 / 3 / 15 / 3 — unchanged |
| `MOT_THST_HOVER` | 0.1634 (learned; was 0.1697) |
| Armed | 83.8 s, one span, **Loiter throughout** |

### `ATC_RAT_RLL_NEF = 1` is inert, and this flight proves it

Mission Planner refuses to write 0 to this parameter — its bundled metadata disagrees with the
firmware's own `@Range: 0 8`. It was left at 1 deliberately, on a source trace: with
`FILT1_TYPE = 0`, `AP_Filters::update()` never allocates the filter, `get_filter(1)` returns
`nullptr`, the `filter != nullptr` guard in `AC_PID::set_notch_sample_rate()` short-circuits, and
the notch object is left **allocated but uninitialised**. `NotchFilter::apply()` then hits
`if (!initialised)` and returns the sample unchanged.

The flight data confirms it. Roll error at 4.65 Hz is back to the noise floor (see below), and roll
`rms_err` is 3.086 against 42.654 when the notch was live. **The notch is not in the loop.**

## Results

### Tracking — best on record, both axes

| axis | rms_err | ratio | corr | | 08-04 baseline | change |
| --- | --- | --- | --- | --- | --- | --- |
| **RLL** | **3.086** | **1.13** | **0.93** | vs | 3.652 / 1.23 / 0.87 | **−15.5%** rms |
| **PIT** | **4.455** | **1.00** | 0.77 | vs | 5.114 / 1.07 / 0.73 | **−12.9%** rms |
| YAW | 3.886 | 0.81 | 0.89 | | — | — |

Gains are **identical** to the baseline, so this is not a gain comparison. Both flights were flown
in Loiter. See confidence limits — the improvement is real but **not attributable** to any single
change.

### Step response

| axis | events | over% | rise ms | settle ms | ring |
| --- | --- | --- | --- | --- | --- |
| RLL | 10 | 20.0 | 135 | 974 | **2.5** |
| PIT | 8 | 22.2 | 120 | 997 | 0.5 |
| YAW | 7 | 30.5 | 211 | 973 | 1.0 |

**Roll still rings five times as much as pitch.** That is the unresolved item and it is exactly
what the deferred gain-reduction sortie targets.

### `PSC_D_ACC` correction — large, attributable, confirmed

`PIDA.SRate` is the vertical controller's own P+D slew rate, so it measures the changed loop
directly:

| | median | p95 | max |
| --- | --- | --- | --- |
| Baseline 08-04 | 0.25 | 0.54 | **13.62** |
| **This flight** | **0.15** | **0.22** | **0.28** |

**Peak fell 49×; p95 fell 2.5×.** The prior measurement had found `PIDA.Act` peaking at 13.65
against a `PIDA.Tar` of 2.63 — the vertical controller was tracking noise rather than demand. It no
longer is. The gains were 2.95× above the documented rule `PSC_D_ACC_P = 0.1 × MOT_THST_HOVER`,
`_I = 0.2 × MOT_THST_HOVER`; they are now correct for this aircraft.

This is the one result on this flight that survives the two-flight-comparison objection — 49× is
far outside any plausible scatter, and it is measured on the loop that was changed.

### `SMAX` 50 → 12 — engages never in normal flight, as designed

`Dmod` held at **1.000**, `Dlim%` **0.0** across the whole flight. Peak roll `SRate` in normal
flight has historically been 5.9, so a threshold of 12 sits comfortably above routine activity
while still being reachable during a divergence — during the 08-06 event `SMAX=12` would have been
active for 60% of the final two seconds. It is now a live safety net instead of a decorative one.

### Airframe mode — measured properly for the first time

Raw gyro logging at 1923 Hz, 126269 samples, 0.235 Hz resolution:

```
instance 0 [PRE-filter]   GyrX (roll ) peaks: 5.9 Hz, 5.6 Hz, 199.5 Hz
instance 2 [POST-filter]  GyrX (roll ) peaks: 5.9 Hz, 5.6 Hz, 6.1 Hz
```

| freq | roll PSD | | freq | roll PSD |
| --- | --- | --- | --- | --- |
| 4.70 Hz | 0.235 | | **5.87 Hz** | **8.303** |
| 5.16 Hz | 0.675 | | 6.10 Hz | 2.783 |
| 5.40 Hz | 1.672 | | 6.34 Hz | 0.683 |
| **5.63 Hz** | **7.437** | | 6.57 Hz | 0.268 |

**It is one broad mode centred ~5.75 Hz, not two.** An earlier reading of 5.72 and 6.12 Hz as
separate peaks came from lower-resolution data and is superseded.

**The 4.65 Hz "mode" is at 0.235 — the noise floor.** It was iatrogenic, created by the notch's
skirt lag, and it is gone with the notch.

Band power on the roll axis:

| band | pre | post | attenuation | share |
| --- | --- | --- | --- | --- |
| 0.5–2 pilot/wind | 9.502 | 9.541 | 0.0 dB | 58.62% |
| 2–4 | 0.920 | 0.920 | −0.0 dB | 5.65% |
| **4–7 airframe mode** | **5.475** | **5.356** | **−0.1 dB** | **32.91%** |
| 7–10 | 0.064 | 0.060 | −0.3 dB | 0.37% |
| 10–25 | 0.0093 | 0.0073 | −1.1 dB | 0.04% |
| 25–50 | 0.438 | 0.00035 | **−31.0 dB** | 0.00% |
| 50–70 motor 1st | 0.241 | 0.00011 | **−33.4 dB** | 0.00% |
| 70–140 motor 2nd | 1.180 | 0.000043 | **−44.4 dB** | 0.00% |
| 140+ | 13.23 | 0.00027 | **−46.9 dB** | 0.00% |

**−0.1 dB across the mode: no filter touches it.** This is genuine structural motion, and it is
**32.91% of all roll gyro energy**. Filter chain is healthy and comparable to the 08-04 baseline
(−24.8 / −35.8 / −44.7 / −46.4 dB), with 25–50 Hz notably better.

A useful asymmetry: pre-filter, roll's dominant peak is the **5.9 Hz airframe mode**, while pitch
and yaw are dominated by **106 Hz motor noise**. Roll is the axis carrying the structural mode.

## Decision

1. **Keep all four corrections.** All verified, none regressed anything.
2. **Raw logging stays on.** It cost nothing measurable and it is the only configuration that
   resolves this mode.
3. **The gain-reduction sortie is still the next mission.** Roll `ring` 2.5 against pitch 0.5, and
   a mode holding a third of all roll gyro energy, are unchanged by anything done here.
4. Clear `ATC_RAT_RLL_NEF` to 0 via MAVProxy when convenient — tidy-up only, removes the latent
   re-arm if `FILT1_TYPE` is ever set back to 1.

## Confidence limits

- **The `PSC_D_ACC` result is solid, ~95%.** It is measured on the loop that changed, the effect is
  49× on peak and 2.5× on p95, and the mechanism was predicted in advance.
- **The tracking improvement is real but not attributable, ~50% that it is more than scatter.**
  Four things changed at once and the standing rule is that scatter at fixed gains is ±20%
  relative. A 15.5% improvement sits **inside** that band. It is recorded as "best on record", not
  as "caused by the corrections". Only an in-flight sweep can attribute it.
- **Flight was 83.8 s against the baseline's 223 s.** `rms_err` and `corr` are duration-robust;
  maxima are not. The 49× `PIDA.SRate` max ratio is partly flattered by the shorter flight, which
  is why the p95 ratio of 2.5× is quoted alongside it.
- **Air was calm and vibration unusually low** (VIBE median 1.58/1.75/2.24 against a typical ~0.19
  hover figure measured differently, and max 6.36 against 12.7 the previous evening). Benign
  conditions plausibly contribute to the tracking numbers.
- **Mode frequency 5.87 Hz is now well established** — 0.235 Hz bins, 126269 samples, pre and post
  filter agreeing. Confidence ~95%.

## Health and telemetry

| Metric | Value | Limit | Status |
| --- | --- | --- | --- |
| Clip count | **0** | 0 | pass |
| VIBE median | 1.58 / 1.75 / 2.24 | < 15 | pass |
| VIBE max | 3.69 / 4.26 / 6.36 | < 30 | pass |
| `PM.Load` max | 388 | — | nominal |
| **`PM.NLon` max** | **0** | — | **no long loops at all** |
| `PM.Mem` min | 269856 | — | ample |
| EKF / failsafe | none | — | pass |
| `Dmod` | 1.000 | — | `SMAX` never engaged |

**Raw IMU logging at 1923 Hz cost zero scheduler overruns.** `PM.NLon` was 0 for the entire flight
against 4 on the previous, non-raw flight. CPU headroom on the H743 is not a constraint for this.

EKF3 IMU0 and IMU1 both yaw-aligned and using GPS, no compass, as designed. No `PreArm` messages.

## Next step

Fly the roll gain-reduction sweep — scale P, I and D together ×1.00 / ×0.85 / ×0.70 with pitch held
as the control axis. That is the only remaining lever on the 5.87 Hz mode and on the 0.53 dB gain
margin, and it now has a clean, well-characterised baseline to be measured against.
