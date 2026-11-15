# MCP Tool Reference

Every tool in the cad-agent registry (the kernel-side tool set) served at `https://cad.openie.dev/mcp`.
Auto-generated from the substrate source on each release. For the full served surface, including planner-server tools, see https://cad.openie.dev/reference/mcp-tools/.

Call any tool by sending an MCP `tools/call` request with the tool's `name`
and the documented arguments. The session ID returned from the first call
keeps the project in memory for subsequent calls — see the
[Quickstart](./quickstart.md) for the round-trip pattern.

---

_81 tools registered._

---

## `add_track`

Add copper track segments between consecutive points on the \ project's PCB. Args: `net` (default "VCC"), `width` (mm, \ default 0.25), `layer` (e.g. "F.Cu", default "F.Cu"), \ `points` (REQUIRED array of [x, y] in mm, at least 2 points). \ If the project has no PCB yet, a default 100x80 mm 4-layer \ board is created first. Returns the segment count, layer, net, \ and width. Errors: missing/fewer than 2 points, malformed \ [x, y] pairs, or an unknown layer name.

**Arguments**

- `net` *(string)*
- `width` *(number)*
- `layer` *(string)*
- `points` *(array)*

---

## `add_via`

Add a through-hole via (F.Cu to B.Cu) on the project's PCB. \ Args: `x`, `y` (mm, default 0), `drill` (mm, default 0.3, must \ be positive), `pad` (mm, default drill + 0.5), `net` (default \ "GND"). If the project has no PCB yet, a default 100x80 mm \ 4-layer board is created first. Returns position, drill, pad, \ and net. Errors when `drill` is not positive.

**Arguments**

- `x` *(number)*
- `y` *(number)*
- `drill` *(number)*
- `pad` *(number)*
- `net` *(string)*

---

## `certify_drc`

Certify the project's PCB against the DRC engine and return \ a certification receipt (CERTIFIED or REFUSED) bound by \ SHA-256 to the exact board certified. Runs the five DRC \ checks — track widths, via sizes, clearance, hole spacing, \ copper edge — covering six violation types (TrackWidth, \ ViaDrill, ViaAnnularRing, Clearance, HoleSpacing, \ CopperEdge). Silkscreen, Courtyard, Connectivity, and \ Unconnected have no check function, are NOT run, and are \ named on the receipt. CERTIFIED iff the DRC report passed; \ warnings are listed and never fail. Re-running verification \ against a tampered board fails the hash check. No inputs. \ Errors when the project has no PCB yet — create one with \ create_board or import_kicad first.

**No arguments.**

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

Calculate differential-pair impedance (edge-coupled, 1 oz \ copper, 0.035 mm thickness, FR-4 er=4.3 fixed). Args: \ `trace_width` (mm, default 0.127), `gap` (mm, default 0.127), \ `dielectric_height` (mm, default 0.2). Returns \ `Diff pair Z_diff=<n>Ω, Z_common=<n>Ω, coupling=<n>`. Pure \ calculation — no project state is read or written.

**Arguments**

- `trace_width` *(number)*
- `gap` *(number)*
- `dielectric_height` *(number)*

---

## `check_impedance`

Calculate microstrip trace impedance for a surface trace \ (1 oz copper, 0.035 mm thickness, is fixed). Args: \ `trace_width` (mm, default 0.254), `dielectric_height` (mm, \ default 0.2), `er` (dielectric constant, default 4.3). Returns \ `Microstrip Z0=<n>Ω, delay=<n>ps/mm, Eff.Er=<n>`. Pure \ calculation — no project state is read or written.

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

## `control_plant`

Stabilize ANY controllable plant in the registry by name using \ iLQR — compute a control sequence that drives the state from `x0` \ toward a `target` under quadratic state/control/terminal costs. \ Works generically across the 350+ plants (no per-plant solver \ wiring); autonomous plants with no inputs are rejected (use \ simulate_plant instead). Args: `name` (string, required), `x0` \ (array, required — initial state), `target` (array, optional — \ goal state, default origin), `horizon` (integer, default 30), `q` \ (number — state-error weight, default 1.0), `r` (number — control \ weight, default 0.01), `qf` (number — terminal weight, default \ 10.0), `max_iter` (integer, default 50). Returns convergence, cost, \ the immediate control u0, the full control sequence, and the final \ state.

**Arguments**

- `name` *(string)*
- `x0` *(array)*
- `target` *(array)*
- `horizon` *(integer)*
- `q` *(number)*
- `r` *(number)*
- `qf` *(number)*
- `max_iter` *(integer)*

---

## `create_board`

Create a rectangular PCB and install it as the project's PCB \ (replaces any existing project PCB). Args: `name` (default \ "board"), `width` (mm, default 35), `height` (mm, default 25), \ `layers` (copper count, default 2). Returns the board dimensions \ and area, e.g. `Created 35x25mm 2-layer board (875mm²) and set \ it as the project's PCB`. Errors when width, height, or layers \ are not positive.

**Arguments**

- `name` *(string)*
- `width` *(number)*
- `height` *(number)*
- `layers` *(number)*

---

## `create_box`

Create an axis-aligned box solid in the kernel, centered at the \ origin. Args: `width`, `height`, `depth` (numbers, mm; each \ defaults to 10). Returns the new body's UUID and its dimensions, \ e.g. `Created box <uuid> (10x10x10mm)`. Fails with the kernel's \ error if the primitive is rejected.

**Arguments**

- `width` *(number)*
- `height` *(number)*
- `depth` *(number)*

---

## `create_cylinder`

Create a solid cylinder in the kernel along the Y axis, centered \ at the origin. Args: `radius` (mm, default 5), `height` (mm, \ default 10). Returns the new body's UUID. Fails with the \ kernel's error if the primitive is rejected.

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

Delete a kernel body by UUID. Arg: `body_id` (string, required) \ — a body UUID from create_box / eval_ocl / list_bodies. Returns \ `Deleted body <id>`. Errors: `Invalid body ID '<id>': ...` when \ the string is not a valid entity id; `Failed to delete body: ...` \ when no such body exists.

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

Estimate manufacturing cost for the project's PCB + BOM. Args: \ `quantity` (number, default 5), `fab` (string, default \ "JLCPCB"; also "PCBWay", "OSHPark"). Returns the \ estimated cost for the requested quantity at that fab's \ pricing. Unknown fabs fall back to a generic per-board rate.

**Arguments**

- `quantity` *(number)*
- `fab` *(string)*

---

## `estimate_mhe`

Wave 81.A — Moving Horizon Estimation, the dual of MPC. \ Given a window of past noisy measurements (y_seq) and \ controls (u_seq) plus an arrival-cost prior (x_prior, p_0), \ returns the Bayesian-optimal smoothed state trajectory. On \ linear-Gaussian data the final-time estimate matches a \ Kalman filter run over the same window — but the QP framing \ extends to state bounds (later wave) and nonlinear plants \ (iterative LTV, like Wave 76's MPC, also a later wave). \ Args: `a` (n×n required), `b` (n×m optional — omit for \ autonomous), `c` (p×n required), `q_proc` (n×n required, \ process noise covariance), `r_meas` (p×p required, \ measurement noise covariance), `p_0` (n×n required, prior \ covariance), `x_prior` (length n required), `u_seq` (array \ of length-m vectors), `y_seq` (array of length-p vectors). \ horizon = y_seq.len(). Returns current_estimate, full x_seq, \ cost, iterations.

**Arguments**

- `a` *(array)*
- `b` *(array)*
- `c` *(array)*
- `q_proc` *(array)*
- `r_meas` *(array)*
- `p_0` *(array)*
- `x_prior` *(array)*
- `u_seq` *(array)*
- `y_seq` *(array)*

---

## `eval_ocl`

Parse and evaluate an OCL document against the engine. Arg: \ `source` (string, required). Produces kernel geometry plus \ ECAD netlist/PCB, schematic, sim, and firmware records. Parse \ errors fail fast (`OCL evaluation error: ...`); errors inside \ domain blocks are non-fatal by design but reported alongside \ the result (`... errors inside domain blocks: ...`) so they \ are never silent.

**Arguments**

- `source` *(string)*

---

## `export_brep`

Export bodies as OpenCascade BREP (polyhedral subset — \ each triangle becomes a planar Face. Importable by FreeCAD / \ Salome / OCCT-CLI for viz or further Boolean ops; not a \ substitute for full surface BREP — for that use IGES / STEP).

**No arguments.**

---

## `export_gerber`

Report the Gerber layer set for the project's PCB (F.Cu, B.Cu, \ F.Mask, B.Mask, F.SilkS, Edge.Cuts, drill). No inputs. \ Currently returns the layer list only — no files are written \ yet (roadmap); the full Gerber writer belongs to the \ format/mfg surface.

**No arguments.**

---

## `export_gltf`

Encode the FIRST kernel body as glTF 2.0 in memory and report \ its size. No inputs. Returns `glTF exported: <N> bytes`. \ Errors: `No bodies` when the project has no geometry. Note: the \ payload is not written to disk or returned — this validates the \ export path only.

**No arguments.**

---

## `export_obj`

Encode the FIRST kernel body as Wavefront OBJ text in memory \ and report its size. No inputs. Returns `OBJ exported: <N> \ bytes`. Errors: `No bodies` when the project has no geometry. \ Note: the text is not written to disk or returned — this \ validates the export path only.

**No arguments.**

---

## `export_ply`

Encode the FIRST kernel body as Stanford PLY (ASCII) in memory \ and report its size. No inputs. Returns `PLY exported: <N> \ bytes, <M> triangles`. Errors: `No bodies` when the project has \ no geometry. Note: the text is not written to disk or returned \ — this validates the export path only.

**No arguments.**

---

## `export_stl`

Encode the FIRST kernel body as binary STL in memory and \ report its size. No inputs. Returns `STL exported: <N> bytes, \ <M> triangles`. Errors: `No bodies` when the project has no \ geometry. Note: the bytes are not written to disk or returned \ — this validates the export path only.

**No arguments.**

---

## `export_xao`

Export bodies as Salome XAO (BREP payload + topology + groups). \ Argument `name` overrides the geometry name (defaults to project.name).

**Arguments**

- `name` *(string)*

---

## `generate_bom`

Count the project's BOM lines from PCB footprints and schematic \ symbols. No inputs. Returns `<N> entries (<F> footprints, <S> \ schematic symbols)`. Note: this reports the count only — it \ does not emit a BOM file; use generate_jlcpcb_bom for a JLCPCB \ CSV.

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

## `generate_drawing`

Wave 81 — 2D engineering drawing of a kernel body with hidden-line \ removal. Tessellates the body and renders a 2×2 multi-view SVG \ (front / top / right / iso) where edges occluded by the part's own \ surface are drawn dashed and visible edges solid — the standard \ drafting convention. Returns the full SVG inline plus a length and \ snippet. Args: `body` (index into the project's bodies, default 0), \ `part_name` (title block, default 'part'), `tolerance` (tessellation \ chord error mm, default 0.1, clamped 0.01–1.0), `feature_angle_deg` \ (crease angle for edge extraction, default 30, clamped 1–89), \ `samples` (per-edge occlusion sampling, default 24, clamped 2–64). \ Requires at least one body in the project.

**Arguments**

- `body` *(integer)*
- `part_name` *(string)*
- `tolerance` *(number)*
- `feature_angle_deg` *(number)*
- `samples` *(integer)*

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

Generate firmware skeleton code for a target MCU family. Arg: \ `target` (string, default "Esp32S3"; e.g. stm32f4, stm32h7, \ nrf52840, esp32, esp32c3, rp2040, rp2350 — case-insensitive). \ Returns the generated skeleton source. The output is a marked \ skeleton (peripheral drivers not yet generated) — see the \ cad-firmware scope; unknown targets are an error naming the \ supported families.

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

Parse a KCL (Zoo.dev) source string and report the statement \ count. Arg: `source` (string, required) — the KCL text. Returns \ `Imported KCL: <N> statements` or `KCL error: ...` with the \ parse failure. Note: currently validates/parses only — the \ parsed program is not yet converted into kernel geometry \ (roadmap).

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

## `list_control_plants`

Browse the cad-control dynamics-plant registry — 350+ canonical \ control/dynamical-systems plants (chaos, robotics, power, process, \ aerospace, biology, economics, …), each callable by name. Lists \ plants as `Name [state×control] — summary`. Args: `filter` (string, \ optional — case-insensitive substring matched against the name and \ summary, e.g. "motor", "chaos", "pendulum"), `limit` (integer, \ optional, default 40). Pair with `plant_info` and `simulate_plant`.

**Arguments**

- `filter` *(string)*
- `limit` *(integer)*

---

## `optimize_placement`

Wave 81 — simulated-annealing GLOBAL component placement on \ project.eda.pcb. Unlike run_placement (a local force-directed \ relaxation), this minimizes total half-perimeter wirelength (HPWL) \ plus an overlap/out-of-bounds penalty over a geometric cooling \ schedule, so it escapes the local minima that trap the FR solver. \ Reports HPWL and overlap before/after. Args: `iterations` (move \ budget, default 8000, clamped 100–50000), `seed` (PRNG seed for \ reproducibility), `overlap_weight` (penalty on overlap area mm², \ default 50), `allow_rotation` (90° transposes, default true), \ `locked` (array of refs to pin). Requires a PCB with ≥2 footprints.

**Arguments**

- `iterations` *(integer)*
- `seed` *(integer)*
- `overlap_weight` *(number)*
- `allow_rotation` *(boolean)*
- `locked` *(array)*

---

## `parse_ocl`

Parse an OCL source string into an AST without evaluating it. \ Arg: `source` (string, required) — the OCL document text. \ Returns `Parsed OCL: <N> items` or the first parse error with \ its source span (`OCL parse error: ...`). Use eval_ocl to \ actually execute the document.

**Arguments**

- `source` *(string)*

---

## `place_footprint`

Place a component footprint instance on the project's PCB at \ (x, y) mm. Args: `reference` (e.g. "U1", default "U1"), \ `lib_id` (footprint library id, e.g. "Package:QFP", default \ "Package:QFP"), `x`, `y` (mm, default 0). If the project has \ no PCB yet, a default 100x80 mm 4-layer board is created first. \ Returns the reference, position, and footprint UUID. The \ footprint carries no pads — use place_vendor_footprint for a \ fully-detailed vendor footprint from a .kicad_mod file.

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

## `plant_info`

Report the state dimension, control (input) dimension, and one-line \ summary of a single registered dynamics plant. Args: `name` (string, \ required — the plant's registry name, e.g. "Lorenz", "CartPole", \ "DynamicBicycle"; browse with list_control_plants).

**Arguments**

- `name` *(string)*

---

## `recognise_features`

Recognise manufacturing features in a STEP B-rep solid by calling the quiddity CLI (0.4.9) as a process and parsing its JSON recognition document. Arg: `step` (string, required) — STEP file content. Returns the recognition receipt as JSON: verdict Recognised/Refused, records with parameters and evidence_status, face accounting, source_sha256, and the measured-joule fields under boundary (b). Refuses (never crashes) when the CLI is absent, the input is not a B-rep solid/shell with faces, or the document fails validation — the refusal receipt names the precondition. Requires the quiddity CLI installed (`pip install quiddity==0.4.9`). No project state is read or written.

**Arguments**

- `step` *(string)*

---

## `render_pcb_svg`

Render project.eda.pcb as an SVG board preview — outline, copper \ tracks per layer, vias, placed footprints with reference designators. \ Returns the SVG inline so agents/users can see what was imported. \ Optional args: `scale` (px/mm, default 10), `margin_mm` (default 5), \ `show_references` (default true). Requires a PCB.

**Arguments**

- `scale` *(number)*
- `margin_mm` *(number)*
- `show_references` *(boolean)*

---

## `rl_policy_action`

Wave 80.D — query a Candle-trained Gaussian MLP policy \ (Wave 80.A's GaussianMlpPolicy) for an action given a \ state. Loads weights via the same pure-Rust mmaped-\ safetensors path as cad-deepcad. Use this to deploy a \ policy trained by Wave 80.B/C's `train_vpg` / `train_ppo`, \ OR a policy trained externally (PyTorch / JAX) and \ exported to safetensors with the substrate's tensor \ naming convention (`fc1`, `fc2`, `mean_head`, `log_std`). \ Stateless across calls: caller threads the state. \ Args: `weights_path` (string, required, .safetensors path), \ `n_state` (int, required), `n_action` (int, required), \ `hidden` (int, default 64), `state` (length-`n_state` \ vector, required), `stochastic` (bool, default false — \ returns the mean if false, samples from the Gaussian if \ true), `seed` (int, optional — used when stochastic=true).

**Arguments**

- `weights_path` *(string)*
- `n_state` *(integer)*
- `n_action` *(integer)*
- `hidden` *(integer)*
- `state` *(array)*
- `stochastic` *(boolean)*
- `seed` *(integer)*

---

## `route_dmf_droplets`

Route a digital-microfluidics (EWOD) assay: move droplets across \ an electrode grid from start to target, collision-free, and \ return the activation schedule. Enforces the fluidic constraint \ that two droplets may never be on adjacent electrodes at once \ (they'd merge) via prioritised time-expanded A* with a one-cell \ exclusion halo. Args: `nx`,`ny` (electrode grid size, required), \ `droplets` (array; each {`id`?, `start`:[x,y], `target`:[x,y]}). \ Returns the timestep count and each droplet's routed endpoints; \ a verified ≥2-cell separation holds at every step. Set \ `optimal` true to route with Conflict-Based Search (minimum \ total travel) instead of the fast greedy planner.

**Arguments**

- `nx` *(integer)*
- `ny` *(integer)*
- `droplets` *(array)*
- `optimal` *(boolean)*

---

## `route_microfluidic_design`

Synthesise a microfluidic layout from a MINT-subset netlist — \ the fluidic analogue of PCB place-and-route. Parses components \ and the channels between their ports, places the components \ without overlap, and routes each channel as a Manhattan path \ (grid A*) around every footprint. MINT grammar (one per line): \ `component <id> <kind> <width_um> <height_um>` and \ `channel <id>.<port> <id>.<port>`, where ports are the \ auto-generated edge midpoints W/E/N/S. Args: `mint` (netlist \ text, required), `gap_um` (placement gap, default 400), \ `row_width_um` (shelf wrap width, default 8000), `pitch_um` \ (routing grid, default 100). Returns component placements, \ channel count, and total channel length.

**Arguments**

- `mint` *(string)*
- `gap_um` *(number)*
- `row_width_um` *(number)*
- `pitch_um` *(number)*

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

Run the Design-for-Manufacturability check against the \ project's PCB for a target fab. Arg: `fab` (string, default \ "JLCPCB"; also "PCBWay", "OSHPark"). Returns \ manufacturable yes/NO, error/warning counts, the first 10 \ violations, and the estimated cost per board. Errors: unknown \ fab name (`Unknown fab ... — supported: JLCPCB | PCBWay | \ OSHPark`), or no PCB in the project — create one with \ create_board or import_kicad first.

**Arguments**

- `fab` *(string)*

---

## `run_drc`

Run the Design Rule Check (cad-eda DRC engine) against the \ project's PCB using the project's DRC rules, augmented with \ per-fab energy/CO2 estimates. No inputs. Returns PASSED/FAILED \ plus the violation list. Errors when the project has no PCB \ yet — create one with create_board or import_kicad first. (A \ project with no layout is an error, not a pass.)

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

Report the thermal-simulation entry point for the project. \ Arg: `ambient_c` (number, °C, default 25). Currently returns \ the requested ambient temperature and notes the solver needs a \ mesh — a full thermal solve is NOT wired yet (roadmap). Do not \ treat the result as a simulation outcome.

**Arguments**

- `ambient_c` *(number)*

---

## `simulate_conjugate_heat`

Solve conjugate heat transfer across a two-material (e.g. \ solid/fluid) composite wall: a single temperature field with \ per-cell conductivity, harmonic-mean face coupling (which makes \ temperature AND heat flux continuous across the interface \ automatically), and optional flow that advects heat. Reports the \ steady interface temperature vs the analytic series-resistance \ value and how continuous the flux is across the conductivity jump. \ Args: `nx`,`ny` (grid, default 60×6), `dx` (m, default 1e-4), \ `k1`,`k2` (W/m·K, left/right halves, default 1/3), `rho_cp` \ (J/m³·K, default 1e6), `t_left`,`t_right` (default 1/0), `flow` \ (m/s, default 0).

**Arguments**

- `nx` *(integer)*
- `ny` *(integer)*
- `dx` *(number)*
- `k1` *(number)*
- `k2` *(number)*
- `rho_cp` *(number)*
- `t_left` *(number)*
- `t_right` *(number)*
- `flow` *(number)*

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

## `simulate_microfluidic_network`

Solve a microfluidic channel network by nodal analysis — the \ hydraulic analogue of circuit MNA (pressure↔voltage, flow↔current, \ Hagen-Poiseuille resistance↔resistance, reservoir↔voltage source, \ flow source↔current source). This is MMFT-1D-class 1D network \ simulation: give it the channel graph + boundary conditions and it \ returns every nodal pressure and channel flow rate. Args: `n_nodes` \ (int, required), `viscosity` (Pa·s, default 1e-3 = water), `channels` \ (array; each {`from`,`to`} plus either `resistance` (Pa·s/m³), or \ `radius`+`length` (circular), or `width`+`height`+`length` \ (rectangular) — all SI metres), `boundaries` (array; each {`node`} \ plus either `pressure` (Pa) or `flow` (m³/s); at least one pressure \ BC required to anchor the network). Series/parallel channels \ reproduce the textbook R_eq; a single channel reproduces \ Hagen-Poiseuille exactly.

**Arguments**

- `n_nodes` *(integer)*
- `viscosity` *(number)*
- `channels` *(array)*
- `boundaries` *(array)*

---

## `simulate_mixing_field`

Simulate solute mixing in a channel as a 2D advection-diffusion \ field — the grid solver behind the analytical Péclet/mixing-length \ estimates. Seeds a left-half slug (sharp interface) and advances it; \ reports how much the interface homogenises (variance drop = % mixed). \ Runs on the GPU compute kernel when the server is built with the \ `gpu` feature and an adapter is present, else on the identical CPU \ stencil (they agree to f32 precision). Args: `nx`,`ny` (grid, default \ 48), `dx` (m, default 1e-5), `diffusivity` (m²/s, default 1e-9), \ `ux`,`uy` (m/s, default 0), `steps` (default 200).

**Arguments**

- `nx` *(integer)*
- `ny` *(integer)*
- `dx` *(number)*
- `diffusivity` *(number)*
- `ux` *(number)*
- `uy` *(number)*
- `steps` *(integer)*

---

## `simulate_natural_convection`

Solve buoyancy-driven natural convection in a differentially-heated \ square cavity (Boussinesq, vorticity-streamfunction) — the velocity \ is generated by the temperature field, not prescribed. Reports the \ hot-wall Nusselt number (Nu=1 ⇒ pure conduction; de Vahl Davis \ benchmark Nu≈1.118 at Ra=10³) and the peak flow speed. Set \ `smagorinsky` > 0 to enable the algebraic eddy-viscosity turbulence \ closure. NOTE: this runs a steady-state CFD solve synchronously and \ is bounded (grid ≤51, capped iterations) — higher Ra takes longer. \ Args: `n` (grid, default 31, clamped 11–51), `ra` (Rayleigh, default \ 1e3), `pr` (Prandtl, default 0.71), `smagorinsky` (Cs, default 0), \ `max_steps` (default 40000, capped 60000).

**Arguments**

- `n` *(integer)*
- `ra` *(number)*
- `pr` *(number)*
- `smagorinsky` *(number)*
- `max_steps` *(integer)*

---

## `simulate_plant`

Roll a registered dynamics plant forward in time from an initial \ state under a constant control input, returning the (down-sampled) \ state trajectory and final state. Works for any of the 350+ plants in \ the registry — no per-plant wiring. Args: `name` (string, required), \ `x0` (array, required — initial state, must match the plant's state \ dimension; see plant_info), `control` (array, optional — constant \ input, defaults to zeros of the plant's control width), `steps` \ (integer, optional, default 100). Returns final_state + a sampled \ trajectory + a finiteness flag.

**Arguments**

- `name` *(string)*
- `x0` *(array)*
- `control` *(array)*
- `steps` *(integer)*

---

## `simulate_rans_channel`

Wave 81 — RANS turbulent channel flow via the Spalart-Allmaras \ one-equation model. Solves the fully-developed half-channel velocity \ profile in wall units (ν=1, u_τ=1) and reports the log-law slope \ (1/κ ≈ 2.44 when the model is correct; κ≈0.41), the centreline u⁺, \ and a few profile points (y⁺, u⁺, ν_t⁺) across the wall / buffer / \ log layers. NOTE: a synchronous solve, bounded (grid ≤256, sweeps \ ≤2000). Args: `re_tau` (friction Reynolds number, default 395, \ clamped 50–5000), `n` (grid points across half-channel, default 96, \ clamped 16–256), `outer_iters` (Picard sweeps, default 300, clamped \ 20–2000).

**Arguments**

- `re_tau` *(number)*
- `n` *(integer)*
- `outer_iters` *(integer)*

---

## `simulate_rayleigh_benard_3d`

Solve 3D Rayleigh-Bénard convection — a fluid layer heated from \ below — with a full 3D incompressible Navier-Stokes projection \ solver (Boussinesq buoyancy). Reports the Nusselt number and peak \ speed, and whether the layer has crossed the convective instability \ (Nu>1) or stays conductive (Nu=1). Above the critical Rayleigh \ number (Ra_c≈1708) a small perturbation grows into convection \ cells. NOTE: a synchronous 3D CFD solve, bounded (grid ≤16³, steps \ ≤2500) — higher Ra/steps cost more. Args: `n` (horizontal grid, \ default 12, 6–16), `nz` (vertical, default n), `ra` (Rayleigh, \ default 5e4), `pr` (Prandtl, default 1), `steps` (default 1200).

**Arguments**

- `n` *(integer)*
- `nz` *(integer)*
- `ra` *(number)*
- `pr` *(number)*
- `steps` *(integer)*

---

## `solve_acrobot_step`

Wave 89.B — one iLQR call on the Acrobot, a 2-link arm \ with only the ELBOW joint actuated (shoulder is passive). \ The Spong-canonical underactuated benchmark — like \ cart-pole, only one control acts on 4 states, but the \ coupling is via Coriolis (not cart acceleration). True \ swing-up to upright is genuinely fragile for local \ methods, so this tool returns whatever the iLQR finds \ without claiming stabilisation — useful for trajectory \ exploration, energy-shaping bootstrap, or RL warm-starts. \ Args: arm physical params m1, m2, l1, l2, lc1, lc2, i1, \ i2, gravity (defaults to standard Spong-1995 acrobot). \ `dt` (default 0.02), `horizon` (int, default 25), `x0` \ (length-4: q1, q2, qd1, qd2; required), `x_target` \ (length-4; required). Optional cost: matrices q/r/qf. \ iLQR knobs max_iter/tol/reg_init. Returns elbow-torque \ sequence + predicted trajectory.

**Arguments**

- `m1` *(number)*
- `m2` *(number)*
- `l1` *(number)*
- `l2` *(number)*
- `lc1` *(number)*
- `lc2` *(number)*
- `i1` *(number)*
- `i2` *(number)*
- `gravity` *(number)*
- `dt` *(number)*
- `horizon` *(integer)*
- `x0` *(array)*
- `x_target` *(array)*
- `q` *(array)*
- `r` *(array)*
- `qf` *(array)*
- `max_iter` *(integer)*
- `tol` *(number)*
- `reg_init` *(number)*

---

## `solve_arm_setpoint`

Wave 85.C — solve setpoint regulation for a 2-DOF planar arm \ via the Wave 85.A PlanarTwoLinkArm plant + Wave 78 iLQR. \ The plant is built from a cad-assembly Mechanism (Wave 85.B \ bridge) — the same robot-arm model a CAD user designs in \ the assembly editor feeds straight into the optimal-control \ substrate, no manual dynamics derivation needed. \ Args: arm physical params m1, m2, l1, l2, lc1, lc2, i1, i2, \ gravity (all numbers, defaults to the textbook m=l=1, \ lc=0.5, i=0.1 arm with g=None for floating); `name` \ (string, optional), `dt` (s, default 0.01), `horizon` (int, \ default 30), `x0` (length-4 vector: q1, q2, qd1, qd2; \ required), `x_target` (length-4 vector; required), \ optional cost matrices `q`/`r`/`qf` or scalar weights \ `q_pos`/`q_vel`/`r_torque`, `max_iter` (default 50), \ `tol` (default 1e-6), `reg_init` (default 1e-2). \ Returns the joint-torque sequence τ_0..τ_{N-1} and the \ predicted state trajectory under the true nonlinear arm \ dynamics.

**Arguments**

- `name` *(string)*
- `m1` *(number)*
- `m2` *(number)*
- `l1` *(number)*
- `l2` *(number)*
- `lc1` *(number)*
- `lc2` *(number)*
- `i1` *(number)*
- `i2` *(number)*
- `gravity` *(number)*
- `dt` *(number)*
- `horizon` *(integer)*
- `x0` *(array)*
- `x_target` *(array)*
- `q` *(array)*
- `r` *(array)*
- `qf` *(array)*
- `q_pos` *(number)*
- `q_vel` *(number)*
- `r_torque` *(number)*
- `max_iter` *(integer)*
- `tol` *(number)*
- `reg_init` *(number)*

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

## `solve_cartpole_stabilize`

Wave 86.B — solve one step of receding-horizon iLQR to \ stabilize a cart-pole near its upright (unstable) \ equilibrium. The cart-pole is the canonical UNDERACTUATED \ control benchmark: 1 control (horizontal cart force) over \ 4 states (cart pos, pole angle, cart vel, pole ang-vel), \ with q2 = 0 the unstable equilibrium (pole upright). The \ substrate's iLQR (Wave 78) handles this without \ modifications — same algorithm as the bicycle and arm \ plants, fully nonlinear inertial coupling. \ Args: `cart_mass` (kg, default 1.0), `pole_mass` (kg, \ default 0.1), `pole_length` (m, default 0.5), `gravity` \ (m/s², default 9.81), `dt` (s, default 0.02), `horizon` \ (int, default 25), `x0` (length-4 vector required: \ cart_pos, pole_angle, cart_vel, pole_ang_vel; angles in \ rad), optional cost overrides via matrices `q`/`r`/`qf` \ OR scalar weights `q_cart`/`q_angle`/`r_force`, iLQR \ knobs `max_iter`/`tol`/`reg_init`. Returns cart-force \ sequence + predicted state trajectory.

**Arguments**

- `cart_mass` *(number)*
- `pole_mass` *(number)*
- `pole_length` *(number)*
- `gravity` *(number)*
- `dt` *(number)*
- `horizon` *(integer)*
- `x0` *(array)*
- `q` *(array)*
- `r` *(array)*
- `qf` *(array)*
- `q_cart` *(number)*
- `q_angle` *(number)*
- `r_force` *(number)*
- `max_iter` *(integer)*
- `tol` *(number)*
- `reg_init` *(number)*

---

## `solve_impedance_field`

Wave 81 — field-solver transmission-line impedance. Runs a 2D \ finite-difference Laplace solve over the conductor/dielectric \ cross-section to extract the TRUE Z0, effective εr, C, L and phase \ velocity, whereas check_impedance uses the analytic microstrip \ closed form. Default geometry is a microstrip: a signal trace on a \ layered substrate over a ground plane. NOTE: a synchronous PDE solve, \ bounded (grid ≤200×200). Args: `nx`,`ny` (grid, default 120×80, \ clamped 20–200), `dx`,`dy` (cell pitch m, default 5e-5), \ `trace_width` (m, default 0.254e-3), `dielectric_height` (m, default \ 0.2e-3), `trace_thickness` (m, default 0.035e-3), `er` (default 4.3).

**Arguments**

- `nx` *(integer)*
- `ny` *(integer)*
- `dx` *(number)*
- `dy` *(number)*
- `trace_width` *(number)*
- `dielectric_height` *(number)*
- `trace_thickness` *(number)*
- `er` *(number)*

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

## `solve_n_link_arm_setpoint`

Wave 87.C — receding-horizon iLQR for a planar n-link \ serial arm of any chain length. Uses the RNEA-based \ forward dynamics in cad-control (Wave 87.A) — same iLQR \ algorithm as solve_arm_setpoint, generalised to arbitrary \ n. The plant is built from a cad-assembly Mechanism (Wave \ 87.B bridge) so a CAD-designed robot arm with any number \ of joints feeds straight into the substrate. \ Args: `links` (array of {mass, length, com_offset, inertia} \ objects; required), `gravity` (m/s², optional), `dt` (s, \ default 0.01), `horizon` (int, default 25), `x0` (length \ 2·n: q₀…q_{n-1}, q̇₀…q̇_{n-1}; required), `x_target` (same \ length; required), `name` (optional). Cost can be supplied \ via matrices `q`/`r`/`qf` OR via scalar weights `q_pos`/`q_vel`/\ `r_torque`/`qf_pos`/`qf_vel`. iLQR knobs: `max_iter`/`tol`/\ `reg_init`. Returns joint-torque sequence (each length n) \ + predicted state trajectory (each length 2·n).

**Arguments**

- `name` *(string)*
- `links` *(array)*
- `gravity` *(number)*
- `dt` *(number)*
- `horizon` *(integer)*
- `x0` *(array)*
- `x_target` *(array)*
- `q` *(array)*
- `r` *(array)*
- `qf` *(array)*
- `q_pos` *(number)*
- `q_vel` *(number)*
- `r_torque` *(number)*
- `qf_pos` *(number)*
- `qf_vel` *(number)*
- `max_iter` *(integer)*
- `tol` *(number)*
- `reg_init` *(number)*

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

## `solve_quadrotor_hover`

Wave 88.B — drive a planar VTOL quadrotor (PVTOL) to a \ hover setpoint via the substrate's iLQR. The PVTOL has 6 \ states (px, py, body-tilt θ, ẋ, ẏ, ω) and 2 controls \ (left and right rotor thrusts) — underactuated, so \ horizontal motion has to be commanded by tilting the \ body. Hover equilibrium is θ = 0, both rotors at m·g/2. \ Args: `mass` (kg, default 1.0), `arm_length` (m, body \ centre to rotor; default 0.25), `inertia` (kg·m², default \ 0.1), `gravity` (m/s², default 9.81), `dt` (s, default \ 0.02), `horizon` (int, default 30), `x0` (length-6, \ required), `x_target` (length-6, required). Optional \ cost: full matrices `q`/`r`/`qf` (defaults emphasise tilt \ and altitude). iLQR knobs `max_iter`/`tol`/`reg_init`. \ Returns rotor-thrust sequence + predicted state \ trajectory + hover_thrust_per_rotor for reference.

**Arguments**

- `mass` *(number)*
- `arm_length` *(number)*
- `inertia` *(number)*
- `gravity` *(number)*
- `dt` *(number)*
- `horizon` *(integer)*
- `x0` *(array)*
- `x_target` *(array)*
- `q` *(array)*
- `r` *(array)*
- `qf` *(array)*
- `max_iter` *(integer)*
- `tol` *(number)*
- `reg_init` *(number)*

---

## `solve_vlm`

C3-1 — vortex-lattice solve of a wing planform via the \ clean-room VLM, routed through the V1 input validator \ (`validate_builder_job` → `ValidatedVlmJob::solve_metered`). \ Args: `planform` (string: rectangular | tapered | elliptic, \ default rectangular), `span` (default 10), `chord` (default \ 1; root chord for tapered/elliptic), `tip_chord` (default = \ chord; tapered only), `sweep_deg` / `dihedral_deg` / \ `twist_deg` (default 0), `nx` / `ny` (panel counts, defaults \ 4 / 20), `alpha_deg` (default 5), `v_inf` (default 50), \ `rho` (default 1.225). Returns result + both receipts \ (validation + solve) as separate JSON records, never summed. \ A refused job returns its refusal receipt as a normal \ result. Low-speed attached-flow theory only.

**Arguments**

- `planform` *(string)*
- `span` *(number)*
- `chord` *(number)*
- `tip_chord` *(number)*
- `sweep_deg` *(number)*
- `dihedral_deg` *(number)*
- `twist_deg` *(number)*
- `nx` *(integer)*
- `ny` *(integer)*
- `alpha_deg` *(number)*
- `v_inf` *(number)*
- `rho` *(number)*

---

## `solve_vspaero`

C3-1 — VSPAERO vortex-lattice / panel solve via the vendored \ solver subprocess, metered. Same planform inputs as the VLM \ path: `planform` (rectangular/tapered/elliptic, default \ rectangular), `span` (default 10), `chord` (default 1), \ `tip_chord` (default = chord), `sweep_deg`/`dihedral_deg`/ \ `twist_deg` (default 0), `nx`/`ny` (defaults 4/20, clamped \ 0–4096 each; the validator's 2,000-panel cap is the binding \ bound, and 0 is a named refusal). Flow: `mach` (default 0.05), `alpha_deg` (default 5), \ `beta_deg` (default 0), `v_inf` (default 50), `rho` (default \ 1.225), `re_cref` (default 1e7), `wake_iters` (default 3, \ clamped 0–10; 0 is a frozen wake upstream). The mesh is \ validated by the V1 validator before the case is built; a \ refusal returns its receipt as a normal result. A solve \ returns JSON with the polar (cl, cdi, cd0, cm) plus the \ validation receipt and the solve receipt as separate \ records, never summed. Joule fields are measured or \ absent, never estimated. No project state is read or written.

**Arguments**

- `planform` *(string)*
- `span` *(number)*
- `chord` *(number)*
- `tip_chord` *(number)*
- `sweep_deg` *(number)*
- `dihedral_deg` *(number)*
- `twist_deg` *(number)*
- `nx` *(integer)*
- `ny` *(integer)*
- `mach` *(number)*
- `alpha_deg` *(number)*
- `beta_deg` *(number)*
- `v_inf` *(number)*
- `rho` *(number)*
- `re_cref` *(number)*
- `wake_iters` *(integer)*

---

## `tessellate`

Tessellate the FIRST body in the kernel into a triangle mesh. \ Arg: `tolerance` (number, mm, default 0.1) — smaller is finer. \ Returns `<N> vertices, <M> triangles`. Errors: `No bodies to \ tessellate` when the project has no geometry — create some with \ create_box or eval_ocl first. Note: only the first body is \ meshed; there is no body selector yet.

**Arguments**

- `tolerance` *(number)*

---

