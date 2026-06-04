# Designing a PCB end-to-end (aether-t flow)

This recipe walks through the substrate's flagship PCB flow on
a board class that historically broke open-source tooling.

**aether-t** is a real-world design: 65 × 56.5 mm 6-layer with **8
wireless radios + PCIe Gen 3 + DDR data bus + 60 GHz mmWave radar**.
The 06-02 substrate audit found a Python-pipeline implementation
stuck at "rev 0.12, 162 of ~350 tracks routed, 140 dangling pads,
20 DRC violations" — every named failure mode traced to a missing
algorithm in the open-source toolchain. Waves 62-70 of openie-cad
closed each one in turn.

This recipe is what the agent-callable equivalent looks like.

## Prerequisites

- An HTTP client and an `Mcp-Session-Id` from any prior `tools/call`
  on `https://cad.openie.dev/mcp` (see the [Quickstart](../quickstart.md)).

## 1. Ingest the design

Two options. If you have a KiCad project:

```json
{
  "method": "tools/call",
  "params": {
    "name": "import_kicad",
    "arguments": {
      "schematic_source": "...",
      "pcb_source": "..."
    }
  }
}
```

If you have a Zener (`pcb-sch` JSON) source-of-truth:

```json
{
  "method": "tools/call",
  "params": {
    "name": "import_zener_with_layout",
    "arguments": { "zener_source": "...", "kicad_pcb_source": "..." }
  }
}
```

## 2. Validate the BGA escape *up front*

The aether-t pipeline hit "BGA fanout impossible" only after dozens
of failed routing attempts. The substrate fails-fast:

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

Returns `INFEASIBLE` with the exact geometric reason. Switch
`"fab_tier"` to `"hdi"` to see the via-in-pad path. See the
[LCIS BGA escape algorithm](../algorithms/lcis-bga-escape.md) for
the substrate's implementation of Kong/Yan/Ozdal ICCAD 2007.

## 3. Place the components

Replaces aether-t's hand-coded 800-line `auto_place.py`:

```json
{
  "method": "tools/call",
  "params": {
    "name": "run_placement",
    "arguments": {
      "iterations": 200,
      "locked": ["J1", "J8", "J2"]
    }
  }
}
```

Force-directed Fruchterman-Reingold iterations pull net-connected
footprints toward shared centroids and push overlapping footprints
apart. `locked` pins board-edge connectors.

## 4. Route the board

The headline call — composes BGA escape, length-match groups,
PathFinder negotiation, SALT topology, and the general A* routing
engine in one MCP invocation:

```json
{
  "method": "tools/call",
  "params": {
    "name": "run_autorouter",
    "arguments": {
      "epsilon": 0.5,
      "negotiation": true,
      "bgas": [{
        "footprint_ref": "U8",
        "pitch_mm": 0.4,
        "pad_diameter_mm": 0.2,
        "fab_tier": "hdi",
        "inner_layers": ["In1Cu", "In2Cu"],
        "buses": { "DDR_DIQ1": ["DIQ1_0", "DIQ1_1", "DIQ1_2", "DIQ1_3"] },
        "strategy": "via_in_pad"
      }],
      "length_match_groups": [{
        "name": "DDR_DIQ1",
        "nets": ["DIQ1_0", "DIQ1_1", "DIQ1_2", "DIQ1_3"],
        "tolerance_mm": 0.127
      }]
    }
  }
}
```

## 5. Check the impedance at frequency

For RF nets (the 60 GHz BGT60TR13C feed, the LoRa front-end, etc.)
the substrate's mmWave-correct impedance check:

```json
{
  "method": "tools/call",
  "params": {
    "name": "check_impedance_at_frequency",
    "arguments": {
      "trace_width": 0.55,
      "dielectric_height": 0.254,
      "frequency_ghz": 60,
      "material": "ro4350b",
      "roughness": "vlp"
    }
  }
}
```

See [mmWave loss models](../algorithms/mmwave-loss-models.md) for
the Djordjevic-Sarkar + Cannonball-Huray pair of refinements.

## 6. Inspect the result

Static SVG:

```json
{ "method":"tools/call",
  "params":{"name":"render_pcb_svg","arguments":{}} }
```

Interactive in the browser (KiCanvas):

```text
https://cad.openie.dev/viewer/kicad?session=<your-session-id>
```

## 7. Ship it

```json
{ "method":"tools/call", "params":{"name":"export_gerber","arguments":{}} }
{ "method":"tools/call", "params":{"name":"generate_jlcpcb_bom","arguments":{}} }
{ "method":"tools/call", "params":{"name":"generate_jlcpcb_cpl","arguments":{}} }
```

## Where each piece came from

| Step | Wave | Algorithm |
|---|---|---|
| BGA escape feasibility + routing | 63.A-F | [Kong/Yan/Ozdal LCIS](../algorithms/lcis-bga-escape.md) |
| Force-directed placement | 66 | Fruchterman-Reingold |
| Negotiation routing | 55 | [McMurchie-Ebeling PathFinder](../algorithms/pathfinder-negotiation.md) |
| Multi-pin topology | 58/59 | [SALT shallow-light tree](../algorithms/salt-shallow-light.md) |
| Length-match groups | 64.A-D | Fang DAC 2024 + capacity-aware insertion |
| 60 GHz impedance | 65 | [Djordjevic-Sarkar + Cannonball-Huray](../algorithms/mmwave-loss-models.md) |
| KiCanvas viewer | 61 | Thea Flowers' KiCanvas embedded via session route |
| Fabrication export | (legacy) | Gerber X2 + ODB++ + JLCPCB BOM/CPL |
