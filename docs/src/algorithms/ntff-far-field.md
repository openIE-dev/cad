# FDTD Near-to-Far-Field (NTFF)

> Balanis — *Antenna Theory: Analysis and Design*, 4th ed., §6.7
> (Wiley 2016).

`cad-sim::antenna` ships the **only MIT/Apache implementation of
the Huygens-equivalence Near-to-Far-Field transform** in the
open-source ecosystem. Every existing OSS FDTD with a built-in
NTFF is GPL:

- **openEMS** — GPL-3.0
- **MEEP** (NanoComp / MIT) — GPL-2.0
- **gprMax** — GPL-3.0-or-later

`cad-sim::antenna::ntff_far_field_at` re-derives the integral from
textbook Balanis so openie-cad's MIT/Apache posture stays clean.

## The problem

A finite-difference time-domain simulation gives you E and H fields
*everywhere on the Yee grid*, but radiation patterns are properties
of the **far field** — the field at distances large compared to the
radiator and the wavelength. Computing the far field by extending
the FDTD grid out to many wavelengths is wasteful: most of that
volume is empty propagation.

The Huygens equivalence principle reduces the problem to a
**closed surface integral**: pick any closed surface enclosing the
radiating structure, record the tangential E and H fields on that
surface, and the far-field anywhere outside is determined.

## The transform

Define equivalent surface currents on the Huygens surface:

```text
J = n̂ × H        (electric surface current, A/m)
M = −n̂ × E      (magnetic surface current, V/m)
```

The far-field at observation direction `r̂(θ, φ)` is:

```text
E_θ(θ, φ) ∝ Σᵢ [Jᵢ·θ̂ + (1/η) Mᵢ × r̂·θ̂] · Aᵢ · e^{jk r̂·rᵢ}
E_φ(θ, φ) ∝ Σᵢ [Jᵢ·φ̂ + (1/η) Mᵢ × r̂·φ̂] · Aᵢ · e^{jk r̂·rᵢ}
```

with the leading factor `−jωμ / 4π`.

The phase factor `e^{jk r̂·rᵢ}` is the only direction-dependent
piece; everything else is precomputed per sample. The whole
transform is a matrix-vector multiply.

## openie-cad implementation

| Component | Wave |
|---|---|
| `SurfaceCurrentSample` data shape | 69 |
| `ntff_far_field_at(samples, θ, φ, λ)` integral | 69 |
| `ntff_radiation_pattern(samples, f, n_θ, n_φ)` sweep | 69 |
| `HuygensSurface::new_box(box_idx, grid, freq)` | 69.B |
| `HuygensSurface::step_sample(grid, t)` running DFT | 69.B |
| `HuygensSurface::extract_currents()` to SurfaceCurrentSample | 69.B |

The Wave 69 substrate tests verify:

- A single Hertzian dipole sample reproduces the analytic sin θ
  pattern shape with consecutive-angle ratios within 10⁻⁶ of the
  expected sin θ ratio.
- Two unit elements at z = ±λ/4 in phase show the classic broadside
  pattern (|E_θ| at θ = 90° dominates near-axis values by >5×).
- The pattern sweep peak lands near θ = 90° (broadside) for a
  z-oriented dipole, gain normalised to 0 dB at peak.

The Wave 69.B substrate tests verify:

- A box surface `(2..=5)³` emits 96 cells with correct ±unit
  outward normals; opposing-face normals sum to zero.
- A static Ex = 1 V/m field, sampled at t = 0, accumulates exactly
  `(dt, 0)` in the DFT phasor — pure cosine projection.
- At t = T/4 the same field accumulates `(0, −dt)` — quarter-period
  phase rotation correct.
- End-to-end: an FDTD grid with non-trivial Ez / Hy fields, sampled
  once, produces finite E_θ / E_φ when fed to the NTFF integral.

## Calling it

The NTFF integral is a library function. There's no MCP tool yet —
Wave 69.C will add one. For now, hosts that want antenna patterns
run cad-sim FDTD locally and call the NTFF function in-process.

## Reference

Balanis — *Antenna Theory: Analysis and Design*, 4th ed., §6.7
(Wiley 2016). The Huygens-equivalence integral form used here.
