# Production Run Cycle

This page explains how to obtain precise simulation data with ALF. The focus is on choosing the simulation control parameters — the knobs that determine whether your data is trustworthy and your compute time well spent. The physics parameters (which model, which coupling, which lattice size) depend on the specific study and are covered elsewhere; here we concentrate on the machinery.

## The Iterative Cycle

Production data is rarely obtained in a single shot. The typical workflow is:

1. **Set up and run** — Choose initial values for the control parameters, launch the simulation
2. **Analyze** — Run the analysis tools, inspect error bars and the QMC time series
3. **Evaluate** — Is the data precise enough? Are there signs of thermalization issues or autocorrelation?
4. **Extend or adjust** — Add more bins, tune parameters, rerun

This cycle repeats until the data quality is sufficient for the question at hand. The rest of this page explains how to choose the parameters that govern this cycle.

## Simulation Control Parameters

### `Dtau` — Imaginary-Time Step

`Dtau` controls the Trotter discretization. The number of time slices is $L_\text{trot} = \text{nint}(\beta / \Delta\tau)$.

- **`Dtau = 0.1` is a good default** for most models where the hopping sets the energy scale ($t = 1$).
- Models with larger energy scales (e.g., large `Ham_T`, strong spin-orbit coupling) may need smaller `Dtau`.
- If `Nwrap = 1` still gives poor Green function precision (see below), reducing `Dtau` may help because it reduces the norm of the individual propagators.
- Make sure `Beta / Dtau` is an integer to avoid rounding surprises.
- For publication-quality results, run at 2–3 values of `Dtau` and extrapolate to $\Delta\tau \to 0$. The error is $O(\Delta\tau^2)$ with symmetric Trotter decomposition. See [[Discretization]].

### `Nwrap` — Stabilization Interval

The Green function is propagated ("wrapped") from one time slice to the next via rank-1 updates. Every `Nwrap` slices, it is recomputed from scratch using a numerically stable UDV decomposition. See [[Stabilization Parameters]].

**Choose `Nwrap` as large as possible** while maintaining precise Green functions:
- After a run, check the `info` file for:
  ```
  Precision Green  Mean:  1.234E-08   Max:  5.678E-06
  ```
- **Target: mean deviation < $10^{-7}$–$10^{-8}$.** This ensures the wrapping error is negligible compared to statistical error bars.
- Deviations around $10^{-4}$ *may* be acceptable in extreme cases (large lattices, low temperature) if you understand that the systematic bias could be comparable to or larger than your error bars.
- If the precision is poor, reduce `Nwrap`. If the run aborts with "This calculation is unstable!", reduce `Nwrap` or `Dtau`.
- Larger `Nwrap` is more efficient (fewer expensive UDV recomputations per sweep), so there is a real payoff in finding the largest stable value.

### `NSweep` — Sweeps per Bin

Each bin consists of `NSweep` full sweeps through all time slices. Observables are averaged within a bin and written to disk at the end.

The right value of `NSweep` depends on how long a single sweep takes:

- **Fast runs** (small lattices, high temperature, debugging against exact diagonalization): Use a **large** `NSweep` (e.g., 50–200). This gives precise per-bin averages without producing large data files.
- **Production runs on large systems**: Aim for a bin duration of roughly **one minute to one hour**. This provides a good balance between per-bin statistics and the ability to monitor the time series.
- **Very expensive systems** (e.g., a single sweep takes 20 hours and the queuing system limits wall-clock to 24 hours): You may have to reduce `NSweep` to 1 and rely on accumulating many single-sweep bins across multiple job submissions.

There is no universal "best" value — it depends on the cost per sweep and the job scheduling constraints.

### `NBin` — Number of Bins

`NBin` sets how many bins the simulation produces in one invocation. The total number of effective bins used in the analysis is `(total_bins - n_skip) / N_rebin`, where `total_bins` may come from multiple job submissions.

In practice:
- Start with a moderate value, e.g., `NBin = 100`.
- Run the simulation, analyze, and inspect the results with error bars.
- If the data is not precise enough, **rerun the same parameter set** — ALF appends new bins to the existing data files when restarted from a saved configuration (see [Restarts](#restarts-and-extending-runs) below).
- Repeat until the desired precision is reached. How many bins you ultimately need depends strongly on the observable and the physics (sign problem, proximity to a phase transition, etc.), but it is often obvious from looking at the data: if the error bars are too large to answer your question, you need more bins.

### `CPU_MAX` — Wall-Clock Safety

`CPU_MAX` (given in **hours** in the `parameters` file, e.g., `CPU_MAX = 24.0` for a one-day limit) ensures graceful termination. After each bin, ALF estimates whether there is enough time for another bin. If not, it stops cleanly, writes the field configuration, and exits.

This is primarily a safety measure to **prevent data corruption**: if the queuing system kills the job during I/O, data files can be left in an inconsistent state. Always set `CPU_MAX` to your job's wall-clock limit or slightly below.

> **Note:** `CPU_MAX` overrides `NBin` — if wall-clock time runs out, the simulation stops even if fewer than `NBin` bins have been completed.

```fortran
&VAR_QMC
NSweep  = 40
NBin    = 100
Nwrap   = 10
CPU_MAX = 24.0   ! hours
/
```

### `Ltau` — Time-Displaced Measurements

Set `Ltau = 1` to enable time-displaced Green function measurements ($G(\tau)$, spin correlations in $\tau$, etc.). This adds computational cost, so **only enable it when you need dynamical correlations** — e.g., for spectral functions via [[Analytic Continuation]]. For studies that only require equal-time observables, leave it at the default `Ltau = 0`.

### `Global_moves` — Ergodicity

Some models benefit from global (space-time) updates in addition to the local sequential or HMC updates. Set `Global_moves = .true.` and `N_Global` to the number of global moves per sweep. Whether this is needed depends on the model — for the standard Hubbard model it is not required, but models with discrete symmetries or flat directions in the action may need it. Check acceptance rates in `info`.

### `Projector` and `Theta` — Ground-State Algorithm

For ground-state (PQMC) calculations, set `Projector = .true.` and choose `Theta` large enough that the projection $e^{-\Theta H}$ filters out the ground state. Typical values are `Theta = 10`–`40` (in units of $1/t$). Larger `Theta` is safer but more expensive ($L_\text{trot}$ grows). Convergence in `Theta` should be checked explicitly.

### HMC Parameters

For continuous auxiliary fields, the Hybrid Monte Carlo (HMC) update scheme is available via `HMC = .true.`. The key control parameters are `Delta_t_Langevin_HMC`, `Leapfrog_steps`, and `N_HMC_sweeps`. See [[HMC Parameters]] for details.

> HMC auto-tuning is under active development. A procedure to automatically determine efficient values for these hyper-parameters is being worked on.

## Analysis: Thermalization and Autocorrelation

ALF does **not** have a separate warmup phase. All bins are identical from the simulation's perspective. Thermalization is handled entirely at analysis time via the `VAR_errors` namelist:

```fortran
&VAR_errors
n_skip  = 5     ! discard first 5 bins (warmup)
N_rebin = 1     ! rebinning factor
/
```

### `n_skip` — Discarding Warmup Bins

The first `n_skip` bins are excluded from the jackknife analysis. For a cold start (no `confin`), the system needs time to reach equilibrium.

**How to choose `n_skip`:** Look at the QMC time series of a scalar observable such as the energy (`Ener_scal`). The initial transient — where the observable drifts from its starting value toward the equilibrium mean — tells you how many bins to discard. pyALF provides interactive widgets for this; it is **highly recommended** to visually inspect the time series rather than guessing.

### `N_rebin` — Rebinning for Autocorrelation

`N_rebin` groups consecutive bins: `N_rebin = 2` averages pairs of bins into one before the jackknife. This reduces the effect of autocorrelation between bins.

**How to choose `N_rebin`:** Increase `N_rebin` = 1, 2, 4, ... and watch the error bar. If it stabilizes (plateaus), the bins are uncorrelated at that rebinning level. If the error keeps growing, the bins are still correlated and you need either a larger `N_rebin` or more `NSweep` per bin to decorrelate.

pyALF provides widgets to visualize both the time series and the error-bar-vs-rebinning curve, which makes this straightforward.

The number of effective bins after analysis is `(total_bins - n_skip) / N_rebin`. You need at least 2 effective bins for the jackknife to produce an error estimate.

## Restarts and Extending Runs

ALF writes the field configuration to `confout` at the end of every bin. To continue a run:

1. Rename the configuration: `bash out_to_in.sh` (renames `confout_*` → `confin_*`)
2. Resubmit the job — ALF detects `confin`, starts from the saved configuration, and **appends** new bins to the existing data files
3. The first bins of the extension are already thermalized (warm start), so `n_skip` only needs to account for the initial cold-start transient, not the restart

### What You Can and Cannot Change Between Runs

When extending a run by appending bins, the new bins are concatenated with the old ones in the data files and analyzed together. This means some parameters **must stay fixed**, while others can be changed freely:

| Parameter | Can change? | Why |
|---|---|---|
| `NBin` | **Yes** | Each run simply adds more bins. |
| `Nwrap` | **Yes** (with care) | Safe as long as you are not at the stability boundary — changing `Nwrap` should not change the observable values, only the numerical precision of the Green function. |
| `CPU_MAX` | **Yes** | Only affects when the run stops. |
| `n_skip`, `N_rebin` | **Yes** | These are analysis-time parameters, not simulation parameters. |
| **`NSweep`** | **No** | Bins with different `NSweep` represent averages over different numbers of sweeps. Mixing them in the analysis produces incorrect error estimates. If you need to change `NSweep`, delete the existing data and start fresh. |
| **Number of MPI ranks** | **No** | The number of MPI workers must remain the same across runs for the same parameter set. Changing it would produce incompatible `confout`/`confin` files and data layouts. |

> **Note:** It is perfectly fine to use different `NSweep` values for *different parameter sets* (e.g., larger `NSweep` for bigger lattices). The constraint is that all runs contributing bins to the *same data file* must use the same `NSweep` and MPI configuration.

> **Note:** The `RUNNING` lock file prevents accidental concurrent runs in the same directory. If a previous run crashed, you must delete `RUNNING` manually before restarting.

On HPC clusters, a common pattern is to include `bash out_to_in.sh` in the job script before the `srun` command, so resubmission automatically continues from the previous state. See [[Running on Clusters]] for examples.

## Checking Your Results

After analysis, inspect these key diagnostics:

| What to check | Where | What it tells you |
|---|---|---|
| Green function precision | `info`: `Precision Green Mean, Max` | Wrapping/stabilization quality. Mean should be $< 10^{-7}$. |
| Average sign | `Sign_scal` or HDF5 `Sign` | Sign $\ll 1$ means exponentially noisy data. |
| Acceptance rates | `info`: `Acceptance`, `Acceptance_HMC` | Too low → poor sampling. Too high → steps too small. |
| QMC time series | `Ener_scal` bin-by-bin (use pyALF widgets) | Thermalization transient, stationarity, obvious outliers. |
| Error bar vs. rebinning | Increase `N_rebin` and re-analyze | Plateau → bins are uncorrelated. Growing → more `NSweep` needed. |

## Quick Reference

| Parameter | Namelist | Typical starting value | Key consideration |
|---|---|---|---|
| `Dtau` | `VAR_Model_Generic` | `0.1` | Reduce for large energy scales or poor stability |
| `Nwrap` | `VAR_QMC` | `10` | As large as possible with Green precision $< 10^{-7}$ |
| `NSweep` | `VAR_QMC` | `20`–`200` | Target 1 min – 1 hour per bin for production |
| `NBin` | `VAR_QMC` | `100` | Extend by rerunning; precision is the criterion |
| `CPU_MAX` | `VAR_QMC` | Job wall-clock (hours) | Prevents data corruption; overrides `NBin` |
| `Ltau` | `VAR_QMC` | `0` | Set to `1` only when time-displaced data is needed |
| `n_skip` | `VAR_errors` | `5`–`20` (cold start) | Inspect time series to determine |
| `N_rebin` | `VAR_errors` | `1` | Increase until error bar plateaus |

## See Also

- [[Quick Start]] — First run tutorial
- [[Tuning and Best Practices]] — Detailed parameter guidance
- [[Stabilization Parameters]] — `Nwrap` and numerical stability in depth
- [[HMC Parameters]] — HMC-specific tuning
- [[Analysis Tools]] — Post-processing reference
- [[Running on Clusters]] — HPC job submission and restart patterns
