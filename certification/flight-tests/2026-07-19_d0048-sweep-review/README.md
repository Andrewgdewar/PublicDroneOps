# Scout-15 D 0.0048 Sweep Review at P/I 0.150

**Date:** 2026-07-19  
**Log:** `2026-07-19 14-01-16.bin`  
**Baseline:** `2026-07-19 13-50-30.bin` at D `0.0045`

## Configuration

- roll and pitch P/I fixed at `0.150`
- roll and pitch D `0.0048`
- gyro LPF 25 Hz, harmonic notch enabled
- roll block 53-77 s
- pitch block 77-98 s
- no sortie, AutoTune, SystemID or in-flight gain change

## Decisions

- **Roll:** D `0.0048` is a mixed result. It reduces upper ringing severity and
  6-10 Hz error but worsens median overshoot and small-command response. Test the
  midpoint D `0.00465` at fixed P/I.
- **Pitch:** restore D `0.0045`. D `0.0048` lowers median ring count but worsens
  control-band error, D noise and mean transfer authority.

## Roll comparison

| Metric | D 0.0045 | D 0.0048 |
| --- | ---: | ---: |
| Filtered-target peak / tail | 0.87 / 0.79 | 0.89 / 0.84 |
| Median overshoot | 35.4% | 51.4% |
| Overshoot p75 | 44.9% | 64.8% |
| Median ring | 1.5 | 2.0 |
| Ring p75 | 5.25 | 3.0 |
| 3-6 Hz error RMS | 2.68 deg/s | 2.82 deg/s |
| 6-10 Hz error RMS | 2.59 deg/s | 1.68 deg/s |
| Dominant ring peak | 6.14 Hz, 10.9 dB | 4.67 Hz, 8.3 dB |
| D RMS, 20-100 Hz | 0.000680 | 0.000648 |

The metrics do not justify selecting either endpoint as a clear optimum. D `0.00465`
is a targeted interpolation point, not a claimed mathematical optimum.

## Pitch comparison

| Metric | D 0.0045 | D 0.0048 |
| --- | ---: | ---: |
| Filtered-target peak / tail | 0.75 / 0.71 | 0.67 / 0.62 |
| Median overshoot | 31.4% | 30.7% |
| Median ring | 1.5 | 0.0 |
| Ring p75 | 3.75 | 2.25 |
| 3-6 Hz error RMS | 1.26 deg/s | 2.19 deg/s |
| 6-10 Hz error RMS | 1.35 deg/s | 1.51 deg/s |
| D RMS, 20-100 Hz | 0.000611 | 0.000717 |

The spectral and authority costs outweigh the event ring-count improvement. Pitch D
`0.0045` is the preferred current point at P/I `0.150`.

## Health

- vibration median/p95/max 7.26/16.64/30.32 m/s2
- zero clipping on both IMUs
- current median/p95/max 9.93/13.55/18.37 A
- minimum voltage 23.04 V
- maximum speed 11.0 m/s
- maximum IMU temperature 42.4 C
- maximum ESC temperature 39 C

All four ESC error fields rose together, with p95 about 8-9% and maxima about 73-77%.
The simultaneous pattern indicates a shared bidirectional-DShot telemetry disturbance,
not four independent motor faults. Monitor it in the next flight; do not ignore it if
the elevated p95 persists.