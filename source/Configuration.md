# Configuration

Options for `configure.sh` and the ALF build system.

## configure.sh Usage

```bash
source configure.sh <MACHINE> [MODE] [STAB] [options...]
```

You **must** source (not execute) this script — it sets environment variables that the Makefile reads. Arguments are case-insensitive and order-independent (except MACHINE must come first or be the only positional argument).

## Machine / Compiler

| MACHINE | Compiler | Linear Algebra | Notes |
|---------|----------|----------------|-------|
| `GNU` | `gfortran` / `mpifort` | `-llapack -lblas` | Most common for local builds |
| `Intel` | `ifort` / `mpiifort` | MKL (`-qmkl` or `-mkl`) | Classic Intel Fortran |
| `IntelLLVM` or `IntelX` | `ifx` / `mpiifort -fc=ifx` | MKL (`-qmkl`) | New LLVM-based Intel Fortran |
| `PGI` | `pgfortran` / `mpifort` | `-llapack -lblas` | NVIDIA/PGI compilers |
| `SuperMUC-NG` | `mpiifort` | MKL | Auto-loads LRZ modules |
| `JUWELS` | `mpiifort` | MKL | Auto-loads JSC modules |
| `FRITZ` | Intel (auto-detected) | MKL | Auto-loads NHR@FAU modules |

If MACHINE is unrecognized, ALF falls back to `gfortran` serial mode.

## Build Mode

| MODE | Preprocessor Flags | Description |
|------|-------------------|-------------|
| `MPI` (default) | `-DMPI` | MPI parallelization across bins |
| `noMPI` | *(none)* | Serial execution |
| `Tempering` | `-DMPI -DTEMPERING` | Parallel tempering (requires MPI) |
| `PP` or `PARALLEL_PARAMS` | `-DMPI -DTEMPERING -DPARALLEL_PARAMS` | Parallel runs with different parameters (requires MPI) |

## Stabilization Scheme

| STAB | Flag | Description |
|------|------|-------------|
| *(default)* | *(none)* | Standard stabilization |
| `STAB1` | `-DSTAB1` | Older UDV decomposition |
| `STAB2` | `-DSTAB2` | UDV with additional normalizations |
| `STAB3` | `-DSTAB3` | Newest: separates large and small scales |
| `LOG` | `-DSTABLOG` | Log storage for internal scales — extends accessible parameter ranges ($\beta$, interaction strength) |

See [[Stabilization Parameters]] for guidance on choosing a scheme.

## Optional Flags

| Flag | Effect |
|------|--------|
| `HDF5` | Enable HDF5 output. ALF auto-installs HDF5 locally if not yet built for the current compiler. |
| `Devel` | Enable development/debug flags: warnings, bounds checking, backtraces |
| `NO-INTERACTIVE` | Suppress interactive prompts (e.g., auto-accept HDF5 download). Useful in scripts. |
| `NO-FALLBACK` | Return error instead of falling back to serial gfortran for unknown machines |

## Custom Compiler Flags

To pass additional flags to the compiler:

```bash
export ALF_FLAGS_EXT="-my-flag"
source configure.sh GNU MPI
```

## Custom HDF5 Directory

By default, ALF installs HDF5 into `ALF/HDF5/<compiler_version>/`. To override:

```bash
export ALF_HDF5_DIR=/path/to/custom/hdf5
source configure.sh GNU MPI HDF5
```

## Environment Variables Set by configure.sh

After sourcing, these are exported and used by `make`:

| Variable | Purpose |
|----------|---------|
| `ALF_DIR` | Root directory of ALF |
| `ALF_FC` | Fortran compiler command |
| `ALF_FLAGS_PROG` | Compiler flags for the main program |
| `ALF_FLAGS_MODULES` | Compiler flags for library modules |
| `ALF_FLAGS_ANA` | Compiler flags for analysis tools |
| `ALF_FLAGS_QRREF` | Compiler flags for QR library |
| `ALF_LIB` | Linker flags (libraries) |

## Compiler-Specific Notes

### GNU (gfortran)
- Default optimization: `-O3 -ffast-math -ffree-line-length-none`
- OpenMP enabled by default (`-fopenmp`)
- Devel mode adds: `-Wconversion -Werror -fcheck=all -g -fbacktrace`

### Intel Classic (ifort)
- Default optimization: `-O3 -fp-model fast=2 -xHost -ipo -ip`
- OpenMP enabled (`-parallel -qopenmp`)
- Devel mode adds: `-warn all -check all -g -traceback`

### Intel LLVM (ifx)
- Default optimization: `-O3 -fp-model=fast=2 -xHost`
- OpenMP **not** enabled by default (can be uncommented in `configure.sh`)
- Devel mode adds: `-warn all -check all,nouninit -g -traceback`

## Common Recipes

```bash
# Local development (serial, debug checks, fast iteration)
source configure.sh GNU noMPI devel

# Production run (MPI + HDF5)
source configure.sh GNU MPI HDF5

# Parallel tempering
source configure.sh GNU Tempering HDF5

# Non-interactive (CI/scripting)
source configure.sh GNU MPI HDF5 NO-INTERACTIVE
```
