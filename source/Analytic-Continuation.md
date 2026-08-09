# Analytic Continuation

Maximum entropy and stochastic analytic continuation methods for extracting real-frequency spectral functions from imaginary-time data.

## Overview

QMC simulations produce correlation functions in imaginary time $G(\tau)$. Extracting the real-frequency spectral function $A(\omega)$ requires solving the inverse problem:

$$G(\tau) = \int d\omega \, K(\tau, \omega) \, A(\omega)$$

where $K$ is a known kernel. This is an ill-conditioned problem. ALF provides both stochastic analytic continuation (SAC) and maximum entropy (MaxEnt) methods via `Max_SAC.out`.

## Usage

Analytic continuation is performed after the standard error analysis, on the Jackknife-analyzed time-displaced data.

### Parameters

Two namelists in the `parameters` file control the analytic continuation:

**`&VAR_Max_Stoch`** — method and frequency grid:

```fortran
&VAR_Max_Stoch
OM_st       = -10.d0      ! Start of frequency grid
OM_en       =  10.d0      ! End of frequency grid
Ndis        =  501        ! Number of frequency points
Stochastic  = .true.      ! .true. = SAC, .false. = MaxEnt
Ngamma      =  500        ! Number of delta functions in SAC
NBins       =  10         ! Number of bins for SAC sampling
NSweeps     =  1000       ! MC sweeps per bin
Nwarm       =  500        ! Warmup sweeps
N_alpha     =  20         ! Number of temperatures (must be > 10)
alpha_st    =  100.d0     ! Starting alpha
R           =  1.3d0      ! Ratio for geometric temperature set: alpha_n = alpha_st * R^(n-1)
Tolerance   =  0.01d0     ! Relative-error cutoff for data rescaling
Checkpoint  = .false.     ! Keep dump files
/
```

**`&VAR_errors`** — data handling:

```fortran
&VAR_errors
N_skip      =  0          ! Bins to skip
N_rebin     =  1          ! Rebinning factor
N_cov       =  0          ! 1 = use full covariance matrix from g_dat
N_Back      =  0          ! Background subtraction
N_auto      =  0          ! Autocorrelation handling
/
```

### Running

The script `Scripts_and_Parameters_files/Spectral.sh` automates the process for k-resolved spectral functions:

1. Loops over k-points from the analyzed time-displaced data
2. Substitutes MaxEnt parameters into an input file
3. Runs `Max_SAC.out` for each k-point
4. Collects output into a single file suitable for plotting

```bash
# After running ana.out or ana_hdf5.out:
bash $ALF_DIR/Scripts_and_Parameters_files/Spectral.sh
```

## Kernels

The kernel channel is a string read from the `g_dat` file header and dispatched automatically — you do not choose it manually.

| Channel | Kernel | Used for |
|---------|--------|----------|
| `PH` (particle-hole) | $K(\tau,\omega) = \frac{e^{-\tau\omega} + e^{-(\beta-\tau)\omega}}{\pi(1 + e^{-\beta\omega})}$ | Spin correlations, density correlations |
| `PH_C` (conductivity) | Same as `PH` | Optical conductivity |
| `PP` (particle-particle) | $K(\tau,\omega) = \frac{e^{-\tau\omega}}{\pi(1 + e^{-\beta\omega})}$ | Pairing correlations |
| `P` (particle) | $K(\tau,\omega) = \frac{e^{-\tau\omega}}{\pi(1 + e^{-\beta\omega})}$ | Single-particle Green's function |
| `P_PH` (particle, p-h symmetric) | $K(\tau,\omega) = \frac{e^{-\tau\omega} + e^{-(\beta-\tau)\omega}}{\pi(1 + e^{-\beta\omega})}$ | Single-particle Green's function at half filling |
| `T0` (zero temperature) | $K(\tau,\omega) = \frac{e^{-\tau\omega}}{\pi}$ | $T = 0$ (PQMC) spectral functions |

For channels `PH`, `P_PH`, and `PH_C`, `OM_st` is forced to `0`.

## Practical Tips

- **Frequency grid:** Make sure `OM_st` and `OM_en` bracket the expected spectral weight. If the spectrum shows increasing spectral weight at the boundary, the window is likely too small — ideally the spectrum should decay towards both `OM_st` and `OM_en`. Too wide wastes resolution.
- **Stochastic vs. MaxEnt:** SAC (`Stochastic=.true.`) is generally more robust and is the default. MaxEnt (`Stochastic=.false.`) may give sharper features but is more sensitive to noise.
- **Covariance matrix:** Using the full covariance (`N_cov=1`) can improve results when correlations between time slices are significant, but requires enough bins for a reliable covariance estimate.
- **Quality check:** Always compare the back-transformed $G(\tau)$ from the obtained $A(\omega)$ against the original QMC data to assess the reliability of the continuation.
