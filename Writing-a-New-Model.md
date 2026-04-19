# Writing a New Model

How to implement a custom Hamiltonian in ALF using the template system.

## Overview

ALF uses Fortran submodules to define Hamiltonians. Each model is a submodule of `Hamiltonian_main` that overrides a set of procedures from the `ham_base` type. A template file and an auto-generation system make the process straightforward.

## Design Decisions

Before writing code, settle these key questions — they shape the entire structure of your model.

### Flavors (`N_FL`) and Colors (`N_SUN`)

Every model must set `N_FL` and `N_SUN`. These are defined in the base Hamiltonian but must be specified by each model, either hard-coded or via the parameter input file.

- **`N_FL`** (number of flavors): Propagation is block-diagonal in flavor — each flavor has its own Green function. For a spin-½ model without SU(2) symmetry (e.g., Hubbard with a magnetic field), use `N_FL = 2`. If your Hamiltonian preserves SU(N) symmetry across flavors, use `N_FL = 1` and encode the degeneracy through `N_SUN`.

- **`N_SUN`** (number of colors): Propagation is color-independent — all colors share the same Green function, so the cost scales with `N_FL`, not `N_FL × N_SUN`. For a standard SU(2)-symmetric Hubbard model, `N_FL = 1, N_SUN = 2` is the efficient choice.

The rule of thumb: **use the smallest `N_FL` your symmetry allows**, because each additional flavor multiplies the computational cost. Colors (`N_SUN`) are essentially free.

| Scenario | `N_FL` | `N_SUN` | Example |
|----------|:------:|:-------:|---------|
| SU(2)-symmetric spin-½ | 1 | 2 | Hubbard at half filling |
| Broken SU(2) (e.g., magnetic field) | 2 | 1 | Hubbard with `Mz = .true.` |
| SU(N) symmetric | 1 | N | SU(N) Kondo, SU(N) Hubbard |
| Multi-band with independent bands | N_bands | 1 or 2 | Multi-orbital models |

### Discrete vs. Continuous Auxiliary Fields

ALF supports both discrete Hubbard-Stratonovich (HS) fields (Ising-like, ±1) and continuous fields. Discrete fields are simpler to implement and work with sequential updates out of the box. Continuous fields require implementing the bosonic action `S0` and the corresponding force for Langevin/HMC updates — see the full Hubbard model for an example.

### Lattice and Observables

Use the [[Predefined Lattices]] and [[Predefined Observables]] whenever possible. They produce output in the standard format that the analysis tools expect. Custom lattices and observables are straightforward to add but require more boilerplate.

## Step-by-Step

### 1. Copy the Template

```bash
cd Prog/Hamiltonians/
cp 'Hamiltonian_##NAME##_smod.F90' Hamiltonian_MyModel_smod.F90
```

Replace **every** occurrence of `##NAME##` with `MyModel` in the new file.

### 2. Register the Model

Add the model name to `Prog/Hamiltonians.list`:

```
MyModel
```

One name per line. The build system looks for `Hamiltonians/Hamiltonian_MyModel_smod.F90` by default. To use a different path, specify it after the name:

```
MyModel path/to/Hamiltonian_MyModel_smod.F90
```

### 3. Declare Parameters

Define your model's parameters between the marker comments. The build system's `parse_ham.py` script reads these markers and auto-generates the `read_parameters()` and `write_parameters_hdf5()` subroutines:

```fortran
!#PARAMETERS START# VAR_MyModel
real(Kind=Kind(0.d0)) :: ham_T  = 1.d0   ! Hopping parameter
real(Kind=Kind(0.d0)) :: Ham_U  = 4.d0   ! Interaction strength
!#PARAMETERS END#
```

This creates a `&VAR_MyModel` namelist that users set in their `parameters` file:

```fortran
&VAR_MyModel
ham_T = 1.0d0
Ham_U = 4.0d0
/
```

### 4. Implement the Required Subroutines

The type declaration in the template specifies which procedures to override:

```fortran
type, extends(ham_base) :: ham_MyModel
contains
  procedure, nopass :: Ham_Set
  procedure, nopass :: Alloc_obs
  procedure, nopass :: Obser
  procedure, nopass :: ObserT
end type ham_MyModel
```

#### `Ham_Set` — Set Up the Model

Called once at initialization. Responsibilities:

1. Call `read_parameters()` (auto-generated)
2. Compute `Ltrot = nint(Beta/Dtau)` (and `Thtrot` if projective)
3. Set up the Bravais lattice (see [[Predefined Lattices]])
4. Define hopping operators `Op_T(:,:)` — kinetic energy
5. Define interaction operators `Op_V(:,:)` — Hubbard-Stratonovich decomposition
6. Initialize auxiliary fields `nsigma`
7. Set trial wave functions `WF_L`, `WF_R` (projective QMC only)

The propagation on a time slice reads:

$$\prod_{n=1}^{N_V} e^{V_n(\tau)} \prod_{n=1}^{N_T} e^{T_n}$$

With the `Symm` option enabled, it becomes:

$$\prod_{n=N_T}^{1} e^{T_n/2} \prod_{n=1}^{N_V} e^{V_n(\tau)} \prod_{n=1}^{N_T} e^{T_n/2}$$

#### `Alloc_obs` — Declare Observables

Allocate the observable arrays and configure each measurement:

```fortran
! Scalar observables
Allocate(Obs_scal(4))
Call Obser_Vec_make(Obs_scal(1), 1, "Kin")
Call Obser_Vec_make(Obs_scal(2), 1, "Pot")
Call Obser_Vec_make(Obs_scal(3), 1, "Part")
Call Obser_Vec_make(Obs_scal(4), 1, "Ener")

! Equal-time correlators
Allocate(Obs_eq(2))
Call Obser_Latt_make(Obs_eq(1), 1, "Green", Latt, Latt_unit, '--', dtau)
Call Obser_Latt_make(Obs_eq(2), 1, "SpinZ", Latt, Latt_unit, '--', dtau)

! Time-displaced correlators (only when Ltau=1)
If (Ltau == 1) then
  Allocate(Obs_tau(2))
  Call Obser_Latt_make(Obs_tau(1), Ltrot+1-2*Thtrot, "Green", Latt, Latt_unit, 'P', dtau)
  Call Obser_Latt_make(Obs_tau(2), Ltrot+1-2*Thtrot, "SpinZ", Latt, Latt_unit, 'PH', dtau)
endif
```

The `Channel` string (`'P'`, `'PH'`, `'PP'`, `'T0'`, …) determines the analytic continuation kernel. See [[Analytic Continuation]].

#### `Obser` — Equal-Time Measurements

Called once per time slice during measurements. Receives the Green function `GR(Ndim, Ndim, N_FL)`, the sign/phase `Phase`, and the time slice index `Ntau`. Use the predefined measurement routines (see [[Predefined Observables]]) or write custom contractions.

#### `ObserT` — Time-Displaced Measurements

Called with two Green functions `GR_T0(i,j)` = $\langle c_i(\tau) c_j^\dagger(0) \rangle$ and `GR_eq` (equal-time). Same structure as `Obser` but for dynamical correlations.

### 5. Optional Overrides

Additional procedures from `ham_base` can be overridden when needed:

| Procedure | Purpose |
|-----------|---------|
| `S0` | Bosonic action for continuous Hubbard-Stratonovich fields |
| `Global_move` | Space-time global moves (e.g., Ising-field flips) |
| `Global_move_tau` | Single-time-slice global moves |
| `Apply_B_HMC` | Custom mass matrix for HMC (see [[HMC Parameters]]) |
| `Ham_Langevin_HMC_S0` | Bosonic force for Langevin/HMC updates |

To enable an override, uncomment the corresponding `procedure` line in the type declaration and implement it.

### 6. Build and Test

```bash
source configure.sh GNU nompi devel
make cleanprog && make -j5 program
```

The `devel` flag enables runtime checks (bounds checking, NaN traps) that are invaluable during development.

## Tips

- **Start from a working model.** `Hamiltonian_Hubbard_Plain_Vanilla_smod.F90` is the simplest complete example — study it before writing your own.
- **Use predefined lattices and observables.** They save effort and produce output in the standard format that the analysis tools expect.
- **Check the `info` file** after a short test run for acceptance rates, stabilization precision, and model metadata.
- **Continuous fields** (`Continuous = .true.` in `&VAR_Hubbard`) require implementing `S0` and the Langevin/HMC force. See the full Hubbard model for an example.

## Available Building Blocks

- [[Predefined Lattices]] — Bravais lattices you can use out of the box
- [[Predefined Observables]] — Measurements you can enable without writing new code

## Validating a New Model

When implementing a new Hamiltonian, it is essential to verify correctness before running production simulations. Bugs in the operator setup or the HS decomposition can produce plausible-looking but wrong results. The following checks catch most implementation errors.

### Non-Interacting Limit

Set the interaction to zero (e.g., `Ham_U = 0`). The HS fields decouple completely and ALF should reproduce the exact free-fermion result — obtainable by diagonalizing the hopping matrix. This tests your lattice geometry, boundary conditions, hopping operators, and `Dtau` discretization. Any disagreement beyond the Trotter error indicates a bug in `Op_T` or the lattice setup.

### Comparison with Exact Diagonalization

For small systems (e.g., $2 \times 2$ or $4 \times 1$), compare ALF results with exact diagonalization (ED). Use a large `NSweep` and many bins to get precise QMC data, and a small `Dtau` to minimize Trotter error. Energy, density, and local correlators should agree within error bars after $\Delta\tau \to 0$ extrapolation. This is the single most powerful validation of a new model.

> **Tip:** Build with `devel` mode (`source configure.sh GNU nompi devel`) for runtime checks (bounds checking, NaN traps) during this phase.

### Symmetry Checks

Verify that known symmetries are respected:
- **Particle-hole symmetry** — For the repulsive Hubbard model on a bipartite lattice at half-filling (`Ham_chem = 0`): the average sign must be exactly 1 and the density must be exactly 0.5 per site per color. ALF automatically exploits this symmetry when detected (computes only one flavor, reconstructs the other).
- **Time-reversal symmetry** — For attractive interactions (`Ham_U < 0`), similar simplifications apply automatically.
- **Lattice symmetries** — Correlation functions should respect the point group of the lattice. An asymmetry in observables that should be symmetric signals an error in the operator definitions.

### Dtau → 0 Extrapolation

Run at 2–3 values of `Dtau` and verify that observables converge as $O(\Delta\tau^2)$ (symmetric Trotter decomposition). Deviations from this scaling indicate either an implementation error or insufficient numerical stabilization. See [[Discretization]].

### Known Limits and Sum Rules

If your model has analytically known limits (strong coupling, weak coupling, high temperature) or sum rules (e.g., frequency sum rules for spectral functions, particle number constraints), check them. These provide independent verification that does not rely on comparison codes.
