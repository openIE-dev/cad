# MCP Tool Reference

Every tool the openie-cad MCP server at `https://cad.openie.dev/mcp` exposes.
Auto-generated from the substrate source on each release.

Call any tool by sending an MCP `tools/call` request with the tool's `name`
and the documented arguments. The session ID returned from the first call
keeps the project in memory for subsequent calls — see the
[Quickstart](./quickstart.md) for the round-trip pattern.

---

_55 tools registered._

---

## `add_track`

Add a copper track between points

**Arguments**

- `net` *(string)*
- `width` *(number)*
- `layer` *(string)*

---

## `add_via`

Add a via at a position

**Arguments**

- `x` *(number)*
- `y` *(number)*
- `drill` *(number)*
- `net` *(string)*

---

## `check_bga_escape_feasibility`

Wave 63.A — answer the 'can this BGA escape at all on this fab tier?' \ question *before* the autorouter chews through 100+ net pairs only to \ fail. Models the BGA as a rows×cols pad grid; computes via geometry \ from the fab tier (Standard JLCPCB = 0.45mm via pad, HDI tier = 0.25mm \ microvia); reports whether vias fit between adjacent BGA pads at all, \ and whether the perimeter has enough channel capacity for the \ assigned inner-ring nets. Args: `rows` (int), `cols` (int), \ `pitch_mm` (number, e.g. 0.4 for LMS7002M aQFN), `pad_diameter_mm` \ (number, the BGA's own pad diameter, vendor-defined), `fab_tier` \ ("standard" | "hdi", default standard), `assigned_inner_nets` \ (int, count of inner pads needing escape, default 0).

**Arguments**

- `rows` *(integer)*
- `cols` *(integer)*
- `pitch_mm` *(number)*
- `pad_diameter_mm` *(number)*
- `fab_tier` *(string)*
- `assigned_inner_nets` *(integer)*

---

## `check_diff_pair`

Calculate differential pair impedance

**Arguments**

- `trace_width` *(number)*
- `gap` *(number)*
- `dielectric_height` *(number)*

---

## `check_impedance`

Calculate microstrip trace impedance

**Arguments**

- `trace_width` *(number)*
- `dielectric_height` *(number)*
- `er` *(number)*

---

## `check_impedance_at_frequency`

Wave 65 — microstrip impedance + loss at mmWave (>10 GHz) with \ Djordjevic-Sarkar causal dielectric model and Cannonball-Huray \ conductor-roughness multiplier. Replaces single-point loss tangent \ (good only to ~10 GHz) with frequency-anchored material + roughness \ anchored to a single datasheet (Dk, tanδ) point. Args: \ `trace_width` mm (default 0.55), `dielectric_height` mm (default 0.254), \ `trace_thickness` mm (default 0.018), `frequency_ghz` (default 60), \ `material` ("ro4350b"|"fr4", default ro4350b), `roughness` \ ("vlp"|"std_ed", default vlp).

**Arguments**

- `trace_width` *(number)*
- `dielectric_height` *(number)*
- `trace_thickness` *(number)*
- `frequency_ghz` *(number)*
- `material` *(string)*
- `roughness` *(string)*

---

## `create_board`

Create a PCB board with dimensions

**Arguments**

- `name` *(string)*
- `width` *(number)*
- `height` *(number)*
- `layers` *(number)*

---

## `create_box`

Create a box with width, height, depth in mm

**Arguments**

- `width` *(number)*
- `height` *(number)*
- `depth` *(number)*

---

## `create_cylinder`

Create a cylinder with radius and height in mm

**Arguments**

- `radius` *(number)*
- `height` *(number)*

---

## `deepcad_forward`

Wave 74.B — pure-Rust forward pass through the DeepCAD command-\ sequence decoder (Wu et al. ICCV 2021), implemented in Candle. \ Substrate-side architecture lives in cad-deepcad: 6-token vocab, \ 256-d embedding, 4-layer transformer, 256-bin arg quantization \ matching the paper. Weights are randomly initialized in Wave 74.B; \ Wave 74.C will swap in the MIT-licensed DeepCAD .pt → safetensors \ path so this tool produces real image-conditioned CAD command \ sequences. The whole inference path stays single-binary Rust — no \ Python, no PyTorch runtime, no Torch C++ linking. Wave 74.C wires \ weight loading from `.safetensors` and PyTorch `.pt` files (both \ via Candle's pure-Rust loaders). Args: `batch` (int, default 1), \ `seq_len` (int, default 8), `weights_path` (string, optional — \ file path; .safetensors uses mmap, .pt/.pth uses pickle reader; \ omit for random init), `state_key` (string, optional — for nested \ .pt files like `{"model": <state_dict>}`).

**Arguments**

- `batch` *(integer)*
- `seq_len` *(integer)*
- `weights_path` *(string)*
- `state_key` *(string)*

---

## `delete_body`

Delete a body by its UUID

**Arguments**

- `body_id` *(string)*

---

## `design_lqr`

Wave 75.C — design a discrete-time infinite-horizon Linear-\ Quadratic Regulator via iterated DARE (Discrete Algebraic Riccati \ Equation). For x_{k+1} = A x + B u and cost Σ (x^T Q x + u^T R u), \ returns the optimal feedback gain K such that u = -K x is the \ stabilising optimal policy. Closed-loop spectral radius is \ reported; values strictly under 1 mean (A, B) is stabilisable with \ the chosen Q, R. Args: `a`, `b`, `q`, `r` (matrices, required), \ `max_iter` (int, default 500), `tol` (number, default 1e-10).

**Arguments**

- `a` *(array)*
- `b` *(array)*
- `q` *(array)*
- `r` *(array)*
- `max_iter` *(integer)*
- `tol` *(number)*

---

## `estimate_cost`

Estimate manufacturing cost

**Arguments**

- `quantity` *(number)*
- `fab` *(string)*

---

## `eval_ocl`

Parse and evaluate OCL, producing geometry

**Arguments**

- `source` *(string)*

---

## `export_brep`

Export bodies as OpenCascade BREP (polyhedral subset — \ each triangle becomes a planar Face. Importable by FreeCAD / \ Salome / OCCT-CLI for viz or further Boolean ops; not a \ substitute for full surface BREP — for that use IGES / STEP).

**No arguments.**

---

## `export_gerber`

Export Gerber fabrication files

**No arguments.**

---

## `export_gltf`

Export bodies as glTF 2.0

**No arguments.**

---

## `export_obj`

Export bodies as OBJ

**No arguments.**

---

## `export_ply`

Export bodies as Stanford PLY (ASCII)

**No arguments.**

---

## `export_stl`

Export bodies as STL binary

**No arguments.**

---

## `export_xao`

Export bodies as Salome XAO (BREP payload + topology + groups). \ Argument `name` overrides the geometry name (defaults to project.name).

**Arguments**

- `name` *(string)*

---

## `generate_bom`

Generate Bill of Materials

**No arguments.**

---

## `generate_contour_gcode`

Wave 70.B — follow an arbitrary 2D path at successive Z depths. \ Caller supplies `path_mm` (array of [x,y] waypoints in mm). At each \ Z level the tool follows the path in order; `closed: true` adds a \ closing move back to the first waypoint (perimeter milling). No \ cutter compensation — caller offsets the path by the cutter radius. \ Args: `path_mm` (required, array of [x,y]), `depth_mm` (default 5), \ `step_down_mm` (default 1), `closed` (default true), \ `feedrate_mm_min` (default 800), `plunge_rate_mm_min` (default 200), \ `spindle_rpm` (default 10000), `safe_z_mm` (default 5).

**Arguments**

- `path_mm` *(array)*
- `depth_mm` *(number)*
- `step_down_mm` *(number)*
- `closed` *(boolean)*
- `feedrate_mm_min` *(number)*
- `plunge_rate_mm_min` *(number)*
- `spindle_rpm` *(number)*
- `safe_z_mm` *(number)*

---

## `generate_drill_gcode`

Wave 70.B — drill a list of holes. Args: `positions_mm` (required, \ array of [x,y]), `depth_mm` (default 5), `peck_increment_mm` \ (optional; when set, enables G83-style peck drilling with full \ retract between pecks), `plunge_rate_mm_min` (default 200), \ `spindle_rpm` (default 10000), `safe_z_mm` (default 5).

**Arguments**

- `positions_mm` *(array)*
- `depth_mm` *(number)*
- `peck_increment_mm` *(number)*
- `plunge_rate_mm_min` *(number)*
- `spindle_rpm` *(number)*
- `safe_z_mm` *(number)*

---

## `generate_firmware`

Generate firmware skeleton code

**Arguments**

- `target` *(string)*

---

## `generate_jlcpcb_bom`

Generate a JLCPCB-format BOM CSV from project.eda.pcb (columns: \ Comment, Designator, Footprint, LCSC Part #). Designators sharing \ (value, footprint, LCSC) collapse into one row. Returns the CSV \ inline. Requires a PCB — run import_zener_with_layout / \ import_kicad first.

**No arguments.**

---

## `generate_jlcpcb_cpl`

Generate a JLCPCB-format CPL (component placement list) CSV from \ project.eda.pcb (columns: Designator, Mid X, Mid Y, Layer, \ Rotation). Coordinates in mm to 4 decimals; F.Cu / B.Cu mapped \ to Top / Bottom. Returns the CSV inline. Requires a PCB.

**No arguments.**

---

## `generate_pocket_gcode`

Wave 70 — emit ready-to-run CNC G-code for a rectangular pocket \ zig-zag toolpath. Picks safe Z plane → plunges in step-down \ increments → sweeps zig-zag rows at each Z level → retracts \ between Z levels. Cutter-radius inset is applied automatically so \ the pocket walls match the requested width × height exactly. The \ generated program is RS-274 (Fanuc-style) via cad-cam5's RTCP \ post. Args: `width_mm` (default 20), `height_mm` (default 20), \ `depth_mm` (default 5), `cutter_diameter_mm` (default 3), \ `step_down_mm` (default 1), `step_over_mm` (default cutter/2), \ `feedrate_mm_min` (default 800), `plunge_rate_mm_min` (default \ 200), `spindle_rpm` (default 10000), `safe_z_mm` (default 5), \ `x_mm`/`y_mm` (pocket origin, default 0).

**Arguments**

- `width_mm` *(number)*
- `height_mm` *(number)*
- `depth_mm` *(number)*
- `cutter_diameter_mm` *(number)*
- `step_down_mm` *(number)*
- `step_over_mm` *(number)*
- `feedrate_mm_min` *(number)*
- `plunge_rate_mm_min` *(number)*
- `spindle_rpm` *(number)*
- `safe_z_mm` *(number)*
- `x_mm` *(number)*
- `y_mm` *(number)*

---

## `import_atopile`

Import an atopile (.ato) source string. atopile is the Python-flavored \ text-HDL from atopile.io; this tool runs the cad-interop atopile bridge \ which extracts module declarations, `new` instantiations, `~` connections, \ and parameter assignments into a UDD. Returns a summary; UDD is generated \ but not yet lowered into project.eda (follow-on).

**Arguments**

- `source` *(string)*
- `name` *(string)*

---

## `import_atopile_with_layout`

Atopile-pipeline completer: imports a `.ato` file's schematic + netlist \ AND overlays a companion `.kicad_pcb` file's placements + traces in one \ call. After this, the session's project has a real routed board — DRC \ finds real violations, generate_jlcpcb_bom/cpl + export_gerber produce \ fab-ready output. Use this whenever you have both the atopile source and \ the KiCad layout that `ato build` emitted.

**Arguments**

- `atopile_source` *(string)*
- `kicad_pcb_source` *(string)*
- `name` *(string)*

---

## `import_deepcad`

Wave 72 — ingest a DeepCAD command-sequence JSON (Wu et al. ICCV \ 2021; output format of GenCAD, arXiv 2409.16294 MIT DECODE Lab). \ Pure-Rust parser + lifter: no Python / PyTorch / OpenCascade runtime, \ no ML bloat in the eval path. Each `SOL..EOS..Ext` operation block is \ lifted to a cad-kernel primitive (rectangular sketches → box, \ circular sketches → cylinder) and the Wave 72 boolean op \ (NewBody / Join / Cut / Intersect) is applied against the prior body. \ Args: `json` (string, required, the DeepCAD command-sequence JSON).

**Arguments**

- `json` *(string)*

---

## `import_kcl`

Import a KCL (Zoo.dev) source file

**Arguments**

- `source` *(string)*

---

## `import_kicad`

Import a KiCad project — accepts `pcb_source` (.kicad_pcb contents) and/or \ `schematic_source` (.kicad_sch contents). Replaces project.eda with the \ result. To layer a layout on top of an existing schematic (e.g. one from \ a prior import_zener call), use `import_zener_with_layout` instead.

**Arguments**

- `name` *(string)*
- `pcb_source` *(string)*
- `schematic_source` *(string)*

---

## `import_tscircuit`

Import a tscircuit Circuit JSON document. tscircuit (tscircuit.com) is a \ React/JSX DSL for PCB; you write JSX, the TS runtime emits Circuit JSON. \ This tool consumes that JSON (skipping the JSX eval) and lifts source_* \ elements into a UDD. Returns a summary; UDD is generated but not yet \ lowered into project.eda (follow-on).

**Arguments**

- `source` *(string)*
- `name` *(string)*

---

## `import_zener`

Import a Zener (Diode Computers' Starlark PCB-as-code) source string. \ Populates the project's EDA schematic + netlist with the parsed \ components, nets, and pin connections; if the Zener Board(...) call \ is present, also creates a placeholder Pcb with the declared layer \ count (board width/height default to 50x50mm — actual layout lives \ in the KiCad file at Board(layout_path=) and arrives via a follow-up \ import). Errors carry line numbers from the Zener parser.

**Arguments**

- `source` *(string)*

---

## `import_zener_with_layout`

Zener-pipeline completer: imports a `.zen` file's schematic + netlist AND \ overlays a `.kicad_pcb` file's placements + traces in one call. After \ this, the session's project has a real routed board — DRC finds real \ violations, `export_gerber` produces fab-ready output. Use this whenever \ you have both the Zener source and the KiCad layout it `layout_path=`s.

**Arguments**

- `zener_source` *(string)*
- `kicad_pcb_source` *(string)*
- `name` *(string)*

---

## `kalman_step`

Wave 75.B — single predict+update step of a linear Kalman filter \ for x_{k+1} = F x + B u + w, y = H x + v with w ~ N(0, Q), \ v ~ N(0, R). Stateless from the caller's POV: pass in current \ (x, P, u, y) and the matrices (F, B, H, Q, R); receive back the \ new posterior mean, covariance, and innovation. B and u may be \ omitted for autonomous systems. Args: `f`, `h`, `q`, `r`, `p` \ (matrices, required), `x`, `y` (vectors, required), `b` (matrix, \ optional), `u` (vector, optional).

**Arguments**

- `f` *(array)*
- `h` *(array)*
- `q` *(array)*
- `r` *(array)*
- `p` *(array)*
- `x` *(array)*
- `y` *(array)*
- `b` *(array)*
- `u` *(array)*

---

## `parse_ocl`

Parse an OCL source string

**Arguments**

- `source` *(string)*

---

## `place_footprint`

Place a component footprint on the PCB

**Arguments**

- `reference` *(string)*
- `lib_id` *(string)*
- `x` *(number)*
- `y` *(number)*

---

## `place_vendor_footprint`

Wave 68 — place a vendor-validated footprint into project.eda.pcb \ from a standalone `.kicad_mod` source string. Pad positions, sizes, \ shapes, and drill come from the parsed module verbatim — closing the \ aether-t "140 dangling pads from approximate footprint stubs" \ failure mode by guaranteeing the land pattern matches the vendor \ datasheet. Creates an empty 100×80 mm PCB if none exists. Args: \ `kicad_mod` (string, required, the .kicad_mod source), `reference` \ (string, default "U?"), `value` (string, optional, e.g. the MPN), \ `x_mm` (number, default 0), `y_mm` (number, default 0), \ `rotation_deg` (number, default 0).

**Arguments**

- `kicad_mod` *(string)*
- `reference` *(string)*
- `value` *(string)*
- `x_mm` *(number)*
- `y_mm` *(number)*
- `rotation_deg` *(number)*

---

## `plan_rrt_star`

Wave 79.C — 2D RRT* (Karaman & Frazzoli 2011) for global \ path planning through circular obstacles. Pure Rust; pairs \ with the bicycle MPCs — feed the returned waypoints in as \ the reference trajectory (interpolated to 4-state \ (X, Y, ψ, v)) and have solve_bicycle_mpc / _ilqr / _mppi \ track it. Asymptotically optimal: more iterations → better \ path. Reuses the same Obstacle primitive as \ solve_bicycle_mpc, so plan + control share data. \ Args: `start` (length-2 vector, required), `goal` (length-2, \ required), `bounds` (length-4 vector [x_min, x_max, y_min, \ y_max], required), `obstacles` (array of {x, y, radius, \ safety_margin?, slack_penalty?}, optional), `step_size` \ (number, default 0.5), `goal_radius` (number, default 0.3), \ `rewire_radius` (number, default 1.0), `n_iterations` (int, \ default 2000), `goal_bias` (number 0..1, default 0.1), \ `seed` (int, optional). Returns the waypoint sequence or \ an error if no path was found in `n_iterations`.

**Arguments**

- `start` *(array)*
- `goal` *(array)*
- `bounds` *(array)*
- `obstacles` *(array)*
- `step_size` *(number)*
- `goal_radius` *(number)*
- `rewire_radius` *(number)*
- `n_iterations` *(integer)*
- `goal_bias` *(number)*
- `seed` *(integer)*

---

## `render_pcb_svg`

Render project.eda.pcb as an SVG board preview — outline, copper \ tracks per layer, vias, placed footprints with reference designators. \ Returns the SVG inline so agents/users can see what was imported. \ Optional args: `scale` (px/mm, default 10), `margin_mm` (default 5), \ `show_references` (default true). Requires a PCB.

**Arguments**

- `scale` *(number)*
- `margin_mm` *(number)*
- `show_references` *(boolean)*

---

## `run_autorouter`

Route all unconnected nets on project.eda.pcb. Groups footprint \ pads by net name, decomposes each multi-pin net via SALT \ (Wave 58 shallow-light tree), and routes each tree edge with the \ cad-router A* engine — optionally using PathFinder negotiation \ (Wave 55) when `negotiation = true`. Resulting paths get lifted \ back into project.eda.pcb.tracks. Args: `epsilon` (number, SALT \ radius/wirelength knob, default 1.0); `negotiation` (boolean, \ default false); `length_targets` (Wave 62: optional array of \ `{net: string, target_mm: number, tolerance_mm: number, \ amplitude_mm?: number, spacing_mm?: number}` — each named net \ gets a serpentine meander inserted to match the target length, \ useful for DDR group-skew and PCIe diff-pair matching); \ `length_match_groups` (Wave 64.A: optional array of \ `{name: string, nets: [string, ...], tolerance_mm: number, \ target_mm?: number, amplitude_mm?: number, spacing_mm?: number}` \ — names a group of nets to be length-matched. When \ target_mm is omitted the substrate picks the longest \ routed length as the consensus target, so every \ undersize net meanders up to match. Replaces \ length_targets when the caller wants substrate- \ computed targets); \ `bgas` (Wave 63.F: optional array of \ `{footprint_ref: string, pitch_mm: number, \ pad_diameter_mm: number, fab_tier: "standard"|"hdi", \ inner_layers: ["In1Cu", ...], buses: {bus_name: [net_name, ...]}, \ strategy?: "offset"|"via_in_pad"}` — each entry \ names a BGA in the PCB by footprint reference; the \ substrate runs the Wave 63 LCIS escape stack and \ splices the resulting tracks + vias before routing \ the general fabric). Requires a PCB.

**Arguments**

- `epsilon` *(number)*
- `negotiation` *(boolean)*
- `length_targets` *(array)*
- `length_match_groups` *(array)*
- `bgas` *(array)*

---

## `run_dfm`

Run Design for Manufacturability check

**Arguments**

- `fab` *(string)*

---

## `run_drc`

Run Design Rule Check on the PCB

**No arguments.**

---

## `run_placement`

Wave 66 — force-directed component placement on project.eda.pcb. \ Models each footprint by an AABB derived from its pad layout; pulls \ net-connected footprints toward their common centroid; pushes \ overlapping footprints apart; clamps to the board's outline. \ Replaces aether-t's hand-coded auto_place.py. Args: \ `iterations` (int, default 200), `attraction` (number, spring \ strength on connected nets, default 0.001), `repulsion` (number, \ separation force on overlaps, default 10000), `step_mm` (per-\ iteration cap on movement, default 0.1), `locked` (array of \ footprint refs to pin in place). Requires a PCB with footprints.

**Arguments**

- `iterations` *(integer)*
- `attraction` *(number)*
- `repulsion` *(number)*
- `step_mm` *(number)*
- `locked` *(array)*

---

## `run_thermal`

Run thermal simulation

**Arguments**

- `ambient_c` *(number)*

---

## `simulate_coupled`

Wave 67 — partitioned multi-physics coupling primitive. Runs a \ linear coupling toy (solve_a(b) = a0 + α·b, solve_b(a) = b0 + β·a) \ through the cad_sim::coupling engine to demonstrate convergence \ behavior at different stiffness levels (αβ) and relaxation modes. \ Useful for *choosing* a coupling scheme before plugging in real \ cad-sim domains. Args: `coupling_alpha`, `coupling_beta` (stiffness, \ default 0.5/0.5), `a0`, `b0` (offsets, default 1/1), `scheme` \ ("implicit"|"explicit", default implicit), `relaxation` \ ("aitken"|"constant"|"iqn_ils", default aitken), `omega` (for \ constant), `iqn_history_max` (for IQN-ILS, default 10), \ `max_iterations` (default 200), `tolerance` (default 1e-8). \ Reports iteration count, final residual, and the analytic fixed \ point for comparison.

**Arguments**

- `coupling_alpha` *(number)*
- `coupling_beta` *(number)*
- `a0` *(number)*
- `b0` *(number)*
- `scheme` *(string)*
- `relaxation` *(string)*
- `omega` *(number)*
- `iqn_history_max` *(integer)*
- `max_iterations` *(integer)*
- `tolerance` *(number)*

---

## `solve_bicycle_ilqr`

Wave 78.A/B — bicycle trajectory optimization via iLQR \ (Iterative Linear Quadratic Regulator). Backward Riccati \ + forward line search; locally quadratic-convergent on \ smooth quadratic-cost problems — typically 5-20 outer \ iterations vs solve_bicycle_nmpc's ~200 PANOC iters. \ No box constraints in this variant (use solve_bicycle_mpc \ for hard control bounds, or solve_bicycle_nmpc for ball/box \ via PANOC projection). \ Args: `wheelbase` (m, default 0.3), `dt` (s, default 0.1), \ `horizon` (int, default 10), `q`/`r`/`qf` (matrices, \ required), `x0` (length-4 vector, required), `x_ref` \ (horizon+1 length array of 4-vectors, required), `u_ref` \ (horizon length array of 2-vectors, required), `max_iter` \ (int, default 30), `tol` (float, default 1e-6), `reg_init` \ (float, default 1e-4 — initial Tikhonov regularization on \ Q_uu; raised on backward-pass / line-search failures). \ Returns u_seq, x_seq, cost, iterations, converged.

**Arguments**

- `wheelbase` *(number)*
- `dt` *(number)*
- `horizon` *(integer)*
- `q` *(array)*
- `r` *(array)*
- `qf` *(array)*
- `x0` *(array)*
- `x_ref` *(array)*
- `u_ref` *(array)*
- `max_iter` *(integer)*
- `tol` *(number)*
- `reg_init` *(number)*

---

## `solve_bicycle_mpc`

Wave 76.C/D — solve iterative MPC for the kinematic bicycle \ with optional circular-obstacle avoidance. State x = (X, Y, \ ψ, v) in m, m, rad, m/s. Control u = (a, δ) in m/s² and rad. \ The outer loop linearizes around the nominal trajectory and \ re-solves until the control sequence converges (Σ ‖Δu_k‖ < \ tol) or max_iter is reached. Each obstacle adds a soft \ half-plane constraint per horizon step (linearized keep-out + \ slack penalty) so the solver gracefully handles initial \ nominals that cut through obstacles. \ Args: `wheelbase` (m, default 0.3), `dt` (s, default 0.1), \ `horizon` (int, default 15), `q`/`r`/`qf` (matrices, required: \ 4×4 / 2×2 / 4×4), `x0` (length-4 vector, required), `x_ref` \ (horizon+1 length array of 4-vectors, required), `u_ref` \ (horizon length array of 2-vectors, required), `u_min`/`u_max` \ (length-2 vectors, optional), `max_iter` (int, default 6), \ `tol` (float, default 1e-3), `obstacles` (array of objects \ `{x, y, radius, safety_margin?, slack_penalty?}`, optional). \ Returns u_seq, x_seq (under the true nonlinear dynamics), \ iterations, converged.

**Arguments**

- `wheelbase` *(number)*
- `dt` *(number)*
- `horizon` *(integer)*
- `q` *(array)*
- `r` *(array)*
- `qf` *(array)*
- `x0` *(array)*
- `x_ref` *(array)*
- `u_ref` *(array)*
- `u_min` *(array)*
- `u_max` *(array)*
- `max_iter` *(integer)*
- `tol` *(number)*
- `obstacles` *(array)*

---

## `solve_bicycle_mppi`

Wave 79.A — bicycle Model Predictive Path Integral (MPPI) \ control. Sampling-based MPC: at each step, draws \ `n_samples` Gaussian noise sequences over the horizon, \ rolls each through the plant, and Boltzmann-averages by \ cost. No gradients required — works with any plant. CPU- \ parallel via rayon. Use when: plant is non-smooth or \ discrete; you want robustness to local minima; or you have \ spare CPU/GPU cores to throw at it. \ Args: same plant + cost as solve_bicycle_ilqr, plus \ `n_samples` (int, default 256), `n_iterations` (int, \ default 2), `sigma` (length-2 vector, default [0.5, 0.2] \ — accel/steer noise stdev), `lambda` (number, default 1.0 \ — Boltzmann temperature: smaller = greedier), `u_min`/\ `u_max` (length-2 vectors, optional — sampled controls \ are clamped), `seed` (int, optional — pass for reproducibility).

**Arguments**

- `wheelbase` *(number)*
- `dt` *(number)*
- `horizon` *(integer)*
- `q` *(array)*
- `r` *(array)*
- `qf` *(array)*
- `x0` *(array)*
- `x_ref` *(array)*
- `u_ref` *(array)*
- `n_samples` *(integer)*
- `n_iterations` *(integer)*
- `sigma` *(array)*
- `lambda` *(number)*
- `u_min` *(array)*
- `u_max` *(array)*
- `seed` *(integer)*

---

## `solve_bicycle_nmpc`

Wave 77.B/C — bicycle Nonlinear MPC via OpEn's PANOC solver. \ Companion to solve_bicycle_mpc (which uses iterative LTV \ around a nominal via Clarabel QP). NMPC tackles the full \ nonconvex problem directly, with adjoint-method gradient \ backpropagated through the bicycle dynamics. Use this variant \ when: cost is non-quadratic, control set isn't a box (try \ `bound_kind="ball"` for a Euclidean-ball constraint), or \ dynamics are non-smooth (replace bicycle with another \ NonlinearPlant in future waves). \ Args: same as solve_bicycle_mpc, plus `bound_kind` \ ("rectangle"|"ball"|"inf_ball"|"none", default \ "rectangle"), `bound_radius` (number, used when bound_kind \ is ball/inf_ball, default 10.0 / 2.0), `max_iter` (int, \ default 300), `tol` (number, default 1e-5), `lbfgs_memory` \ (int, default 10). No obstacle field — Wave 77 NMPC \ doesn't include obstacle penalties (use solve_bicycle_mpc \ for obstacles).

**Arguments**

- `wheelbase` *(number)*
- `dt` *(number)*
- `horizon` *(integer)*
- `q` *(array)*
- `r` *(array)*
- `qf` *(array)*
- `x0` *(array)*
- `x_ref` *(array)*
- `u_ref` *(array)*
- `u_min` *(array)*
- `u_max` *(array)*
- `bound_kind` *(string)*
- `bound_radius` *(number)*
- `max_iter` *(integer)*
- `tol` *(number)*
- `lbfgs_memory` *(integer)*

---

## `solve_lti_mpc`

Wave 76.A — solve one step of a linear time-invariant MPC \ via condensed-form QP. Problem: minimize Σ (x^T Q x + u^T R u) \ + x_N^T Qf x_N s.t. x_{k+1} = A x + B u, x_0 = given, optional \ box constraints on u and x. The decision variables are the \ control sequence u_0..u_{N-1}; the predicted state trajectory \ is reconstructed. Returns the full sequence + cost + status. \ For receding-horizon use, call repeatedly with the updated x0. \ If `qf` is omitted, the infinite-horizon LQR cost-to-go P∞ for \ (A, B, Q, R) is auto-synthesized as terminal cost, so even \ short horizons inherit LQR-grade stabilising behaviour. \ Args: `a`, `b`, `q`, `r` (matrices, required), `qf` (matrix, \ optional), `horizon` (int, default 10), `x0` (vector, required), \ `u_min`/`u_max`/`x_min`/`x_max` (vectors, optional).

**Arguments**

- `a` *(array)*
- `b` *(array)*
- `q` *(array)*
- `r` *(array)*
- `qf` *(array)*
- `horizon` *(integer)*
- `x0` *(array)*
- `u_min` *(array)*
- `u_max` *(array)*
- `x_min` *(array)*
- `x_max` *(array)*

---

## `solve_qp`

Wave 75.A — solve a convex quadratic program via Clarabel.rs \ (pure Rust, Apache-2.0). Problem: minimize (1/2) x^T P x + q^T x \ subject to A_eq x = b_eq and A_ineq x ≤ b_ineq. P must be symmetric \ PSD. Matrices are JSON nested arrays; vectors are flat JSON arrays. \ Equality and inequality blocks are optional (omit / null = no \ constraints of that kind). Args: `p` (matrix, required), `q` \ (vector, required), `a_eq` (matrix, optional), `b_eq` (vector, \ optional), `a_ineq` (matrix, optional), `b_ineq` (vector, optional). \ Returns x*, objective, status, iterations.

**Arguments**

- `p` *(array)*
- `q` *(array)*
- `a_eq` *(array)*
- `b_eq` *(array)*
- `a_ineq` *(array)*
- `b_ineq` *(array)*

---

## `tessellate`

Convert body to triangle mesh

**Arguments**

- `tolerance` *(number)*

---

