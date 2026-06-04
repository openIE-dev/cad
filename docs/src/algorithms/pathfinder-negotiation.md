# PathFinder negotiation routing

> McMurchie & Ebeling — "PathFinder: a negotiation-based
> performance-driven router for FPGAs," FPGA 1995.

`cad-router::negotiated` implements PathFinder, the negotiation-based
congestion router that the FPGA community adopted in the 1990s and
that still underlies VPR (Verilog-to-Routing) and most academic
routers today. The algorithm has been a public-domain reference for
decades; openie-cad ships it as a PCB autorouter primitive.

## The problem

Classical rip-up-retry routing falls apart on congested designs:
each iteration rips one net to make room, but the *order* in which
nets are ripped becomes path-dependent. Two competing nets ping-pong
forever, ripping each other in turn.

PathFinder converts the conflict into a **negotiation**. Each cell
in the routing grid carries two cost components:

- **present-congestion cost** — how many nets currently route
  through this cell.
- **history cost** — accumulated congestion across previous
  iterations.

Both components multiply the cell's intrinsic cost in the
A*-style search. Cells that *have been congested in the past*
become permanently more expensive even after they're cleared, so
the same net no longer claims them in the next iteration. Equilibrium
is provably reached when no further reduction is possible — the
final routing is congestion-free if any congestion-free routing
exists.

## openie-cad implementation

`cad-router::negotiated`:

- `Congestion { present, history }` — per-cell cost maps with
  `present_weight` and `history_weight` tuning factors.
- `Congestion::cost(cell, weights)` — combined cost function.
- `Congestion::converged()` — equilibrium predicate.
- `route_negotiated(pathfinder, nets, config)` — the outer
  negotiation loop.

The Wave 55 substrate test verifies the algorithm converges on a
classic two-nets-share-a-channel scenario where naive A* would
ping-pong.

## Composition with the rest of cad-router

`Autorouter::route_multipin_nets` accepts a `negotiation_enabled`
flag that switches the inner loop from classical rip-up-retry to
the negotiation router. Composes naturally with:

- SALT topology generation (Wave 58) — chooses *where* the nets
  want to go.
- Kong/Yan/Ozdal LCIS BGA escape (Wave 63) — pre-places escape
  geometry before negotiation runs.
- Capacity-aware length-match meander insertion (Wave 64.B/C) —
  runs after negotiation converges.

## Calling it

```json
{
  "method": "tools/call",
  "params": {
    "name": "run_autorouter",
    "arguments": {
      "epsilon": 0.5,
      "negotiation": true
    }
  }
}
```

## Reference

McMurchie & Ebeling — "PathFinder: a negotiation-based
performance-driven router for FPGAs," FPGA 1995.
