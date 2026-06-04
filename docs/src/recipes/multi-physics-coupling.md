# Multi-physics coupling

`cad-sim` ships 38 physics domains; Wave 67 added the **partitioned
coupling scheduler** that lets two of them exchange a shared
interface field iteratively. This recipe walks through choosing a
coupling scheme + relaxation mode for your stiffness regime
*before* you wire real solvers in.

See the [IQN-ILS algorithm page](../algorithms/iqn-ils-coupling.md)
for the math + paper citations.

## When you need this

When two physics domains touch at an interface and the response of
one feeds back into the input of the other:

- **Conjugate heat transfer** — solid wall heated by adjacent fluid.
- **Fluid-structural interaction** — flexible airfoil in transonic flow.
- **Electromagnetic-thermal** — induction-heated piece.
- **Piezoelectric-mechanical** — sensor strain feeding voltage.

## Picking a scheme

Coupling stiffness is roughly the product of the two solvers'
sensitivities to each other's interface output. `simulate_coupled`
exposes a two-parameter linear toy `α × β` so you can test your
stiffness regime against each relaxation scheme.

```json
{
  "method": "tools/call",
  "params": {
    "name": "simulate_coupled",
    "arguments": {
      "coupling_alpha": 0.9,
      "coupling_beta": 0.95,
      "scheme": "implicit",
      "relaxation": "iqn_ils",
      "iqn_history_max": 10,
      "max_iterations": 200,
      "tolerance": 1e-8
    }
  }
}
```

Returns `Converged in N iterations, final residual …` plus the
analytic fixed point so you can verify behaviour.

## Which relaxation when

| Regime | Recommended | Why |
|---|---|---|
| αβ < 0.3 (weak) | `explicit` | One solver pass per side per step. Cheapest, stable. |
| αβ ∈ [0.3, 0.7] | `constant` ω = 0.5 | Sub-Newton, robust. |
| αβ > 0.7 (stiff FSI, CHT) | `aitken` or `iqn_ils` | Sub-linear vs. linear convergence rate. |
| Tight FSI, added-mass dominated | `iqn_ils` history 10-20 | preCICE's workhorse for the hardest cases. |

The [Wave 67.B test](../algorithms/iqn-ils-coupling.md#substrate-test-wave-67b)
shows IQN-ILS finishes in ≤ Aitken iterations on a stiff α=0.9, β=0.95
case — the assertion the substrate verifies on every build.

## Wiring real solvers (Wave 67.C preview)

The current MCP tool runs the canonical linear toy so you can
characterise convergence behaviour. Wave 67.C will accept real
cad-sim domain handles — `thermal`, `structural`, `cfd` — and loop
them through the same scheme + relaxation choice you picked on the
toy. Same convergence properties; just plug different solvers in.
