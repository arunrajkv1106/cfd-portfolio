
# CFD Analysis and Validation of the NACA 0015 Airfoil

A 2D RANS study of the NACA 0015 airfoil in ANSYS Fluent — from geometry and
meshing through verification to validation against published experimental data.

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
  sharp vertex .
- Rectangular (box) computational domain surrounding the airfoil, meshed with a
  structured block decomposition — diagonal blocks connecting the domain corners
  to the airfoil give six mapped quad blocks.

<img width="480" alt="Rectangular domain with six-block decomposition" src="https://github.com/user-attachments/assets/9081d3aa-4055-4008-9f60-f929962c9f25" />

*Figure 1: Rectangular computational domain with the six-block structured decomposition; NACA 0015 airfoil at centre.*

## 4. Mesh

> **Baseline mesh** — the first rung of the mesh-independence study (§6).
> Described here as *used*; its adequacy is confirmed by the independence
> study in §6.

- **Type:** block-decomposed domain; inflation layers near the wall,
  quad-dominant mesh in the outer field.
- **Streamwise resolution:** 150 divisions along the airfoil surface.

### Near-wall strategy (y⁺)
- **Target:** y⁺ ≈ 1 (wall-resolved, for k-ω SST). First-cell height
  ~2 × 10⁻⁵ m (from Cf → τ_w → u_τ).
- **Design finding:** a single biased radial edge cannot reach y⁺ ≈ 1 here.
  The far-field edge is L ≈ 2.5 m, so a 2 × 10⁻⁵ m first cell demands a span
  ratio L/h ≈ 125,000, forcing bias factor ~22,600 — beyond what the mapped
  mesher builds cleanly (mapping failed).
- **Resolution:** inflation layers decouple wall resolution from the far field.
  The radial (wall-normal) spacing is set by the inflation control — **not** by
  edge divisions:
  - First layer height **2 × 10⁻⁵ m** (fixes y⁺ ≈ 1), growth rate **1.2**,
    **20 layers** (covers the near-wall boundary layer; reduced from 28 to ease
    trailing-edge layer collision — see TE treatment below).
  - The old biased radial Edge Sizing is **suppressed**; inflation replaces it.
    Streamwise resolution is unchanged (Edge Sizing 2, 150 along the surface).

<img width="521" height="300" alt="naca0015_yplus" src="https://github.com/user-attachments/assets/83072628-4795-435c-9e3d-38271f17c904" />

*Fig: Wall y⁺ along the chord — below 1 everywhere (≈0.88 at LE, ≈0.14 at TE). Wall-resolved.*

### Mesh quality
| Metric | Min | Max | Avg |
|---|---|---|---|
| Orthogonal Quality | 0.011 | 1.00 | 0.95 |
| Skewness | ~0 | 0.9999 | 0.17 |
| Aspect Ratio | 1 | 371 | 37 |

Average quality is excellent. The low OQ-min / high skew-max come from a few
cells at the sharp trailing-edge cusp (see TE treatment below). Negligible area;
did not corrupt the solution (Cp is smooth to the TE).

### Trailing-edge treatment
The TE was closed to a sharp cusp (≈ 0° wedge). Inflation layers from the upper
and lower surfaces collide in that wedge and form a few near-degenerate cells
(OQ ≈ 0.01, skew ≈ 1.0). Mitigations applied:
- Reduced layers **28 → 20** (shrinks the collision zone).
- Collision avoidance = **Stair Stepping** — drops layers locally in the wedge
  instead of crushing them into slivers.

A few slivers remain — inherent to a zero-thickness cusp. **Accepted for this
baseline**: affected area is negligible and the solution is clean there (smooth
Cp, no TE artefact). *Future improvement:* restore the physical ~0.3 %-chord
blunt TE, which removes the cusp entirely.

### Key meshing decisions (base case)
| Decision | Choice | Reason |
|---|---|---|
| Near-wall method | Inflation layers | One biased radial edge can't reach y⁺ ≈ 1 on a 2.5 m domain (L/h ≈ 125 000 → bias ~22 600 → mapper fails) |
| First layer / growth / layers | 2 × 10⁻⁵ m / 1.2 / 20 | y⁺ ≈ 1 with gentle growth; 20 eases TE collision |
| TE collision | Stair stepping | Avoids crushed cells in the sharp-TE wedge |
| Streamwise | 150 (Edge Sizing 2) | Surface resolution, unchanged |
| Outer field | Quad-dominant, body size ~0.08 m | Coarse where gradients are small |

### Baseline result (α = 0°)
- U = 17 m/s, Re ≈ 1.16 × 10⁶, k-ω SST, second-order upwind. Converged.
- **C_d = 0.0118** (pressure 0.0041 + viscous 0.0077 — friction-dominated).
- **C_l = 0.0062** (ideal 0 by symmetry; small residual from the unstructured
  outer mesh not being perfectly mirror-symmetric).
- Cp smooth and physical: stagnation Cp ≈ 1 at LE, suction peak ≈ −0.55 at
  x/c ≈ 0.13, clean recovery to TE.

<img width="521" height="300" alt="naca0015_cp" src="https://github.com/user-attachments/assets/5e0a0d3b-500a-4a34-a08d-fe5d35682eff" />

*Fig: Surface Cp at α = 0° — smooth, no trailing-edge artefact.*

### Next
- Mesh independence · AoA sweep (0–16°) · validation (Sheldahl / XFOIL)

## 5. Solver Setup

| Setting | Choice | Rationale |
|---|---|---|
| Solver | Pressure-based, steady | Incompressible, steady flow |
| Turbulence model | k-ω SST | Resolves boundary layer; handles adverse gradients and separation |
| Boundary conditions | Velocity inlet, pressure outlet, symmetry far-field, no-slip wall | — |
| Reference values | Computed from inlet (ρ, U, A = c × 1 m) | Correct non-dimensionalization of Cl, Cd, Cp |
| Schemes | Second-order upwind | Accuracy |
| Initialization | Hybrid + external-aero favorable settings | Smooth, geometry-aware start; robust with fragile TE cells |

## 6. Verification ("are we solving the equations right?")

### 6.1 Iterative convergence — ✅ PASSED
- Cl and Cd monitored vs iteration; both flat (converged).
- Residuals steadily decreasing (continuity ~10⁻⁵, momentum/turbulence lower).
- **Cd = 0.0118**, **Cl = 0.0062** (≈ 0, as expected for a symmetric airfoil at 0°).

*(figure: convergence history — screenshot to add)*

### 6.2 y⁺ check — ✅ PASSED
- Wall y⁺ **< 1 everywhere** (≈ 0.88 at LE, ≈ 0.14 at TE) → wall-resolved.
- Achieved via inflation layers (first cell 2 × 10⁻⁵ m); see §4 for the
  near-wall strategy and the y⁺ plot.

### 6.3 Mesh independence — ⬜ pending
- Plan: 3–4 meshes (coarse → fine), first-cell height held fixed (y⁺ ≈ 1),
  overall density varied; plot Cl/Cd vs cell count and confirm a plateau.

| Mesh | Cells | Cl | Cd |
|---|---|---|---|
| Coarse (baseline) | — | 0.0062 | 0.0118 |
| Medium | — | — | — |
| Fine | — | — | — |

**Selected mesh:** to be set after the study — the baseline if Cd is already
grid-independent, otherwise the refined mesh. This is the mesh used for all
results in §9.

## 7. Physical Sanity Checks — ✅ PASSED

Independent of external data, from first principles:
- Cp = +1 at the leading-edge stagnation point.
- Cp ≤ 1 everywhere.
- Suction peak near the leading edge, pressure recovery toward the trailing edge.
- Upper and lower Cp nearly identical at 0° → Cl ≈ 0 (a small residual, ~0.006,
  remains from the unstructured outer mesh not being perfectly mirror-symmetric).
- Lift dominated by pressure; viscous contribution negligible.

*(figure: Cp vs x/c — `figures/cp.png`)*

## 8. Validation ("are we solving the right equations?") — ⬜ pending

- Angle-of-attack sweep (0–16°) → Cl–α curve and drag polar.
- Benchmark against **Sheldahl & Klimas (SAND80-2114)** and/or **XFOIL** at matching Re.
- Compare: lift-curve slope, Cl_max, stall angle, Cd, Cp distribution.

## 9. Results (summary so far)

| Quantity | Value (α = 0°) | Status |
|---|---|---|
| Cl | 0.0062 (≈ 0) | ✅ consistent with symmetry (small mesh-asymmetry residual) |
| Cd | 0.0118 | 🟡 slightly high — fully-turbulent model + compact domain |

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
