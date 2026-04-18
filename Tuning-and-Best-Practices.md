# Tuning and Best Practices

Practical advice on choosing simulation parameters for reliable and efficient ALF runs.

## Philosophy

The [formal documentation (PDF)](https://alf.physik.uni-wuerzburg.de/doc.pdf) derives the algorithms. This section focuses on **what to set, what to look for, and what can go wrong** — practical knowledge that comes from experience running simulations.

Each sub-page follows a consistent format:
1. **Parameters** — name, where it's set, typical range
2. **Guidelines** — how to choose good values, what to monitor
3. **Model-specific notes** — when defaults don't apply
4. **Known pitfalls** — common mistakes and their symptoms

## Topics

### [[HMC Parameters]]
Tuning the Hybrid Monte Carlo updating scheme: leap-frog step size (`Delta_t_Langevin_HMC`), number of integration steps (`Leapfrog_Steps`), the mass matrix preconditioner (`Apply_B_HMC`), and how many HMC trajectories to run between sequential sweeps (`N_HMC_sweeps`).

### [[Stabilization Parameters]]
Choosing `Nwrap` (the number of imaginary-time slices between QR stabilizations) and selecting a stabilization scheme (`STAB1`/`STAB2`/`STAB3`/`LOG`). Getting this wrong leads to numerical instability or wasted computation.

### [[Discretization]]
The imaginary-time step `Dtau` controls the Trotter decomposition error. Too large and results are biased; too small and the simulation is unnecessarily expensive. Guidance on choosing `Dtau` and extrapolating to the continuous-time limit.

### [[Tempering]]
Parallel tempering configuration: how to choose the temperature grid, how many replicas to use, and what exchange acceptance rates to target.

## General Advice

- **Always check the `info` file** after a run. It reports acceptance rates, precision of the Green's function, and walltime. Anomalous values are the first sign of trouble.
- **Start small.** Test parameter choices on small lattices (e.g. 4×4) before committing to expensive production runs. The physics should be qualitatively the same.
- **Compare with exact results when possible.** Small systems (e.g. 2-site or 4-site Hubbard) have exact diagonalization results. Use them to validate your setup.
- **Monitor autocorrelation.** If consecutive bins are correlated, increase `NSweep` (sweeps per bin) or use rebinning in the analysis step.
- **Combine update schemes.** When using HMC or Langevin dynamics, always keep sequential updates enabled (`Sequential = .true.`) to avoid ergodicity issues.
