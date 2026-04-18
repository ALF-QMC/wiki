# Predefined Observables

Reference for the observable types available in ALF. These are implemented in `Prog/Predefined_Obs_mod.F90` and can be used in any Hamiltonian's `Alloc_obs`, `Obser`, and `ObserT` subroutines.

## Observable Data Types

ALF provides three observable containers, defined in `Prog/observables_mod.F90`:

| Type | Created with | Purpose | Output suffix |
|------|-------------|---------|---------------|
| `Obser_Vec` | `Obser_Vec_make(obs, N, filename)` | Scalar quantities (one number per bin) | `_scal` |
| `Obser_Latt` | `Obser_Latt_make(obs, Nt, filename, Latt, Latt_unit, Channel, dtau)` | Lattice correlation functions | `_eq` or `_tau` |

## Equal-Time Observables

Called from your `Obser` subroutine using the predefined measurement routines:

| Routine | Output file | What it measures | Requirements |
|---------|-------------|------------------|-------------|
| `Predefined_Obs_eq_Green_measure` | `Green_eq` | Equal-time Green's function $\sum_\sigma \langle c^\dagger_{i\sigma} c_{j\sigma} \rangle$ | Any model |
| `Predefined_Obs_eq_Den_measure` | `Den_eq` | Density-density correlations $\langle N_i N_j \rangle - \langle N_i \rangle \langle N_j \rangle$ | Any model |
| `Predefined_Obs_eq_SpinSUN_measure` | `SpinZ_eq` | SU(N) spin-spin correlations | `N_FL = 1` |
| `Predefined_Obs_eq_SpinMz_measure` | `SpinZ_eq`, `SpinXY_eq`, `SpinT_eq` | $S^z$-$S^z$, $S^x + S^y$, total spin correlations | `N_FL = 2`, `N_SUN = 1` |

## Time-Displaced Observables

Called from your `ObserT` subroutine:

| Routine | Output file | What it measures | Channel |
|---------|-------------|------------------|---------|
| `Predefined_Obs_tau_Green_measure` | `Green_tau` | $\langle c_i(\tau) c^\dagger_j(0) \rangle$ | `P` |
| `Predefined_Obs_tau_Den_measure` | `Den_tau` | Time-displaced density correlations | `PH` |
| `Predefined_Obs_tau_SpinSUN_measure` | `SpinZ_tau` | Time-displaced SU(N) spin correlations | `PH` |
| `Predefined_Obs_tau_SpinMz_measure` | `SpinZ_tau`, `SpinXY_tau`, `SpinT_tau` | Time-displaced spin correlations | `PH` |

The `Channel` string determines the kernel used for [[Analytic Continuation]]: `'P'` (particle), `'PH'` (particle-hole), `'PP'` (particle-particle), `'T0'` (zero temperature).

## Scalar Observables

Standard scalar observables are model-defined (not in `Predefined_Obs`) but typically include:

| Name | Output file | What it measures |
|------|-------------|------------------|
| `Kin` | `Kin_scal` | Kinetic energy |
| `Pot` | `Pot_scal` | Potential energy |
| `Part` | `Part_scal` | Particle number |
| `Ener` | `Ener_scal` | Total energy |

### Entanglement Observables

ALF also provides predefined Rényi entropy and mutual information measurements:

| Routine | What it measures |
|---------|------------------|
| `Predefined_Obs_scal_Renyi_Ent_indep` / `_gen_fl` / `_gen_all` | Rényi entanglement entropy (flavor-independent / general-flavor / all) |
| `Predefined_Obs_scal_Mutual_Inf_indep` / `_gen_fl` / `_gen_all` | Mutual information |

## Controlling Measurements

- **`Ltau`** (in `&VAR_QMC`): Set to `1` to compute time-displaced correlations, `0` to skip them. Skipping saves time when only equal-time data is needed.
- **`LOBS_ST`, `LOBS_EN`** (in `&VAR_QMC`): Measurement time-slice window. Default `1, 1` measures only once per sweep. Setting `LOBS_EN = Ltrot` measures on every time slice (better statistics but slower).

## Adding a Custom Observable

To define a model-specific observable in your Hamiltonian:

1. In `Alloc_obs`: allocate and configure the observable with `Obser_Vec_make` or `Obser_Latt_make`
2. In `Obser` (equal-time) or `ObserT` (time-displaced): compute the Wick contractions from the Green function and store the result
3. The analysis tools handle the rest — Jackknife error analysis, Fourier transforms, and file output are automatic
