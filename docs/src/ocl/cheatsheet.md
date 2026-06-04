# OCL Cheatsheet — OpenIE CAD Language

## Structure

```ocl
product "Product Name" {
  body BodyName { ... }
  sketch SketchName { ... }
  schematic SheetName { ... }
  pcb BoardName { ... }
  circuit CircuitName { ... }
  sim StudyName { ... }
  assembly AssemblyName { ... }
  bom { ... }
  firmware TaskName { ... }
}
```

## Variables & Types

```ocl
let width = 50mm          // immutable, with unit
var count = 4             // mutable integer
let name = "bracket"      // string
let active = true         // boolean
let parts = [1, 2, 3]    // array
let point = (10mm, 20mm) // tuple
let props = { k: 0.5, name: "steel" } // table
```

## Units (dimensional analysis enforced)

```
Length:    mm, cm, m, um, mil, in, ft
Angle:    deg, rad
Electrical: V, mV, A, mA, uA, ohm, kohm, F, pF, nF, uF, H, uH, mH
Frequency: Hz, kHz, MHz, GHz
Power:    W, mW
Temperature: C, K
```

## 3D Bodies

```ocl
body Bracket {
  let base = box(50mm, 30mm, 5mm)
  let hole = cylinder(3mm, 10mm)
  let ball = sphere(5mm)
  let pin = cone(2mm, 8mm)
  let ring = torus(10mm, 2mm)

  // Transform with pipe
  let moved = hole |> translate(25mm, 15mm, 0mm)
  let turned = base |> rotate(0deg, 0deg, 45deg)
  let big = base |> scale(2.0)
}
```

## Schematics

```ocl
schematic Main {
  let U1 = place ESP32(x: 40mm, y: 25mm)
  let R1 = place Resistor(x: 10mm, y: 10mm, value: "10k")
  let C1 = place Cap(x: 20mm, y: 10mm, value: "100nF")

  wire(U1.VCC, C1.1)
  wire(C1.2, GND)
  wire(U1.GPIO2, R1.1)

  net VCC { U1.VCC, C1.1 }
}
```

## PCB Layout

```ocl
pcb MainBoard {
  board(width: 80mm, height: 50mm, layers: 4)

  place U1(x: 40mm, y: 25mm, layer: "F.Cu", rotation: 0deg)
  place R1(x: 10mm, y: 10mm, layer: "F.Cu")

  track(net: "VCC", width: 0.25mm, layer: "F.Cu", points: [(0mm, 0mm), (10mm, 0mm)])
  via(x: 15mm, y: 20mm, drill: 0.3mm, net: "GND")
  zone(net: "GND", layer: "B.Cu", outline: [(0mm,0mm), (80mm,0mm), (80mm,50mm), (0mm,50mm)])
}
```

## Circuits (SPICE)

```ocl
circuit Amplifier {
  V1(VCC, GND) = 12V
  R1(VCC, base) = 100kohm
  R2(base, GND) = 47kohm
  Rc(VCC, collector) = 4.7kohm
  Re(emitter, GND) = 1kohm
  C1(input, base) = 10uF
  C2(collector, output) = 10uF
}
```

## Simulation

```ocl
sim ThermalStudy {
  type: steady_state
  ambient: 25C
  power(U1): 0.5W
  power(R1): 0.1W
}

sim ModalAnalysis {
  type: modal
  modes: 10
  material: aluminum
}

sim DropTest {
  type: transient
  duration: 5ms
  gravity: -9.81m/s2
}
```

## Assembly

```ocl
assembly Complete {
  place Shell(transform: identity)
  place Lid(transform: translate(0mm, 0mm, 25mm))
  place PCB(transform: translate(2mm, 2mm, 6mm))

  mate Lid_to_Shell {
    type: coincident
    face_a: Lid.bottom
    face_b: Shell.top
  }
}
```

## BOM

```ocl
bom {
  entry(reference: "U1", value: "ESP32-WROOM-32E", manufacturer: "Espressif", mpn: "ESP32-WROOM-32E")
  entry(reference: "R1", value: "10k 0402", manufacturer: "Yageo", mpn: "RC0402FR-0710KL")
}
```

## Control Flow

```ocl
// Conditionals
if width > 50mm {
  let result = "too wide"
} else {
  let result = "ok"
}

// Loops
for i in 0..10 {
  place Hole(x: i * 5mm, y: 0mm)
}

// Functions
fn slot(w: length, h: length, d: length) -> body {
  box(w, h, d)
}
```

## Pipe Operator

```ocl
// Chain transformations
let part = box(10mm, 20mm, 5mm)
  |> translate(5mm, 0mm, 0mm)
  |> rotate(0deg, 0deg, 45deg)
```
