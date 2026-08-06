# Scout-15 Raw IMU Filter Review

**Aircraft:** Scout-15 quadcopter
**Flight:** 2026-08-04
**Flight log:** `2026-08-04 19-57-57.bin`
**Objective:** Recovery verification after the notch oscillation event, plus a clean pre/post
filter spectrum from raw IMU logging
**Outcome:** Filter chain confirmed healthy. Best flight on record. A prior noise hypothesis refuted
**Related:** [`../2026-08-04_notch-multisource-oscillation-event/README.md`](../2026-08-04_notch-multisource-oscillation-event/README.md)

## Configuration proof

| Item | Value |
| --- | --- |
| Firmware | ArduCopter 4.8.0-dev, commit `32e43699` |
| `INS_HNTCH_OPTS` / `HMNCS` / `BW` / `MODE` | 0 / 3 / 15 / 3 (ESC telem) |
| `INS_GYRO_FILTER` | 18 |
| `INS_RAW_LOG_OPT` | **9** — primary gyro, pre and post filter. This flight only |
| Rate PIDs | RLL 0.150/0.150/0.0052, PIT 0.150/0.150/0.0051 |
| Armed | 222.9 s |

Only one parameter differed from the control path of the previous good flight. `INS_RAW_LOG_OPT`
was returned to 0 afterwards.

## Recovery confirmed

| Flight | Roll rms err | Pitch rms err |
| --- | --- | --- |
| 08-03 15-03 | 4.66 | 5.29 |
| 08-03 15-58 | 4.92 | 8.81 |
| 08-04 19-12 (event) | **26.53** | 24.64 |
| **08-04 19-57** | **4.09** | **4.21** |

Best result recorded on this airframe. Zero clips.

## Frequency-domain result

`pidreview.py filterreview`, roll axis, 250,041 samples per instance at 1923 Hz:

| Band (Hz) | Attenuation | Share of post-filter energy |
| --- | --- | --- |
| 0.5–2 pilot/wind | 0.0 dB | **72.72%** |
| 2–4 | −0.0 dB | 9.83% |
| **4–7 airframe mode** | −0.1 dB | **11.82%** |
| 7–10 | −0.2 dB | 0.38% |
| 10–25 | −1.3 dB | 0.05% |
| 25–50 | **−24.8 dB** | 0.00% |
| 50–70 motor 1st | **−35.8 dB** | 0.00% |
| 70–140 motor 2nd | **−44.7 dB** | 0.00% |
| 140+ | **−46.4 dB** | 0.00% |

Pre-filter 1/f-corrected peaks: roll 199.8 Hz, pitch 103.8 Hz, yaw 109.6 Hz — motor harmonics,
fully removed. Post-filter, the only surviving structure is at low frequency.

## Prior hypothesis refuted

A working assumption held that a noise source existed in the **25–50 Hz** band, below the motor
fundamental and therefore unreachable by the harmonic notch. That assumption was inferred from an
earlier `INS_GYRO_FILTER` 18 → 25 vibration event and had never been measured.

Pre-filter, 25–50 Hz measures **0.1146** against **9.103** in 0.5–2 Hz — roughly 1% of the
low-frequency energy before any filtering, and **0.00%** of total after. There is no such source.

**Caveat:** measured on a gentle flight. Vibration scales with throttle, so this does not
characterise the high-throttle regime in which the original 18 → 25 event occurred. The gyro
low-pass remains at 18 pending evidence from that regime.

## The 5.5 Hz mode

Post-filter roll PSD shows a distinct peak at **5.40 Hz**, roughly 4x the local background:

```
 3.99 Hz   0.2943    trough
 4.93 Hz   0.6066
 5.40 Hz   1.221     peak
 5.63 Hz   0.9943
 6.10 Hz   0.3963
 7.04 Hz   0.0647
```

Pre and post filter traces are identical below 10 Hz. The mode is therefore **real airframe motion
correctly measured by the gyro**, not sensor noise. No filter can act on it, and notching inside
the control bandwidth would cost phase margin.

## Decision

No filter chain changes. **99.9% of what the rate controller sees is below 10 Hz** — the band where
gains act and filters do not. Tuning effort directed to rate P/I sweeps instead.

## Health and telemetry

Zero clips. `Dmod` 1.00, `Dlim%` 0.0. Notch tracking verified in a separate check:
`FTNS.NF` median 49.86 Hz against measured motor fundamentals of 47.8 / 49.3 / 52.1 / 53.8 Hz,
confirming `SERVO_BLH_POLES=24` is correct.

## Tooling added

`pidreview.py filterreview <log>` — parses the raw `GYR` stream directly rather than through the
array cache, auto-identifies pre versus post filter instances by high-frequency energy rather than
assuming an instance numbering scheme, and reports band power with attenuation and share.
