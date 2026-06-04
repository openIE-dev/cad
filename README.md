# openie-cad

> Rust-first, WASM-native CAD + ECAD + Multi-Physics simulation engine.
> One codebase, one language, one file per product.

This repository hosts the **public documentation, examples, and recipes** for
openie-cad. The substrate itself is free to use — host it locally, or call it
over MCP at [`cad.openie.dev`](https://cad.openie.dev).

📖 **Documentation:** <https://docs.openie.dev>

## What's here

```
docs/         mdBook source — quickstart, MCP tool reference, recipes,
              algorithm theory, changelog
examples/    Sample OCL programs, .kicad_pcb boards, multi-physics setups
```

## What openie-cad does

| Domain | Capability |
|---|---|
| **3D CAD** | Truck B-Rep NURBS kernel. Boolean ops. Parametric history. Sheet metal, mold, CAM toolpaths. |
| **ECAD** | Schematic + 32-layer PCB. A* + PathFinder negotiation autorouter. SALT shallow-light Steiner topology. Kong/Yan/Ozdal LCIS BGA escape (only MIT-licensed implementation). Differential pair length tuning. DRC + DFM + Gerber/ODB++/JLCPCB. |
| **Multi-Physics** | 38 physics domains with real governing equations. preCICE-style partitioned coupling with Aitken + IQN-ILS quasi-Newton acceleration. FEM, CFD, FDTD, SPICE. |
| **RF / mmWave** | Hammerstad-Jensen impedance + Djordjevic-Sarkar causal dielectric + Cannonball-Huray conductor roughness for 60 GHz accuracy. FDTD time-stepper + Huygens NTFF for antenna pattern validation (only MIT-licensed NTFF in the open-source ecosystem). |
| **CAM** | 2.5D pocket clearing with cutter-radius inset. RTCP-style RS-274 G-code emission. |
| **OCL** | One file = one product. Lexer, parser, evaluator, LSP. |
| **Formats** | KiCad, Altium, Eagle, Gerber X2, ODB++, IPC-2581, SPICE, STL, OBJ, glTF, STEP, IGES, DXF, SVG, KCL, OpenSCAD, VTK, NASTRAN, Gmsh, Abaqus, OCD — 21 in total. |

## Calling the substrate

Three entry points:

- **MCP (agent-callable)** at <https://cad.openie.dev/mcp> — 60+ tools.
  See the [MCP Tool Reference](https://docs.openie.dev/mcp-tools.html).
- **REST API** at <https://cad.openie.dev/api> —
  see the [REST docs](https://docs.openie.dev/rest-api.html).
- **Interactive viewer** at <https://cad.openie.dev/viewer/kicad?session=ID>
  — live KiCad PCB viewer for any MCP session.

## License

Documentation in this repository is MIT. The substrate binary's
license is documented in the [release notes](https://docs.openie.dev/changelog.html).
