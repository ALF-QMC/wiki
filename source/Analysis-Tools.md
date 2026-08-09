# Analysis Tools

Post-processing simulation output: error analysis, Fourier transforms, and analytic continuation.

## Overview

After a simulation completes, raw bin data must be analyzed to extract physical observables with error bars. The workflow depends on whether HDF5 output was enabled.

## Analysis Workflow

### With HDF5 (recommended)

```bash
$ALF_DIR/Analysis/ana_hdf5.out              # analyze all observables
$ALF_DIR/Analysis/ana_hdf5.out Ener_scal    # analyze a specific observable
```

Results are written to a `res/` subdirectory. The input is a single `data.h5` file containing all observables.

### Without HDF5

```bash
$ALF_DIR/Analysis/ana.out *          # analyze all observables
$ALF_DIR/Analysis/ana.out Ener_scal  # analyze a specific observable
```

`ana.out` dispatches to the appropriate analysis routine based on the observable name suffix:

| Suffix | Analysis | Output |
|--------|----------|--------|
| `_scal` | Scalar covariance (`cov_scal`) | `<name>J` (Jackknife mean ± error) |
| `_eq` | Equal-time correlations (`cov_eq`) | `<name>JR` (real space), `<name>JK` (k-space) |
| `_tau` | Time-displaced correlations (`cov_tau`) | `<name>_kx_ky/g_dat` (k-resolved, τ-dependent) |

### Legacy Script Workflow

For fine-grained control, the individual analysis programs can be called directly. The script `Scripts_and_Parameters_files/analysis.sh` demonstrates this:

```bash
# Scalar observables
for i in *_scal; do $ALF_DIR/Analysis/cov_scal.out < $i; done

# Equal-time observables
for i in *_eq; do ln -sf $i ineq && $ALF_DIR/Analysis/cov_eq.out; done

# Time-displaced observables (particle channel)
for i in Green_tau; do ln -sf $i intau && $ALF_DIR/Analysis/cov_tau.out; done

# Time-displaced observables (particle-hole channel)
for i in SpinZ_tau Den_tau; do ln -sf $i intau && $ALF_DIR/Analysis/cov_tau_ph.out; done
```

## Available Analysis Programs

| Program | Input | Purpose |
|---------|-------|---------|
| `ana.out` | Plain-text bin files | All-in-one dispatcher |
| `ana_hdf5.out` | `data.h5` | All-in-one dispatcher (HDF5) |
| `Max_SAC.out` | Analyzed τ-dependent data | MaxEnt / Stochastic Analytic Continuation |
| `convert_bins.py` | Plain-text bins | Convert plain-text output to HDF5 format |

### Legacy Programs

The following programs are called internally by `ana.out` / `ana_hdf5.out`. They are rarely needed directly.

| Program | Input | Purpose |
|---------|-------|---------|
| `cov_scal.out` | `*_scal` files | Scalar observable Jackknife analysis |
| `cov_eq.out` | `*_eq` files | Equal-time correlation analysis + Fourier transform |
| `cov_tau.out` | `*_tau` files | Time-displaced correlation analysis (particle channel) |
| `cov_tau_ph.out` | `*_tau` files | Time-displaced correlation analysis (particle-hole channel) |
| `cov_mut.out` | Mutual information data | Mutual information covariance analysis |

## Sub-Pages

- [[HDF5 Output Format]] — Structure of `data.h5`
- [[Bin Conversion]] — Converting plain-text output to HDF5
- [[Analytic Continuation]] — Maximum entropy and stochastic analytic continuation
