# CFD Analysis and Validation of the NACA 0015 Airfoil

A 2D RANS study of the NACA 0015 airfoil in ANSYS Fluent — from geometry and
meshing through verification (convergence, y⁺, mesh independence) to validation
against published experimental data.

**Status:** 🟡 In progress — verification stage.

---

## 1. Objective

Simulate the flow over a NACA 0015 airfoil and produce a **verified and
validated** prediction of its aerodynamic coefficients (Cl, Cd) and surface
pressure distribution (Cp), benchmarked against published data.

The emphasis is not only on *running* the simulation but on **demonstrating that
the result is trustworthy** — convergence, boundary-layer resolution (y⁺), mesh
independence, and validation against experiment.

## 2. Flow Conditions

| Parameter | Value |
|---|---|
| Airfoil | NACA 0015 (symmetric) |
| Chord, c | 1.0 m |
| Freestream velocity, U∞ | 17 m/s |
| Density, ρ | 1.225 kg/m³ |
| Dynamic viscosity, μ | 1.79 × 10⁻⁵ Pa·s |
| Reynolds number, Re | ≈ 1.16 × 10⁶ |
| Angle of attack, α | 0° (done) → sweep 0–16° (planned) |
| Flow regime | Incompressible, turbulent |

## 3. Geometry and Domain

- Airfoil generated from NACA 0015 coordinates; trailing edge closed to a single
  sharp vertex (single continuous curve, no merge required).
- C-type flow domain surrounding the airfoil.
- **Note / limitation:** current domain is relatively small; far-field distance to
  be increased for domain-independence (see Verification).

*(figure: domain + airfoil — `figures/domain.png`)*

## 4. Mesh

- **Type:** structured, block-decomposed (mapped quad cells).
- **Strategy:** near-wall clustering toward the airfoil; refinement at the
  leading and trailing edges.
- **y⁺ target:** ≈ 1 (wall-resolved, for k-ω SST).
  - First-cell height for y⁺ ≈ 1: **~2 × 10⁻⁵ m** (derived from Cf → τ_w → u_τ).
  - **Current status:** baseline mesh gives y⁺ ≈ 100 (wall-function range);
    near-wall mesh being refined to reach y⁺ ≈ 1.

*(figure: mesh overview + leading-edge close-up — `figures/mesh.png`)*

## 5. Solver Setup

| Setting | Choice | Rationale |
|---|---|---|
| Solver | Pressure-based, steady | Incompressible, steady flow |
| Turbulence model | k-ω SST | Resolves boundary layer; handles adverse gradients and separation |
| Boundary conditions | Velocity inlet, pressure outlet, symmetry far-field, no-slip wall | — |
| Reference values | Computed from inlet (ρ, U, A = c × 1 m) | Correct non-dimensionalization of Cl, Cd, Cp |
| Schemes | Second-order upwind | Accuracy |

## 6. Verification ("are we solving the equations right?")

### 6.1 Iterative convergence — ✅ PASSED
- Cl and Cd monitored vs iteration; both flat (converged).
- Residuals: continuity ~1×10⁻¹⁰, momentum/turbulence ~1×10⁻¹² to 10⁻¹⁴.
- **Cd = 0.0150**, **Cl ≈ 0** (as expected for a symmetric airfoil at 0°).

*(figure: convergence history — `figures/convergence.png`)*

### 6.2 y⁺ check — 🟡 in progress
- Baseline y⁺ ≈ 100 (log-law region, wall-function mode).
- Refining near-wall mesh to y⁺ ≈ 1 (wall-resolved) for accurate drag/separation.

*(figure: wall y⁺ distribution — `figures/yplus.png`)*

### 6.3 Mesh independence — ⬜ pending
- Plan: 3–4 meshes (coarse → fine), first-cell height held fixed (y⁺ ≈ 1),
  overall density varied; plot Cl/Cd vs cell count and confirm a plateau.

| Mesh | Cells | Cl | Cd |
|---|---|---|---|
| Coarse | — | — | — |
| Medium | — | — | — |
| Fine | — | — | — |

## 7. Physical Sanity Checks — ✅ PASSED

Independent of external data, from first principles:
- Cp = +1 at the leading-edge stagnation point.
- Cp ≤ 1 everywhere.
- Suction peak near the leading edge, pressure recovery toward the trailing edge.
- Upper and lower Cp identical at 0° → zero lift (symmetry confirmed).
- Lift dominated by pressure; viscous contribution negligible.

*(figure: Cp vs x/c — `figures/cp.png`)*

## 8. Validation ("are we solving the right equations?") — ⬜ pending

- Angle-of-attack sweep (0–16°) → Cl–α curve and drag polar.
- Benchmark against **Sheldahl & Klimas (SAND80-2114)** and/or **XFOIL** at matching Re.
- Compare: lift-curve slope, Cl_max, stall angle, Cd, Cp distribution.

## 9. Results (summary so far)

| Quantity | Value (α = 0°) | Status |
|---|---|---|
| Cl | ≈ 0 | ✅ consistent with symmetry |
| Cd | 0.0150 | 🟡 slightly high — expected to drop with y⁺≈1 mesh |

## 10. Key Learnings

- Verification (convergence + mesh independence + y⁺) must precede validation,
  so that any disagreement with experiment is physics, not numerical error.
- y⁺ sets boundary-layer resolution (fixed once); mesh independence varies overall
  density — two different requirements.
- A fully-turbulent model (k-ω SST without transition) over-predicts drag by
  ignoring the laminar run near the leading edge.

---

## Tools
ANSYS Fluent 2025 R2 (Student) · structured meshing · k-ω SST RANS.

## References
- Sheldahl, R.E. & Klimas, P.C. (1981), *Aerodynamic Characteristics of Seven
  Symmetrical Airfoil Sections*, SAND80-2114, Sandia National Laboratories.
- Abbott, I.H. & von Doenhoff, A.E., *Theory of Wing Sections*.
