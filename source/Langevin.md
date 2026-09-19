# Langevin updates

ALF's Langevin mode updates **continuous real auxiliary fields**.  It is a
global update: during one Langevin step every field is changed and there is no
Metropolis acceptance/rejection step.

## When it can be used

- Enable the mode with `Langevin = .T.` in `&VAR_QMC`.
- All auxiliary fields must be of ALF type 3 (continuous); ALF stops during setup if any field has another type.
- The Hamiltonian must provide the derivative of its bosonic action, `Ham_Langevin_HMC_S0`. The base implementation assumes $S_0=0$ and prints a warning because this is usually incorrect. Existing model implementations may already supply this routine; a new Hamiltonian needs a correct implementation of $\partial S_0/\partial s_{n,\tau}$.
- Langevin mode turns off sequential, HMC, global, MALA, and tempering updates. Do not combine them in the same run.

`Langevin = .T.` is the current input switch.  Do **not** add
`Global_update_scheme = Langevin`: it is not a `VAR_QMC` input variable in the
current ALF source.

## Update equation

For a field vector $\mathbf{s}$, the Langevin equation documented by ALF is

$$
\mathbf{s}(t + \delta t)
= \mathbf{s}(t)
- Q\,\nabla S\bigl(\mathbf{s}(t)\bigr)\,\delta t
+ \sqrt{2\delta t\,Q}\,\boldsymbol{\eta}(t),
$$

where $\boldsymbol{\eta}$ consists of independent unit Gaussian random
variables.  The current implementation uses the identity form $Q=I$;
Fourier acceleration is not implemented.

In the code, the drift force contains the bosonic contribution $F_0$ and the
phase-reweighted fermionic contribution $F_\mathrm{F}$:

$$
F_\mathrm{drift}
= F_0
+ \frac{\operatorname{Re}\!\left(\texttt{Phase}\,F_\mathrm{F}\right)}
                      {\operatorname{Re}\!\left(\texttt{Phase}\right)}.
$$

Each continuous field is then updated as

$$
s \leftarrow s
- F_\mathrm{drift}\,\delta t_\mathrm{run}
+ \sqrt{2\delta t_\mathrm{run}}\,\eta .
$$

## Minimal input

Place the following in the `&VAR_QMC` namelist of the `parameters` file.
The numerical values are examples, not universal defaults.

```fortran
&VAR_QMC
        Langevin             = .T.
        Delta_t_Langevin_HMC = 0.01d0
        Max_Force            = 5.0d0
        NSweep               = 100
        NBin                 = 100
/
```

`Delta_t_Langevin_HMC` is the nominal, dimensionless Langevin-time step.
`Max_Force` is the threshold for adaptive stepping.  Explicitly set both to
positive values: their current runtime defaults are zero and the code does
not perform a positivity check.

ALF performs one Langevin update for each `NSweep` iteration, so a run makes
`NBin * NSweep` updates.  A bin contains the measurements accumulated during
its `NSweep` updates.

## What `Max_Force` does

`Max_Force` does **not** clip the force.  For the largest force estimate
$X_\mathrm{max}$ in a configuration, ALF uses

$$
\delta t_\mathrm{run} =
\begin{cases}
\delta t, & X_\mathrm{max} \leq \texttt{Max\_Force}, \\
\dfrac{\texttt{Max\_Force}}{X_\mathrm{max}}\,\delta t,
& X_\mathrm{max} > \texttt{Max\_Force}.
\end{cases}
$$

Thus, a large force makes the current step shorter while leaving the force
unchanged.  ALF weights measurements with this running time step when
adaptive stepping is active.  The `info` file reports mean and maximum forces
at the end of the run; use it to identify frequent or very large excursions.
An adaptive step can prevent a crash, but it does not make a simulation with
singular or unbounded forces reliable.

## Choosing parameters and checking results

A nonzero `Delta_t_Langevin_HMC` introduces a systematic integration error.
Run several simulations with decreasing nominal time steps, keep all other
physical parameters fixed, and verify convergence (or extrapolate) as

$$
\delta t \longrightarrow 0.
$$

Choose `Max_Force` to protect against rare force spikes, then inspect the
force statistics and the amount of adaptive stepping.  Do not use a small
`Max_Force` as a substitute for a time-step study: it only slows the
Langevin-time advance near a large force.

Measurements begin immediately.  Discard equilibration bins during analysis
with `n_skip`; it is not a Langevin warm-up parameter.  To restart an orderly
completed run, use `out_to_in.sh` to turn the written `confout_*` fields into
input configurations before running ALF again.  ALF also writes the latest
running time step to `Langevin_time_steps` for continuation.

  
A worked pyALF example is available in  [pyALF_MALA_run.ipynb](pyALF_MALA_run.ipynb).