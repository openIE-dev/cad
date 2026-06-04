# Antenna pattern from FDTD

Wave 69 + 69.B closed the loop between cad-sim's existing 3D FDTD
solver and the Huygens-equivalence Near-to-Far-Field transform —
the **only MIT/Apache FDTD → NTFF antenna pattern code in the
open-source ecosystem**.

See the [NTFF algorithm page](../algorithms/ntff-far-field.md) for the
math.

## What you need

The current Wave 69 pipeline is a Rust library — there's no MCP tool
yet (Wave 69.C will add `run_fdtd_radiation_pattern`). The library
sequence is:

1. Configure the FDTD grid (`FdtdConfig`, `FdtdGrid`).
2. Place a source — `FdtdSource::GaussianPulse` at the radiating
   structure's feed point.
3. Define a Huygens surface enclosing the structure:
   `HuygensSurface::new_box(box_idx, &grid, frequency_hz)`.
4. Time-step the simulation. On every step, after `update_e` and
   `update_h` and any source application:
   ```rust
   grid.update_h();
   grid.apply_source(&source);
   grid.update_e();
   surface.step_sample(&grid, t_now);
   ```
5. When stepping completes, convert to NTFF input:
   `let samples = surface.extract_currents();`
6. Compute the radiation pattern:
   `let result = ntff_radiation_pattern(&samples, freq, n_theta, n_phi);`

The `FarFieldResult` exposes per-(θ, φ) gain in dBi, |E_θ|, |E_φ|,
peak direction, etc.

## Validating against analytic patterns

The Wave 69 tests show this loop reproduces:

- **Hertzian dipole** sin θ pattern shape to ratios within 10⁻⁶ at
  9 sample angles.
- **Two-element broadside array** at z = ±λ/4 in phase — the
  classic main-lobe at θ = 90° dominating the near-axis values
  by ≥ 5×.
- **Pattern sweep peak** lands within 10° of θ = 90° (broadside)
  for a z-oriented dipole.

The Wave 69.B Huygens-surface sampling tests verify:

- A static Ex = 1 V/m sampled at t = 0 accumulates exactly `(dt, 0)`
  in the running DFT phasor (pure cosine projection).
- The same field at t = T/4 accumulates exactly `(0, −dt)`
  (quarter-period phase rotation correct).

## What's coming (Wave 69.C)

`run_fdtd_radiation_pattern(geometry, frequency_hz, n_theta,
n_phi, grid_steps)` MCP tool will encapsulate the whole loop so
agents can compute patterns over the network. The current path
requires the substrate running in-process.

## When to bother

For boards using antenna-in-package parts (the Infineon BGT60TR13C
60 GHz radar in aether-t, for example), the antenna sits inside
the chip — the PCB carries no RF trace to it and no NTFF is needed.

For boards with on-PCB antennas (LoRa monopoles, GNSS patches,
Wi-Fi PIFAs, mmWave horns), NTFF validates the radiation pattern
before fabrication. The substrate's NTFF is matched against
analytic shapes at lib-test time; for novel geometries, run an
external reference (HFSS / openEMS / CST) once and use the
substrate's result as the *go-to* for design-iteration speed.
