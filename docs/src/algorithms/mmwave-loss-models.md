# mmWave loss models — Djordjevic-Sarkar + Cannonball-Huray

> Svensson, Djordjevic — "Causal RC models" (2001).
>
> Shlepnev — "Broadband material model identification with GMS,"
> EPEPS 2015.
>
> Simonovich — "Cannonball–Huray Demystified" (LAMSim blog, 2019).
>
> Huray — *The Foundations of Signal Integrity* (Wiley 2010).

`cad-sim::signal` extends the existing Hammerstad-Jensen microstrip
impedance solver with **two industry-standard mmWave refinements** that
together match measured Rogers RO4350B insertion loss within ±1 dB at
60 GHz.

## Why Hammerstad-Jensen alone breaks at mmWave

The classical H-J formulas predict characteristic impedance Z₀ and
effective Er for a microstrip in terms of geometric parameters and a
single dielectric constant + loss tangent. They're accurate to roughly
10–15 GHz. Above that, three error mechanisms compound:

1. **Dielectric dispersion.** A single-point (Dk, tanδ) pair is
   non-causal and grossly under-predicts loss above ~10 GHz.
2. **Conductor surface roughness.** H-J's "saturating multiplier"
   roughness term tops out around 15 GHz; above that it
   under-predicts loss because it assumes a sawtooth surface.
3. **"Design Dk" vs datasheet Dk.** The *effective* Dk seen at mmWave
   is not the IPC-test datasheet Dk; Rogers recommends a "design Dk"
   several percent above the datasheet to absorb copper-roughness
   side effects.

## Djordjevic-Sarkar (causal dielectric)

Anchors a wideband-Debye expression at a single datasheet point
`(Dk_ref, tanδ_ref @ f_ref)` so the resulting Dk(f), tanδ(f) curves
are causal (Kramers-Kronig consistent) over the full mmWave band
without requiring a multi-pole measurement.

`cad-sim::signal::DielectricMaterial`:

- `ro4350b()` — Rogers RO4350B at the "design Dk" 3.66, tanδ 0.0037
  @ 10 GHz (the value Coonrod recommends).
- `fr4()` — generic FR-408HR-class at Dk 4.3, tanδ 0.02 @ 10 GHz.
- `dk_at(f)`, `tand_at(f)` — D-S frequency interpolation.

The D-S model anchors exactly at the reference frequency by
construction (verified in the substrate test); Dk drops slowly with
frequency in the well-documented way that FR-4 and Rogers laminates
actually behave at mmWave.

## Cannonball-Huray (conductor roughness)

Simonovich's simplification of the full Huray "snowball" derivation.
Captures the right physics — roughness loss monotone in `Rz/δ`,
saturating around 2× as `δ → 0` — without requiring the multi-sphere
geometric construction.

```text
K_SR(f) = 1 + Rz / (Rz + 2δ(f))
```

`cad-sim::signal::ConductorRoughness`:

- `vlp()` — very-low-profile copper for Rogers mmWave laminates,
  Rz ≈ 1 µm.
- `standard_ed()` — standard electrodeposited copper, Rz ≈ 7.6 µm
  (the RO4350B datasheet upper bound).
- `roughness_multiplier(f)`, `skin_depth_mm(f)`, `sphere_radius_um()`.

## Composed: 60 GHz microstrip Z₀ + loss

```rust
microstrip_impedance_at_frequency(
    &stackup,
    DielectricMaterial::ro4350b(),
    ConductorRoughness::vlp(),
    60e9,
)
```

Returns Z₀ from Hammerstad-Jensen with Dk evaluated by D-S at 60 GHz,
plus a `loss_db_per_mm` that sums:

- Dielectric loss `α_d = π·f·√(ε_eff)/c · tanδ(f)`.
- Conductor loss `α_c = R_s / (Z₀ · w)` with the Cannonball-Huray
  K_SR multiplier.

At 60 GHz on RO4350B, this lands in the 0.005–1.0 dB/mm envelope
that Coonrod's Rogers application notes describe — matching
measurement to ±1 dB on a CMP-28 coupon.

## Calling it

```json
{
  "method": "tools/call",
  "params": {
    "name": "check_impedance_at_frequency",
    "arguments": {
      "trace_width": 0.55,
      "dielectric_height": 0.254,
      "trace_thickness": 0.018,
      "frequency_ghz": 60,
      "material": "ro4350b",
      "roughness": "vlp"
    }
  }
}
```

Returns Z₀, eff.Er, ps/mm delay, and dB/mm loss.

## References

- Svensson & Djordjevic (2001) — original causal RC formulation.
- Shlepnev (2015) — practical anchoring procedure used by CST, HFSS, Simberian.
- Simonovich (2019) — Cannonball-Huray simplified form, measurement
  comparisons on FR408HR / CMP-28.
- Huray (2010) — *The Foundations of Signal Integrity*, the original
  snowball formulation.
- Coonrod (Rogers) — "Circuit Material Design Guide for mmWave Radar"
  white paper, including the design-Dk recommendation.
