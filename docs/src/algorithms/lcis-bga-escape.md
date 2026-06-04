# LCIS BGA escape routing

> Kong, Yan, Wong & Ozdal — "Optimal bus sequencing for escape routing
> in dense PCBs," ICCAD 2007.

openie-cad's `cad-router::bga_escape` module is the **only
permissively-licensed (MIT/Apache) implementation of the Kong/Yan/Ozdal
escape-routing algorithm in the open-source ecosystem**. Every other
open-source autorouter (Freerouting v2, KiCad, OpenROAD) either lacks
BGA escape entirely (Freerouting removed it in v2.0) or is GPL.

## The problem

A BGA component has hundreds of pads in a 2D grid. Most pads are
*inside* the package outline — the autorouter can't reach them by
fanning outward. The standard solution is **escape routing**: drop a
via at each inner pad, traverse to an inner copper layer, and route
outward to the package boundary.

Two constraints make this hard:

1. **Geometric feasibility**. At fine pitches (≤ 0.5 mm, common for
   modern aQFNs and BGAs), a standard 0.45 mm via pad simply
   doesn't fit between adjacent BGA pads. JLCPCB's standard 6-layer
   service can't route a 0.4 mm-pitch aQFN-261 at all without
   switching to HDI tier with via-in-pad.

2. **Layer assignment / crossing avoidance**. Multiple buses
   competing for the same boundary region must be ordered so their
   escape paths don't cross.

## The algorithm

Kong/Yan/Ozdal split escape into two stages:

**Stage 1 — Bus sequencing.** For each BGA edge, project the buses
onto the boundary and find the **Longest Common Increasing
Subsequence (LCIS)** of (inner-pin position, exit position). Two
buses with (innerᵢ < innerⱼ AND exitᵢ < exitⱼ) are routable in
parallel without crossing; the LCIS picks the maximum such subset
in O(n log n) via tails-binary-search.

**Stage 2 — Layer assignment.** Iteratively peel off LISes. The
first LIS goes on routing layer 1; then those buses are removed;
the remaining buses form a new LIS for layer 2; and so on.

## openie-cad implementation

| Stage | Module | Wave |
|---|---|---|
| Data model + feasibility | `BgaGrid`, `feasibility_report` | 63.A |
| LCIS sequencing + layer assignment | `lcis_non_crossing_buses`, `assign_buses_to_layers` | 63.B |
| Track + Via geometry emission | `emit_all_escapes` | 63.C |
| Via-in-pad strategy (HDI tier) | `EscapeStrategy::ViaInPad` | 63.D |
| Autorouter integration | `Autorouter::route_with_bga_escape` | 63.E |
| MCP surface | `run_autorouter(bgas=[...])`, `check_bga_escape_feasibility` | 63.F |

## What the substrate fixes

The aether-t board audit (a real RF mesh radio design) reached
**rev 0.12 with 162 of ~350 tracks routed, 140 dangling pads, and
20 DRC violations** because its Python `bga_fanout.py` was
*disabled in source* with a comment explaining the 0.4 mm-pitch /
0.45 mm via geometry conflict. With Wave 63 deployed, the
`check_bga_escape_feasibility` MCP tool reports INFEASIBLE up
front with the exact reason — agents pick the right fab tier
*before* burning a 100-pair routing attempt.

## Calling it

```json
{
  "method": "tools/call",
  "params": {
    "name": "check_bga_escape_feasibility",
    "arguments": {
      "rows": 20, "cols": 20,
      "pitch_mm": 0.4, "pad_diameter_mm": 0.2,
      "fab_tier": "standard"
    }
  }
}
```

Returns INFEASIBLE with "no via fits between adjacent pads".

For the full routing pass, see [`run_autorouter`](../mcp-tools.md).
