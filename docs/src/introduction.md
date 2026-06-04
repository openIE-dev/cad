# Introduction

**openie-cad** is a Rust-first, WASM-native CAD + ECAD + Multi-Physics
simulation engine. One codebase, one language, one file per product.

The thesis: **the design IS the simulation.** Every geometric entity carries
physics from the moment it's created. No export/import/mesh-setup loop. No
80% of engineering time lost at the design–simulation interface.

## Three entry points

You can use openie-cad in three ways:

- **MCP (agent-callable)** at <https://cad.openie.dev/mcp> — 60+ tools that
  an AI agent (Claude, GPT, etc.) can call directly. The
  [MCP tool reference](./mcp-tools.md) lists every tool with its arguments.
- **REST API** at <https://cad.openie.dev/api> — see [REST API docs](./rest-api.md).
- **Interactive viewer** at <https://cad.openie.dev/viewer/kicad?session=ID>
  — live KiCad PCB viewer for any MCP session.

## What's substrate-native vs commercial

openie-cad ships several algorithms that, as of June 2026, exist in **no
other open-source codebase under an MIT/Apache license**:

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
