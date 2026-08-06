# Scout-15 25 Hz Gyro Filter Validation

**Date:** 2026-07-19  
**Log:** `2026-07-19 10-51-00.bin`  
**Comparison:** `2026-07-13 10-27-23.bin` at 18 Hz  
**Aircraft:** Scout-15 with T-Motor MS1504 15x5.6 props

## Decision

`INS_GYRO_FILTER=25` **passes** and is now the frozen gyro LPF setting for PID
reference and SystemID work.

The higher cutoff adds a small, acceptable amount of residual high-frequency noise
while materially reducing control-band phase lag. There was no clipping, D limiting,
thermal issue, ESC telemetry issue or visible instability.

## Configuration proof

The log embedded the intended test configuration:

- `INS_GYRO_FILTER=25`
- `INS_RAW_LOG_OPT=9` (primary gyro pre- and post-filter)
- ESC harmonic notch enabled in mode 3
- roll/pitch P/I `0.135`, D `0.0036`, FLTT/FLTD `10`, SMAX `50`
- `SID_AXIS=0`
- no AutoTune messages or tuning parameter changes

The flight contained one 65.7-second armed span: Loiter, brief PosHold, then Loiter.

## Matched spectral comparison

Metrics use the central 45 airborne seconds of each flight to exclude takeoff and
landing and keep duration equal. Combined RMS is the integrated X+Y+Z Welch PSD.

| Metric | 18 Hz baseline | 25 Hz test | Result |
| --- | ---: | ---: | --- |
| Pre-filter 20-200 Hz RMS | 0.11161 rad/s | 0.11011 rad/s | Comparable input noise |
| Post-filter 20-200 Hz RMS | 0.00152 rad/s | 0.00194 rad/s | +2.12 dB, still very low |
| Measured 20-200 Hz attenuation | 37.31 dB | 35.08 dB | Pass |
| Fundamental band 40-70 Hz attenuation | 32.62 dB | 30.11 dB | Pass |
| 70-200 Hz attenuation | 44.13 dB | 39.57 dB | Pass |
| Dominant pre-filter peak | 107.52 Hz | 110.33 Hz | Similar second harmonic |

The 25 Hz post-filter 20-200 Hz residual is approximately 0.11 deg/s combined. The
2.12 dB increase is the expected cost of the higher cutoff, not evidence of poor
filtering.

## Phase benefit

Calculated with ArduPilot's exact `LowPassFilter2p` coefficients at the measured
1923.1 Hz raw gyro sample rate:

| Frequency | 18 Hz phase | 25 Hz phase | Phase recovered |
| ---: | ---: | ---: | ---: |
| 5 Hz | -23.05 deg | -16.41 deg | 6.64 deg |
| 8 Hz | -38.06 deg | -26.74 deg | 11.32 deg |
| 10 Hz | -48.64 deg | -33.94 deg | 14.70 deg |
| 15 Hz | -75.46 deg | -52.95 deg | 22.50 deg |

At 10 Hz, magnitude attenuation also improves from -0.39 dB to -0.11 dB.

## Aircraft health

| Metric, central 45 s | 18 Hz baseline | 25 Hz test |
| --- | ---: | ---: |
| Vibration vector median / p95 | 6.49 / 13.63 m/s2 | 5.03 / 7.77 m/s2 |
| IMU clipping | 0 | 0 |
| Battery current median / p95 | 9.98 / 12.15 A | 9.82 / 12.37 A |
| Minimum battery voltage | 23.23 V | 23.34 V |
| Maximum IMU temperature | 37.08 C | 34.18 C |
| Maximum ESC temperature | 40 C | 39 C |
| ESC error p95 | 0% | 0% |
| Roll/pitch minimum Dmod | 1.0 / 1.0 | 1.0 / 1.0 |

D activity and controller output were lower on July 19 because the flight was calmer
and contained less commanded motion. That is reassuring for safety but is not a
matched PID-response comparison.

## Limitation

The 25 Hz flight had only one qualifying pitch step and no qualifying roll step.
Its step-response metrics cannot decide whether the PID tune improved. The next
flight must keep this frozen filter and stock gains for the complete log while
performing repeated, separated roll and pitch reference movements.

After securing this log, restore `INS_RAW_LOG_OPT=0` using
`certification/parameters/0713-filter-logging-off.param` before the PID-reference
flight. Raw logging is no longer needed and creates unnecessarily large logs.