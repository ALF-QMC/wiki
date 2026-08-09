# Installation

Step-by-step instructions for building ALF from source.

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Fortran compiler** | `gfortran` (GNU), `ifort`/`ifx` (Intel), or `pgfortran` (PGI) |
| **LAPACK + BLAS** | Linear algebra libraries (or Intel MKL) |
| **Python 3** | Required by the build system |
| **MPI** (optional) | For parallel runs: OpenMPI or Intel MPI (`mpifort`) |
| **HDF5** (optional) | ALF can auto-download and compile HDF5 locally — see below |
| **curl or wget** | Needed only if HDF5 auto-install is used |

## macOS

Using [Homebrew](https://brew.sh/):

```bash
brew install gcc                    # provides gfortran
brew install lapack openblas        # linear algebra
brew install open-mpi               # optional, for MPI parallelization
brew install python3                # if not already present
```

## Linux (Debian/Ubuntu)

```bash
sudo apt install gfortran liblapack-dev libblas-dev python3
sudo apt install libopenmpi-dev     # optional, for MPI
```

## Linux (Fedora/RHEL)

```bash
sudo dnf install gcc-gfortran lapack-devel blas-devel python3
sudo dnf install openmpi-devel      # optional, for MPI
```

## Getting the Source

```bash
git clone https://github.com/ALF-QMC/ALF.git
cd ALF
```

## Building ALF

ALF uses a two-step process: **configure** then **make**.

### 1. Configure

```bash
source configure.sh <MACHINE> [MODE] [STAB] [options...]
```

This sets environment variables (`ALF_FC`, `ALF_FLAGS_PROG`, etc.) that the Makefile reads. See [[Configuration]] for full details.

**Minimal examples:**

```bash
# Serial build with GNU compilers
source configure.sh GNU noMPI

# MPI build with GNU compilers
source configure.sh GNU MPI

# MPI + HDF5 output
source configure.sh GNU MPI HDF5
```

### 2. Compile

```bash
make          # builds libraries, program (ALF.out), and analysis tools (ana.out)
```

Or build components individually:

```bash
make lib      # libraries only
make program  # ALF.out only
make ana      # analysis tools only
```

The executable is created at `Prog/ALF.out` and the analysis tool at `Analysis/ana.out`.

### HDF5 Setup

When you pass `HDF5` to `configure.sh`, ALF checks for a locally compiled HDF5 matching your compiler. If not found, it offers to download and build HDF5 automatically into `ALF/HDF5/<compiler_version>/`. This is the recommended approach — ALF does not use system HDF5 installations.

To use a custom HDF5 directory, set `ALF_HDF5_DIR` before sourcing `configure.sh`:

```bash
export ALF_HDF5_DIR=/path/to/your/hdf5
source configure.sh GNU MPI HDF5
```

## Verifying the Installation

Run the test suite to confirm everything works:

```bash
# Serial tests
source configure.sh GNU noMPI
make cleanlib cleanprog && make -j5 all
cd testsuite && mkdir test && cd test
cmake .. && make -j5 && ctest -j5 --output-on-failure
cd ../.. && rm -rf testsuite/test
```

A successful run produces output like:

```
100% tests passed, 0 tests failed out of N
```

See [[Test Suite]] for running MPI and tempering tests.
