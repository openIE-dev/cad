# SALT shallow-light Steiner tree

> Chen, Hu, Hentschel, Held — "SALT: provably good routing topology
> by a novel steiner shallow-light tree algorithm," ICCAD 2017.

`cad-router::steiner` ships SALT, a topology generator that gives the
caller continuous control over the **radius–wirelength tradeoff** —
the central tension in PCB tree topology design.

## The problem

For a multi-pin net, two extreme topologies bracket the design space:

- **Shortest-path tree (SPT)** — every sink reachable by its
  shortest path from the source. Minimum radius (signal arrival
  time), maximum wirelength.
- **Rectilinear Steiner minimal tree (RSMT)** — minimum total
  wirelength. Sinks may sit on stretched paths.

For clock nets and high-speed digital, low radius matters; for
analog and power, total wirelength matters. Everything else lives
on the curve between them.

## The algorithm

SALT exposes the tradeoff as a single parameter ε ∈ [0, ∞]:

- ε = 0 — shortest-path tree (minimum radius).
- ε → ∞ — Steiner minimal tree (minimum wirelength).
- Intermediate — a (1+ε)-shallow-light tree where every sink's
  tree-distance ≤ (1+ε) × its direct distance from the source,
  and total wirelength is bounded by O(1 + 1/ε) × RSMT-optimal.

Construction starts from baseline RSMT and iteratively re-wires any
sink whose path exceeds the (1+ε) bound, swapping the violating
parent edge for a direct edge to the source. Iteration terminates
when every sink satisfies the bound.

## openie-cad implementation

| Surface | Wave |
|---|---|
| `baseline_rsmt(terminals)` — MST over the Hanan grid | 57 |
| `salt(terminals, source, epsilon)` — (1+ε)-shallow-light iteration | 58 |
| `salt_decompose(MultipinNet, epsilon)` — Connection list for Autorouter | 59 |

The `run_autorouter` MCP tool accepts an `epsilon` argument; the
default ε = 1.0 is a reasonable middle ground for mixed boards.

## Why it matters

The substrate's PathFinder negotiation router (Wave 55) and Kong/Yan/Ozdal
BGA escape (Wave 63) both need a topology before they can lay tracks.
Without SALT they'd default to a fixed strategy — usually shortest
path, which leaves wirelength money on the table on relaxed-radius
nets like power rails. With SALT the agent tunes per-net.

## Reference

Chen, Hu, Hentschel, Held — "SALT: provably good routing topology by a
novel steiner shallow-light tree algorithm," ICCAD 2017.
