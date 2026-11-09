# Week 1 Multisim Checks

Hand calculation vs NI Multisim simulation, week of Nov 9, 2026.

All values are RMS phasors. Hand calculations were done first; the Multisim column is filled in from the simulation.

## Series RLC: Boylestad Prob. 15.29 (Fig. 15.101)
R = 30 Ω, L = 20 mH, C = 39 µF, e = √2(20) sin(1000t + 40°) V → E = 20 V ∠40°, f = 159.15 Hz

| Quantity | Hand | Multisim | % error |
| --- | --- | --- | --- |
| Zₜ | 30.53 Ω ∠−10.65° | | |
| I | 0.655 A ∠50.65° | | |
| Vᵣ | 19.66 V ∠50.65° | | |
| Vₗ | 13.10 V ∠140.65° | | |
| V꜀ | 16.80 V ∠−39.35° | | |
| P | 12.88 W | | |
| f₀ = 1/(2π√(LC)) | 180.2 Hz (current peak in AC sweep) | | |

## Parallel RLC: Boylestad Prob. 16.13 (Fig. 16.71)
R = 3 kΩ ∥ L = 3.9 H ∥ C = 0.12 µF, iₛ = 5 sin(1000t − 20°) mA → Iₛ = 3.536 mA ∠−20°, f = 159.15 Hz

| Quantity | Hand | Multisim | % error |
| --- | --- | --- | --- |
| Zₜ | 2777 Ω ∠22.26° | | |
| V | 9.816 V ∠2.26° | | |
| Iᵣ | 3.272 mA ∠2.26° | | |
| Iₗ | 2.517 mA ∠−87.74° | | |
| I꜀ | 1.178 mA ∠92.26° | | |
| f₀ = 1/(2π√(LC)) | 232.6 Hz (voltage peak / impedance maximum in AC sweep) | | |
