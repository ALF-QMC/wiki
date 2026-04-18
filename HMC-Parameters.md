# HMC Parameters

Tuning the Hybrid Monte Carlo (HMC) updating scheme in ALF. HMC is available for models with continuous auxiliary fields (`OP_V%type=3`).

> **Reference:** The algorithm is derived in the [documentation (PDF)](https://alf.physik.uni-wuerzburg.de/doc.pdf), Section on Hybrid Monte Carlo.

## Parameters

All HMC parameters are set in the `&VAR_QMC` namelist in the `parameters` file.

| Parameter | Default | Type | Description |
|-----------|---------|------|-------------|
| `HMC` | `.false.` | Logical | Enable HMC updates |
| `Delta_t_Langevin_HMC` | `0.0` | Real | Leap-frog integration step size $\delta t_m$ |
| `Leapfrog_Steps` | `0` | Integer | Number of leap-frog steps $N_\text{LF}$ per trajectory |
| `N_HMC_sweeps` | `1` | Integer | Number of HMC trajectories between sequential sweeps |
| `Sequential` | `.true.` | Logical | Keep sequential single-site updates active alongside HMC |

The **trajectory length** is $T_m$ = `Delta_t_Langevin_HMC` × `Leapfrog_Steps`.

### Example

```fortran
&VAR_QMC
HMC                    = .true.
Sequential             = .true.
Delta_t_Langevin_HMC   = 0.1d0
Leapfrog_Steps         = 10
N_HMC_sweeps           = 5
NSweep                 = 100
NBin                   = 20
Nwrap                  = 10
/
```

## Guidelines

### Step Size and Acceptance Rate

The leap-frog integrator introduces an $\mathcal{O}(\delta t_m^2)$ discretization error in the Hamiltonian. The Metropolis accept/reject step corrects for this, but large errors lead to low acceptance.

- **Target acceptance rate: ~70–80%.**
- Check the acceptance rate in the `info` file after a run (`Acceptance_HMC`).
- If acceptance is too low: **reduce** `Delta_t_Langevin_HMC`.
- If acceptance is very high (>95%): you're being too conservative — **increase** `Delta_t_Langevin_HMC` or `Leapfrog_Steps` to explore phase space faster.

### Trajectory Length

- The trajectory length $T_m$ controls how far each HMC proposal moves in configuration space.
- Too short: proposals are close to the current state (inefficient, like random walk).
- Too long: computational cost per trajectory is high without proportional gain.
- A reasonable starting point is $T_m \approx 1$.

### Combining with Sequential Updates

**Always keep `Sequential = .true.` when using HMC.** The documentation explicitly warns:

> *"In order to address potential ergodicity issues it is crucial to integrate both sequential and hybrid molecular dynamics moves, rather than relying solely on hybrid molecular dynamics."*

`N_HMC_sweeps` controls how many HMC trajectories are performed between sequential sweeps. Increasing this value shifts the balance toward HMC moves.

## Mass Matrix (`Apply_B_HMC`)

The HMC Hamiltonian is:

$$H(\mathbf{p}, \mathbf{s}) = \frac{1}{2} \mathbf{p}^T M^{-1} \mathbf{p} + S(\mathbf{s})$$

where $M$ is a positive-definite mass matrix. It is factored as $M^{-1} = B^T B$, with the constraint $\text{Ker}(B) = \{0\}$.

**By default, $M = \mathbf{1}$ (identity)** — the base implementation of `Apply_B_HMC` in the code is a no-op.

A non-trivial mass matrix can **precondition** the dynamics:
- Assign larger mass to fast modes → they evolve slowly in HMC time.
- Assign smaller mass to slow modes → they evolve faster.
- This can significantly reduce autocorrelation times.

To use a custom mass matrix, override the `Apply_B_HMC` subroutine in your Hamiltonian submodule. It receives a force/momentum array and a logical flag `ltrans`:
- `ltrans = .false.`: apply $B$ (used in momentum updates)
- `ltrans = .true.`: apply $B^T$ (used in position updates)

> **Note:** No shipped Hamiltonians currently override `Apply_B_HMC`. This is an advanced feature for users who understand the mode structure of their model.

## Validation

The documentation provides a validation benchmark: 6-site Hubbard chain at $\beta t = 4$:

| Method | $\langle H \rangle$ |
|--------|---------------------|
| HMC only | $-2.81715 \pm 0.00023$ |
| Discrete sequential | $-2.81706 \pm 0.00013$ |

Use small exact-diagonalization benchmarks like this to validate your HMC setup before production runs.

## Known Pitfalls

- **HMC requires continuous fields.** If any interaction vertex has `OP_V%type ≠ 3`, the code will terminate with an error. Set `Continuous = .true.` in your Hamiltonian namelist (e.g. `&VAR_Hubbard`).
- **Forgetting to enable HMC.** Setting `Delta_t_Langevin_HMC` and `Leapfrog_Steps` without `HMC = .true.` has no effect.
- **Zero defaults.** Both `Delta_t_Langevin_HMC` and `Leapfrog_Steps` default to `0` — you must set both to nonzero values.
- **Langevin vs. HMC.** The `Delta_t_Langevin_HMC` parameter is shared between Langevin and HMC dynamics. Do not enable both simultaneously unless you understand the interplay.
