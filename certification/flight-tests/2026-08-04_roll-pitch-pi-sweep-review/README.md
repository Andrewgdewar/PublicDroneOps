# Scout-15 Roll and Pitch P/I Sweeps

**Aircraft:** Scout-15 quadcopter
**Flight:** 2026-08-04
**Flight log:** `2026-08-04 20-47-54.bin`
**Objective:** Sweep roll P/I downward; obtain first pitch P/I data
**Outcome:** **Higher P/I is better on both axes.** The documented "back off 25%" heuristic does
not describe this airframe
**Defect found:** RC channel caching bug in the stepper scripts — fixed

## Configuration proof

| Item | Value |
| --- | --- |
| Firmware | ArduCopter 4.8.0-dev, commit `32e43699` |
| `INS_HNTCH_OPTS` / `HMNCS` | 0 / 3 |
| `INS_GYRO_FILTER` | 18 |
| `ATC_RAT_RLL_D` / `PIT_D` | 0.0052 / 0.0051, held |
| Boot P/I both axes | 0.150 |
| Steppers | `roll-pi-step.lua` 303/304, `pitch-pi-step.lua` 305/306, step 0.01 |

## Roll result — clean window, pitch held at 0.150

`pidreview.py segments RLL`

| P/I | ev | nerr | corr | rise ms | ring |
| --- | --- | --- | --- | --- | --- |
| 0.130 | 10 | 0.48 | 0.92 | 96 | 1.0 |
| 0.140 | 12 | 0.42 | 0.95 | 97 | 1.0 |
| 0.150 | 11 | 0.44 | 0.94 | 95 | 1.0 |
| **0.160** | 4 | **0.33** | **0.97** | **57** | 1.0 |

Reducing P/I degrades tracking monotonically. No ringing anywhere in 0.130–0.160. Combined with the
0.170 = ring 5.0 result from `20-03-11`, the roll boundary lies between 0.160 and 0.170.

## Pitch result

| P/I | ev | nerr | corr | rise ms | ring |
| --- | --- | --- | --- | --- | --- |
| 0.130 | 8 | 0.85 | 0.74 | 70 | 0.5 |
| 0.140 | 11 | 0.86 | 0.74 | 65 | 0.0 |
| 0.150 | 6 | 0.77 | 0.80 | 45 | 0.5 |
| **0.160** | 8 | **0.73** | **0.86** | 49 | **0.0** |

Monotonic improvement in `nerr` and `corr`. **Pitch rings less than roll at identical gains**
(0.0–0.5 versus a consistent 1.0), so pitch has headroom roll does not. Its boundary was not
reached.

Note the asymmetry: pitch at its best (`nerr` 0.73, `corr` 0.86) is still worse than roll at its
worst (0.48, 0.92). Pitch is substantially under-gained relative to roll.

## Validity of the pitch data

Roll gains were also stepped during the pitch windows because of the defect below. The pitch data
remains valid: **roll recorded 0 events in every window of that period** — the pilot was flying
pitch reversals only, and the roll and pitch rate loops are independent. The confound is limited to
cross-axis coupling, which is secondary.

## Defect — RC channel caching in the stepper scripts

Both steppers called `rc:find_channel_for_option()` once and cached the result. RC9 and RC10 were
reassigned from 303/304 to 305/306 at runtime without a reboot, leaving the **roll** script still
driving the same physical switches while the **pitch** script also acquired them. From t=418 s both
axes stepped together on every flick:

```
t=418.4   RPIS 0.140   PPIS 0.140
t=459.8   RPIS 0.130   PPIS 0.130
t=476.5   RPIS 0.140   PPIS 0.140
```

**Fix:** both scripts now re-query the option every loop and reset switch state when it disappears,
so a runtime reassignment is picked up instead of silently driving another script's switch.

## Second defect — analysis tooling

`PPIS` was registered in `pidreview.py`'s stream map but never added to the parser, so pitch
collapsed to the boot value. Fixed structurally: the parser now derives its stream list from
`SCRIPTED_PARAM_STREAMS`, so registering a stepper is a single-line change that cannot half-land.

## Frequency-domain note

P-induced and D-induced ringing occur at the **same** frequency on this airframe:

| Condition | Dominant frequency |
| --- | --- |
| P/I 0.150 | 5.4, 4.2, 6.0, 5.5 Hz |
| P/I 0.170 | 5.9, 6.0, 5.5, 5.8 Hz |
| D 0.0052 | 6.1, 6.1, 6.1, 5.5 Hz |
| D 0.0053 | 5.8, 6.1, 5.9, 5.8 Hz |

Because the frequency does not shift with gain, the standard technique of raising P to the ringing
point and then adding D to damp it does not apply — both gains excite the same resonance.

## Health and telemetry

Zero clips. `Dmod` 1.00, `Dlim%` 0.0 throughout.

## Next step

Pitch sweep to 0.170 and 0.180 to locate its boundary, one axis per sortie. Repeat roll 0.160 with
a longer window; the current reading rests on 4 events.
