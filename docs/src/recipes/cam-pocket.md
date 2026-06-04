# CAM pocket clearing

Wave 70 opened the CAM domain with the first high-level operation:
`pocket_zigzag_rect` (rectangular pocket clearing via zig-zag raster)
plus the `generate_pocket_gcode` MCP tool that wraps the existing
5-axis substrate's RTCP post-processor.

## Quick example

A 50 × 30 × 8 mm pocket with a 6 mm cutter:

```json
{
  "method": "tools/call",
  "params": {
    "name": "generate_pocket_gcode",
    "arguments": {
      "width_mm": 50,
      "height_mm": 30,
      "depth_mm": 8,
      "cutter_diameter_mm": 6,
      "step_down_mm": 2,
      "step_over_mm": 3,
      "feedrate_mm_min": 1200,
      "plunge_rate_mm_min": 300,
      "spindle_rpm": 12000,
      "safe_z_mm": 5
    }
  }
}
```

Returns ready-to-run RS-274 G-code with the standard Fanuc-style
program markers (`%`, `O0001`, `G90 G94 G17 G40`), the spindle/
coolant block, line-numbered G00/G01 moves with axis components,
and the `M30` end-of-program.

## What the algorithm does

1. Compute the inset boundary by offsetting inward by the cutter
   radius — the pocket's outer wall ends up at exactly `width` ×
   `height` after machining.
2. For each Z level (0, −step_down, −2·step_down, …, −depth):
   - Rapid above the row start at `safe_z_mm`.
   - Plunge (cut feed) to the next Z.
   - Sweep zig-zag rows: alternating +X / −X traversals spaced
     `step_over_mm` apart along Y.
   - Retract to `safe_z_mm`.
3. Final Z snaps to exactly `−depth_mm` even if `depth_mm` isn't a
   multiple of `step_down_mm` — the floor is never overshot.

## Substrate guarantees (Wave 70 tests)

- Step-down picks the correct Z levels (10 × 10 × 5 mm pocket with
  step-down 2 mm → Z values {−2, −4, −5}).
- Adjacent zig-zag rows go opposite directions.
- All rapids end at Z ≥ `safe_z_mm`.
- Move chain is continuous (every move's start equals the previous
  move's end — no machine teleports).
- Tool centre never crosses the inset boundary
  (20 × 10 mm pocket + 4 mm cutter → centre stays ≥ 2 mm from each wall).

## Composition with other CAM operations

Wave 70 ships only pocket clearing. Wave 70.B will add:

- **Contour milling** (perimeter, with climb / conventional choice).
- **Drilling cycles** (G81, G83 peck, G73 high-speed peck).
- **Arbitrary-polygon pockets** via polygon offsetting + adaptive clearing.

The `cad_cam5::Move` / `ToolPath` model is already 5-axis-capable
(RTCP post-processor, kinematic smoothing, collision checking ship
in the substrate); the high-level operations are the missing piece
that Wave 70 + follow-ons are filling.
