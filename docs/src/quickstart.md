# Quickstart

Five minutes from zero to a routed PCB + a CAM toolpath + a coupled
multi-physics simulation, all via the hosted MCP at `cad.openie.dev`.

## Prerequisites

You need:
- An HTTP client (`curl` is fine).
- An MCP-capable agent (Claude Code, Claude Desktop, or any tool-use LLM).

## 1. Start a session

The first call returns a session ID. Reuse it on subsequent calls to keep
the per-session project in memory.

```bash
curl -s https://cad.openie.dev/mcp \
  -H 'Content-Type: application/json' \
  -d '{
    "jsonrpc":"2.0","id":1,
    "method":"tools/call",
    "params":{
      "name":"create_box",
      "arguments":{"width":10,"height":20,"depth":5}
    }
  }'
```

The response header includes `Mcp-Session-Id: <uuid>`; pass that as
`Mcp-Session-Id: ...` on every subsequent call.

## 2. Design a PCB

The complete aether-t-style flow looks like:

```text
import_kicad / import_zener           # ingest a design
  → run_placement                     # force-directed component placement
  → check_bga_escape_feasibility      # is the BGA routable on this tier?
  → run_autorouter(
      bgas=[{footprint_ref:"U8", ...}],
      length_match_groups=[{name:"DDR_DIQ1", ...}]
    )                                 # full routing pass
  → render_pcb_svg                    # static SVG preview
  → /viewer/kicad?session=<id>        # live interactive viewer
  → export_gerber / generate_jlcpcb_bom / generate_jlcpcb_cpl
```

Walk through every tool in the [MCP Tool Reference](./mcp-tools.md).

## 3. Run a CAM operation

Generate G-code for a rectangular pocket:

```bash
curl -s https://cad.openie.dev/mcp \
  -H 'Content-Type: application/json' \
  -H 'Mcp-Session-Id: YOUR_SESSION_ID' \
  -d '{
    "jsonrpc":"2.0","id":2,
    "method":"tools/call",
    "params":{
      "name":"generate_pocket_gcode",
      "arguments":{
        "width_mm": 50, "height_mm": 30, "depth_mm": 8,
        "cutter_diameter_mm": 6, "step_down_mm": 2
      }
    }
  }'
```

The result includes ready-to-run RS-274 G-code.

## 4. Run a coupled multi-physics simulation

Test which relaxation scheme converges fastest for a given coupling
stiffness:

```bash
curl -s https://cad.openie.dev/mcp \
  -H 'Content-Type: application/json' \
  -H 'Mcp-Session-Id: YOUR_SESSION_ID' \
  -d '{
    "jsonrpc":"2.0","id":3,
    "method":"tools/call",
    "params":{
      "name":"simulate_coupled",
      "arguments":{
        "coupling_alpha":0.9,"coupling_beta":0.95,
        "relaxation":"iqn_ils","iqn_history_max":10
      }
    }
  }'
```

The result reports iteration count, final residual, and the analytic
fixed point for comparison.

## Next

- [MCP Tool Reference](./mcp-tools.md) — every tool with its arguments.
- [Designing a PCB end-to-end](./recipes/aether-t-flow.md) — the long walkthrough.
- [Algorithm theory](./algorithms/lcis-bga-escape.md) — the substantive
  algorithms the substrate ships, with citations.
