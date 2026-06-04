# IQN-ILS partitioned multi-physics coupling

> Degroote, Bathe, Vierendeels — "Performance of a new partitioned
> procedure versus a monolithic procedure in fluid-structure
> interaction," Computers & Structures 2009.
>
> Chourdakis et al. — "preCICE v2: a sustainable and user-friendly
> coupling library," arXiv 2109.14470 (2021).

`cad-sim::coupling` ships the **only MIT/Apache implementation of
preCICE-style partitioned multi-physics coupling** in the
open-source ecosystem. preCICE itself is LGPLv3 — usable via
dynamic linking but not directly portable into MIT/Apache codebases —
so this module re-derives the algorithms from the public papers.

## The problem

When two physics domains share an interface (a heated solid wall
contacted by fluid; an oscillating airfoil in a transonic flow),
solving them jointly as one monolithic system is expensive and
inflexible. The **partitioned** approach keeps each solver
independent and exchanges only the interface field — temperature,
pressure, displacement — but introduces a numerical loop:

```text
loop k:
  x_a  = solve_A(x_b)        # A's response to B's interface
  x_b  = solve_B(x_a)        # B's response to A's update
  if ‖x_a^new − x_a^old‖ < tol: converged
```

For stiff couplings (large added-mass FSI, conjugate heat transfer
with mismatched conductivities) this loop diverges or crawls.
Acceleration schemes choose how to combine the raw solver outputs
into the next iterate.

## The algorithms

`cad-sim::coupling::RelaxationMode` exposes three schemes:

### Constant under-relaxation

`x_{k+1} = x_k + ω · r_k` with fixed ω ∈ (0, 1]. Always stable for
small enough ω but slow when the optimal ω is unknown a priori.

### Aitken's Δ²

Adapts ω from the previous two residuals (Mok 2001, used in preCICE):

```text
ω_new = -ω_old · (r_prev · (r − r_prev)) / ‖r − r_prev‖²
```

Mathematically free (one extra inner product per iteration), and
dramatically faster than constant ω on stiff couplings.

### IQN-ILS (Interface Quasi-Newton with Inverse Least Squares)

The preCICE workhorse for tight FSI and conjugate heat transfer.
Maintains column histories of past displacement and residual
differences `V = [Δr_1 … Δr_k]`, `W = [Δx_1 … Δx_k]`. On each
iteration:

1. Solve the small least-squares problem `min_c ‖V·c − (−r)‖²` via
   normal equations `(VᵀV + λI) c = −Vᵀr` with Tikhonov
   regularisation (`λ ≈ 10⁻¹²`).
2. Compute the IQN-ILS update `Δx = W·c`.
3. Apply: `x_{k+1} = x_k + Δx`.
4. Truncate the oldest column when history exceeds `history_max`.

## Substrate test (Wave 67.B)

On a stiff α = 0.9, β = 0.95 linear toy:

| Scheme | Iterations to ‖r‖ < 10⁻⁸ |
|---|---|
| Constant ω = 0.3 | many |
| Aitken | fewer |
| IQN-ILS (history_max = 20) | fewest (assertion: ≤ Aitken) |

## Calling it

```json
{
  "method": "tools/call",
  "params": {
    "name": "simulate_coupled",
    "arguments": {
      "coupling_alpha": 0.9,
      "coupling_beta": 0.95,
      "relaxation": "iqn_ils",
      "iqn_history_max": 10,
      "max_iterations": 200,
      "tolerance": 1e-8
    }
  }
}
```

Returns iteration count, final residual, and the analytic fixed
point for comparison — useful for choosing a relaxation scheme
**before** plugging real cad-sim domains in.

## Implementation notes

`cad-sim::coupling::couple` pre-primes `field_b = solve_b(field_a)`
before the loop when `RelaxationMode::IqnIls` is selected.
Without this, the first iteration's residual would lie off the
fixed-point manifold (because `field_b` started as `initial_b`,
not `B(initial_a)`), and the first (Δx, Δr) pair would poison the
least-squares solve. The Aitken and constant-ω schemes don't need
this since they're memoryless.

## References

- Degroote, Bathe, Vierendeels (2009) — original IQN-ILS derivation.
- Mok (2001) — the Aitken Δ² ω-update form preCICE uses.
- Chourdakis et al. (2021) — preCICE v2 architecture survey (arXiv 2109.14470).
