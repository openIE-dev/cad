# Introduction

**openie-cad** is a Rust-first, WASM-native CAD + ECAD + Multi-Physics
simulation engine. One codebase, one language, one file per product.

The thesis: **the design IS the simulation.** Every geometric entity carries
physics from the moment it's created. No export/import/mesh-setup loop. No
80% of engineering time lost at the design–simulation interface.

## Three entry points

You can use openie-cad in three ways:

- **MCP (agent-callable)** at <https://cad.openie.dev/mcp> — tools that
  an AI agent (Claude, GPT, etc.) can call directly. The
  [MCP tool reference](./mcp-tools.md) lists every tool with its arguments;
  it is generated from the registry, so it cannot go stale.
- **REST API** at <https://cad.openie.dev/api> — see [REST API docs](./rest-api.md).
- **Interactive viewer** at <https://cad.openie.dev/viewer/kicad?session=ID>
  — live KiCad PCB viewer for any MCP session.

What the rest of the industry now ships is the loop — generate, check,
repair — and openie-cad ships it too, as propose / certify / commit with
background jobs and cancellation. What no other CAD ships, as of November
2026, is the receipt: a content-addressed packet carrying the design with
its model hash, certification verdicts with receipts, and a measured energy
figure for the solve (1.69648 J for the reference plate pack, November 2026).
The agent layer is the table; the receipt and the joules are the stakes.

## What's substrate-native vs commercial

openie-cad ships several algorithms implemented in-tree. Status of this
table: November 2026.

| Algorithm | Paper / Origin | Where in substrate |
|---|---|---|
| LCIS BGA escape routing | Kong/Yan/Ozdal ICCAD 2007 | `cad-router::bga_escape` |
| FDTD Huygens NTFF antenna pattern | Balanis §6.7 textbook | `cad-sim::antenna` |
| Djordjevic-Sarkar + Cannonball-Huray 60 GHz loss | Shlepnev 2015, Simonovich 2015 | `cad-sim::signal` |
| preCICE-equivalent IQN-ILS coupling | Degroote 2009 / Chourdakis 2021 | `cad-sim::coupling` |
| SALT shallow-light Steiner topology | Chen et al. ICCAD 2017 | `cad-router::steiner` |
| Fang DAC 2024 length-match co-optimization | Fang et al. arXiv 2407.19195 | `cad-router::length_tuning` |

Every commercial equivalent — Altium, Cadence, Ansys, Comsol — charges thousands
of dollars per seat per year for the same capabilities. The
[algorithm theory pages](./algorithms/lcis-bga-escape.md) document each one
with the originating paper cited.

## Quickstart

See [Quickstart](./quickstart.md) for a 5-minute walkthrough of:

1. Routing a small PCB via MCP.
2. Generating a CAM toolpath.
3. Running a multi-physics coupled simulation.
