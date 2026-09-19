
# MALA updates

The Metropolis-adjusted Langevin algorithm (MALA) updates continuous
auxiliary fields using the action derivative to guide a Gaussian proposal,
then applies a Metropolis-Hastings accept/reject step. The correction makes
the target distribution exact for a finite proposal time step; unlike an
unadjusted Langevin update, the time step is a tuning parameter rather than a
source of integration bias.

ALF provides three MALA scopes:

| Scope | Use it for | Main switch |
| --- | --- | --- |
| Sequential | Field-by-field updates during a sequential sweep | `Sequential_MALA = .T.` |
| Global tau | A selected group of fields on one imaginary-time slice | `Global_tau_MALA_moves = .T.` |
| Global | A selected group of fields across the full space-time configuration | `Global_MALA_moves = .T.` |

All MALA updates are for real, continuous ALF fields (type 3). In particular,
the selection routines used by global MALA moves must select only fields that
can be updated continuously.

## What ALF proposes

For a selected field, ALF constructs the drift

$$
D = F_0 +
\frac{\operatorname{Re}\!\left(\texttt{Phase}\,F_\mathrm{F}\right)}
     {\operatorname{Re}\!\left(\texttt{Phase}\right)},
$$

where $F_0 = \partial S_0/\partial s$ is the bosonic-action derivative and
$F_\mathrm{F}$ is the fermionic force. It proposes

$$
s' = s - D\,\delta t_\mathrm{run}
     + \sqrt{2\delta t_\mathrm{run}}\,\eta,
\qquad \eta \sim \mathcal{N}(0,1).
$$

Because this proposal is asymmetric, ALF includes the forward and reverse
Gaussian proposal densities in the Metropolis-Hastings test:

$$
P_\mathrm{acc} = \min\!\left(
1,
\frac{T(C' \rightarrow C)\,W(C')}
     {T(C \rightarrow C')\,W(C)}
\right).
$$

The acceptance step is important: do not use MALA terminology interchangeably
with ALF's unadjusted `Langevin = .T.` mode.

## Adaptive time step and `Max_Force`

Each MALA variant has its own nominal time step and force threshold. ALF does
**not** clip the force. Instead, for the largest selected force magnitude
$X_\mathrm{max}$, it uses

$$
\delta t_\mathrm{run} =
\delta t\,
\min\!\left(1,
\frac{\texttt{Max\_Force\_MALA}}{X_\mathrm{max}}
\right).
$$

The running time step is evaluated at both ends of the proposal and included
in the proposal-density ratio. Consequently, adaptive stepping remains part
of the MALA proposal rather than a force cutoff. A small threshold only makes
proposals shorter; it cannot make singular or unbounded forces reliable.

## Quick start: sequential MALA

Sequential MALA is the most direct option for an existing continuous-field
simulation. Put the following in the `&VAR_QMC` namelist of `parameters`:

```fortran
&VAR_QMC
   Sequential                 = .T.
   Sequential_MALA            = .T.
   Delta_t_MALA_sequential    = 0.01d0
   Max_Force_MALA_sequential  = 1.0d0
   NSweep                     = 100
   NBin                       = 100
/
```

ALF applies the MALA proposal to type-3 fields in the sequential operator
range. `Nt_sequential_start` and `Nt_sequential_end` can restrict that range
when required by the Hamiltonian.

Always explicitly set both MALA parameters. In this checkout,
`Delta_t_MALA_sequential` defaults to zero, and ALF terminates if an enabled
MALA mode has a non-positive step size or force threshold.

The Hamiltonian must provide the bosonic force. For sequential MALA, provide
the efficient single-field hook
`Ham_Langevin_HMC_S0_single(Force_0, n, nt)`. If it is absent, ALF falls
back to `Ham_Langevin_HMC_S0(Forces_0)`, extracts one element, and emits a
warning because evaluating the full force array for every field is expensive.

## Global-tau MALA

Global-tau MALA makes a joint proposal for fields on one time slice. It runs
within the sequential update path, so `Sequential = .T.` is required.

```fortran
&VAR_QMC
   Sequential                      = .T.
   Global_tau_MALA_moves           = .T.
   N_Global_tau_MALA               = 1
   Delta_t_MALA_global_tau         = 0.01d0
   Max_Force_MALA_global_tau       = 1.0d0
/
```

`N_Global_tau_MALA` is the number of MALA proposals made on each visited time
slice. A worked configuration example is available in
[pyALF_MALA_run.ipynb](pyALF_MALA_run.ipynb). Implement the following
Hamiltonian hook to choose the fields in a proposal:

```fortran
subroutine Global_MALA_move_tau(Flip_list, Flip_length, ntau)
   integer, intent(out) :: Flip_list(:)
   integer, intent(out) :: Flip_length
   integer, intent(in) :: ntau

   ! Set Flip_list(1:Flip_length) to type-3 operator indices at ntau.
end subroutine Global_MALA_move_tau
```

The base implementation selects every operator index and prints a warning.
Override it when only a subset is appropriate. The Hamiltonian can also
override `N_Global_tau_MALA` through
`Overide_global_tau_sampling_parameters`.

A worked pyALF example is available in
[pyALF_MALA_run.ipynb](pyALF_MALA_run.ipynb).

## Global MALA

Global MALA proposes a selected set of fields across all space and imaginary
time, followed by one accept/reject decision for the full proposal. It does
not require `Sequential = .T.`.

```fortran
&VAR_QMC
   Global_MALA_moves          = .T.
   N_Global_MALA_sweeps       = 1
   Delta_t_MALA_global        = 0.01d0
   Max_Force_MALA_global      = 1.0d0
/
```

`N_Global_MALA_sweeps` is the number of global MALA proposals performed per
simulation sweep. Select the fields in the Hamiltonian with:

```fortran
subroutine Global_MALA_move(Flip_list)
   integer, intent(out) :: Flip_list(:,:)

   ! Use 1 for selected type-3 fields and 0 for fields left unchanged.
end subroutine Global_MALA_move
```

The base implementation marks every field. A custom selector is useful when
a full space-time proposal is too large or mixes different field types. The
global version needs the full bosonic-force routine
`Ham_Langevin_HMC_S0(Forces_0)`.

A worked pyALF example is available in
[pyALF_MALA_run.ipynb](pyALF_MALA_run.ipynb).

## Tuning and checks

1. Start with a small `Delta_t_MALA_*` and a positive `Max_Force_MALA_*`.
   Increase the time step only after checking acceptance and autocorrelation.
2. Inspect `info` after each test run. It reports mean and maximum fermionic
   and bosonic forces for sequential, global-tau, and global MALA. Global
   MALA also reports `Global Acceptance_MALA`; sequential and global-tau
   acceptances contribute to the ordinary update acceptance statistics.
3. Low acceptance or frequent force spikes usually calls for a smaller nominal
   time step. Raising `Max_Force_MALA_*` permits larger instantaneous moves;
   lowering it produces more adaptive shortening.
4. Measurements begin immediately. Discard equilibration bins during
   analysis with `n_skip`, and choose `NSweep` and `NBin` long enough to study
   autocorrelation and statistical errors.

MALA is incompatible with the Green's-function reconstruction path in this
version of ALF. It should also not be combined with `Langevin = .T.`: enabling
Langevin disables the sequential update path and the global MALA modes.

