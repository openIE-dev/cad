# Changelog

Curated wave-by-wave history. The full git log is in the private
substrate repo; only user-facing changes appear here.

## June 2026 — the "aether-t buildability" series

A real-world RF mesh radio design (the **aether-t** board: 65×56.5 mm
6-layer with 8 wireless radios + PCIe Gen 3 + DDR + 60 GHz radar)
exposed every weak spot in the open-source PCB toolchain. Waves 62
through 70 closed each named failure mode in turn.

### Wave 70 — CNC G-code (June 3)

- New `cad_cam5::operations::pocket_zigzag_rect` — 2.5D pocket
  clearing with cutter-radius inset.
- MCP tool `generate_pocket_gcode` returns RS-274 G-code via the
  existing RTCP post-processor.

### Wave 69 / 69.B — FDTD → NTFF antenna validation (June 3)

- `cad_sim::antenna::ntff_far_field_at` — textbook Balanis §6.7
  Huygens-equivalence integral.
- `HuygensSurface::new_box / step_sample / extract_currents` — the
  bridge between the existing FDTD engine and the NTFF math.
- **The only MIT/Apache FDTD→NTFF antenna pattern code in the
  open-source ecosystem** (openEMS, MEEP, gprMax are all GPL).

### Wave 68 — vendor footprint ingest (June 3)

- `cad_format::kicad::parse_kicad_mod` + `cad_interop::kicad_mod_to_eda_footprint`.
- MCP tool `place_vendor_footprint` — closes the
  "140 dangling pads from approximate footprint stubs" failure mode
  by ingesting vendor-validated `.kicad_mod` library files verbatim.

### Wave 67.A / 67.B — partitioned multi-physics coupling (June 3)

- `cad_sim::coupling::couple` with `CouplingScheme::{Explicit,Implicit}Serial`.
- `RelaxationMode::{Constant(ω), Aitken, IqnIls{history_max}}` —
  including the **preCICE workhorse IQN-ILS quasi-Newton**
  acceleration.
- MCP tool `simulate_coupled` — runs the canonical linear coupling
  toy through any scheme + relaxation, reports convergence stats.
- **The only MIT/Apache partitioned-coupling implementation**
  (preCICE itself is LGPLv3, blocking direct use in MIT/Apache codebases).

### Wave 66 — force-directed placement (June 3)

- `cad_router::placement::force_directed_place` — Fruchterman-Reingold
  iterations with bounds clamping and locked-component support.
- MCP tool `run_placement` — replaces aether-t's hand-coded
  800-line `auto_place.py`.

### Wave 65 — Djordjevic-Sarkar + Cannonball-Huray 60 GHz (June 2)

- `cad_sim::signal::DielectricMaterial` with `dk_at(f)` / `tand_at(f)`
  — causal wideband Debye dielectric (Shlepnev 2015 anchor at a
  single datasheet point).
- `ConductorRoughness::roughness_multiplier(f)` — simplified
  Cannonball-Huray (Simonovich 2015) form: K_SR = 1 + Rz/(Rz + 2δ).
- `microstrip_impedance_at_frequency` composes Hammerstad-Jensen +
  D-S + C-H for 60 GHz-accurate impedance + loss.
- MCP tool `check_impedance_at_frequency`.

### Wave 64.A–D — length matching (June 2)

- Group-aware length matching with consensus-target picking.
- Obstacle-aware meander amplitude clamping.
- Capacity-aware segment selection (highest clearance × length).
- Diff-pair coupled tuning (`tune_diff_pair_coupled`).
- Run-aware splicing fixes the A*-output catastrophic overshoot.
- MCP `run_autorouter(length_match_groups=[...])`.

### Wave 63.A–F — Kong/Yan/Ozdal LCIS BGA escape (June 2)

- Full Kong/Yan/Ozdal ICCAD 2007 stack: feasibility predicate → LCIS
  layer assignment → Track + Via emission → ViaOffsetExterior /
  ViaInPad strategies → Autorouter integration → MCP surface.
- See [the algorithm theory](./algorithms/lcis-bga-escape.md).
- MCP `run_autorouter(bgas=[...])` + `check_bga_escape_feasibility`.

### Wave 62 — length-match meander insertion (June 2)

- `length_tuning::tune_route_to_length` — picks the longest single-layer
  segment, inserts a serpentine via the existing `generate_meander`.
- MCP `run_autorouter(length_targets=[...])`.

## Earlier waves

Waves 1–61 (substrate bootstrap, ECAD/CAD/sim domains, MCP server,
KiCanvas viewer, etc.) are documented in the private repo's commit
history; user-visible additions roll up as they become callable from
MCP.
