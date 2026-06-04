# OCL — openie CAD Language

OCL ("openie CAD Language") is a Rust-flavored declarative language for
describing products end-to-end. One `.ocl` file = one product:
mechanical body, schematic, PCB, simulations.

```ocl
product "Widget" {
  body Case {
    let base = box(50mm, 30mm, 10mm)
    let hole = cylinder(3mm, 15mm) |> translate(25mm, 15mm, 0mm)
    Case = base - hole
  }
  schematic Main {
    let U1 = place MCU(x: 20mm, y: 15mm)
    wire(U1.VCC, VCC)
  }
  circuit Power {
    V1(VCC, GND) = 3.3V
    R1(VCC, LED) = 330ohm
  }
}
```

The language enforces **dimensional analysis** — every numeric quantity
carries its unit, and the type system rejects expressions that mix
incompatible units (e.g. adding millimeters to ohms).

## Pages

- [Cheatsheet](./cheatsheet.md) — quick syntax reference.

## Toolchain

`cad-ocl` (lexer + parser + evaluator) and `cad-ocl-lsp` (Language Server)
ship in the private substrate. The MCP tools `parse_ocl` and `eval_ocl`
expose them over the network — see the
[MCP tool reference](../mcp-tools.md).
