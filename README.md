# PL-Accelerated-Nesterov-Lean

A Lean 4 formalization of accelerated Nesterov convergence under a local
Polyak-Łojasiewicz condition near a smooth minimizer manifold.

The proof is **sorry-free**.

## Main Results

The embedded-manifold theorem applies to any L-smooth C² function
f : ℝ^d → ℝ whose argmin set is a C² embedded k-submanifold, including the
endpoint cases k = 0 and k = d, where f satisfies the μ-PL condition on an open
neighborhood U containing the minimizer manifold.
The C³ public theorem assumes f is C³ on an open neighborhood of `argminSet f`
and constructs the minimizer geometry internally.

The formalization proves a local modified-Nesterov convergence theorem with the
sharp exponent `c = 1` and explicit prefactor `2`:

```
f(xₖ) - f⋆ ≤ 2 · exp(-k / √(L / μ)) · (f(x₀) - f⋆)
```

### Public theorem

| theorem | sequence | parameter choice | prefactor |
| --- | --- | --- | --- |
| `nesterov_pl_accelerated_rate` | state positions `xₖ` | existential momentum parameter `ρ` | `2` |
| `nesterov_pl_accelerated_rate_c3` | state positions `xₖ` | existential momentum parameter `ρ` | `2` |

The local versions live in `PLAcceleratedNesterovLean/Convergence/NesterovConvergence.lean`:

| theorem | role |
| --- | --- |
| `nesterov_convergence_at_base_point_position_params` | reusable local core with arbitrary `μ'`, `θ`, `ρ`, and an assumed scalar rate bound |
| `nesterov_convergence_at_base_point_position_theta` | local specialized theorem using `rhoOfTheta` |

The public file uses local syntax for readability:
`R^d = EuclideanSpace ℝ (Fin d)`, `PolyakLojasiewicz(f, μ)[U]` expands to the
raw first-order PL conjunction, and
`C2Manifold(M, k)`/`C2Embedding(ι, k, d)` abbreviate the C²
manifold/embedding hypotheses.

**Embedded-manifold public statement**:
```lean
theorem nesterov_pl_accelerated_rate
    {d : ℕ}
    (L : ℝ≥0)
    (μ : ℝ≥0) :
    ∃ ρ : ℝ,
    ∀ (f : (R^d) → ℝ),
    ∀ (k : ℕ),
    ∀ (M : Type*) [TopologicalSpace M] [ChartedSpace (R^k) M]
      [C2Manifold(M, k)] [Nonempty M]
      (ι : M → (R^d)),
      C2Embedding(ι, k, d) →
      Set.range ι = argminSet f →
    ∀ (U : Set (R^d)),
      IsOpen U →
      Set.range ι ⊆ U →
      ContDiffOn ℝ 2 f U →
      PolyakLojasiewicz(f, μ)[U] →
      LipschitzOnWith (↑L) (gradient f) U →
    ∃ (Ū : Set (R^d)),
      IsOpen Ū ∧ Set.range ι ⊆ Ū ∧ Ū ⊆ U ∧
      ∀ x₀ ∈ Ū,
        ∀ t,
          (nesterovSeqGen f (1 / ↑L) ρ ⟨x₀, 0⟩ t).x ∈ U ∧
          (nesterovSeqGen f (1 / ↑L) ρ ⟨x₀, 0⟩ t).lookahead
            (1 / ↑L) ∈ U ∧
          f ((nesterovSeqGen f (1 / ↑L) ρ ⟨x₀, 0⟩ t).x) - fStar f ≤
            2 * Real.exp (-(↑t / Real.sqrt (↑L / μ))) * (f x₀ - fStar f)
```

**C³ public statement**:
```lean
theorem nesterov_pl_accelerated_rate_c3
    {d : ℕ}
    (L : ℝ≥0)
    (μ : ℝ≥0) :
    ∃ ρ : ℝ,
    ∀ (f : (R^d) → ℝ),
    ∀ (U : Set (R^d)),
      IsOpen U →
      argminSet f ⊆ U →
      ContDiffOn ℝ 3 f U →
      PolyakLojasiewicz(f, μ)[U] →
      LipschitzOnWith (↑L) (gradient f) U →
    ∃ (Ū : Set (R^d)),
      IsOpen Ū ∧ argminSet f ⊆ Ū ∧ Ū ⊆ U ∧
      ∀ x₀ ∈ Ū,
        ∀ t,
          (nesterovSeqGen f (1 / ↑L) ρ ⟨x₀, 0⟩ t).x ∈ U ∧
          (nesterovSeqGen f (1 / ↑L) ρ ⟨x₀, 0⟩ t).lookahead
            (1 / ↑L) ∈ U ∧
          f ((nesterovSeqGen f (1 / ↑L) ρ ⟨x₀, 0⟩ t).x) - fStar f ≤
            2 * Real.exp (-(↑t / Real.sqrt (↑L / μ))) * (f x₀ - fStar f)
```

Key features of the formalization:
- **Exact rate** with the PL constant μ and prefactor `2`.
- **C² embedded-manifold theorem** with endpoint dimensions k = 0 and k = d.
- **Open-neighborhood formulation**: the proof constructs the tubular
  sub-neighborhood internally.
- **C³ alternative**: `nesterov_pl_accelerated_rate_c3` derives the minimizer
  geometry from `ContDiffOn ℝ 3 f U` and `argminSet f ⊆ U`.
- **Dimension-independent algorithm**: the iteration is independent of the
  submanifold dimension k.
- **Direct formal bound**:
  `f(xₖ)-f⋆ ≤ 2 * exp(-k / sqrt (L / μ)) * (f(x₀)-f⋆)`.
- **Existential momentum**: the retuning choice is packaged as `∃ ρ`.

## Project Structure

The repository name is `PL-Accelerated-Nesterov-Lean`; Lean modules use the
valid identifier namespace/directory `PLAcceleratedNesterovLean`.

```
PLAcceleratedNesterovLean/
├── Core/
│   ├── Defs.lean                  # Core definitions (E d, HasAcceleratedRate, etc.)
│   ├── NesterovScheme.lean        # FirstOrderAlgorithm, Nesterov state machine
│   ├── NesterovSeqGen.lean        # State-based Nesterov sequence (nonzero velocity)
│   └── EmbeddedManifold.lean      # Smooth embeddings, tubular neighborhoods
├── MorseBott/                     # PL → Morse-Bott bridge, tubular projections
├── Convergence/
│   ├── LocalGeometry/             # Local Hessian/aiming/growth bounds
│   ├── Coercivity/                # Lyapunov coercivity
│   ├── MotionError/               # Gradient/velocity/step bounds
│   ├── LyapunovContraction/       # One-step Lyapunov contraction
│   ├── CurvAbsorb/                # Curvature absorption
│   ├── Bootstrap/                 # Geometric decay bootstrap
│   ├── StateContraction/          # State-based aux variable recursion
│   ├── LocalArgument.lean         # Per-base-point convergence assembly
│   ├── GenLocalArgument.lean      # State-based gen local convergence
│   ├── NesterovConvergence.lean   # θ-retuned local convergence
│   ├── ConvergenceHelpers.lean    # Auxiliary convergence helpers
│   ├── PhaseSchedule.lean         # Retuned parameter definitions
│   └── RateArithmetic.lean        # Geometric → exponential rate conversion
└── MainTheorem.lean               # Main theorem
```

## Building

Requires Lean 4 (v4.28.0) and Mathlib. Build with:

```
lake build
```

## License

This repository is licensed under the MIT License; see `LICENSE`.

## Citation

If you cite this companion formalization, cite the repository artifact:

```bibtex
@software{placceleratednesterovlean2026,
  author = {Obreiter, Max and Steinbrecher, Tobias and F{\"o}rster, Robert},
  title = {{PL-Accelerated-Nesterov-Lean}: Lean Formalization of Accelerated Nesterov Convergence under a Local Polyak--{\L}ojasiewicz Condition},
  year = {2026},
  url = {https://tobotis.github.io/PL-Accelerated-Nesterov/},
  note = {Companion Lean 4 formalization}
}
```

## References

- Gupta, K. & Wojtowytsch, S. (2025). *Nesterov acceleration in benignly non-convex landscapes.* ICLR 2025.
- Rebjock, Q. & Boumal, N. (2024). *Fast convergence to non-isolated minima.* Mathematical Programming.
