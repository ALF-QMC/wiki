# Quick Start

Get a simulation running as quickly as possible. This page uses the **Hubbard model on a 6×6 square lattice** as a minimal working example.

> **Prerequisites:** ALF must be compiled first — see [[Installation]].

## Choose Your Workflow

| Approach | Best for | Guide |
|----------|----------|-------|
| **pyALF** (Python) | New users, parameter scans, scripted workflows | [[Running with pyALF]] |
| **Direct** (Fortran) | Full control, HPC jobs, custom setups | [[Running without pyALF]] |

---

## Option A: Quick Start with pyALF

Install [pyALF](https://github.com/ALF-QMC/pyALF) and run:

```python
from py_alf import Simulation

sim = Simulation(
    'Hubbard',
    {
        "Model": "Hubbard",
        "Lattice_type": "Square",
        "L1": 6, "L2": 6,
        "Beta": 5.0,
        "Ham_U": 4.0,
        "Ham_T": 1.0,
        "Nsweep": 200,
        "NBin": 50,
        "Ltau": 0,
        "Mz": False,
    },
    alf_dir='/path/to/ALF',
    machine='gnu',
    mpi=True,
    n_mpi=4,
)

sim.compile()
sim.run()
sim.analysis()

res = sim.get_obs(['Ener_scalJ'])
print(f"Energy: {res['Ener_scalJ']['obs']}")  # [mean, error]
```

---

## Option B: Quick Start Without pyALF

### 1. Build

```bash
cd /path/to/ALF
source configure.sh GNU noMPI
make
```

> **Recommended: Enable HDF5.** The above builds without HDF5 for simplicity. For production use, HDF5 is strongly recommended — it produces a single compressed `data.h5` file instead of many plain-text files. See [Switching to HDF5](#switching-to-hdf5) below. pyALF requires HDF5.

### 2. Set Up a Run Directory

ALF ships with an example setup:

```bash
cp -r Scripts_and_Parameters_files/Start ./Run
cd Run
```

This directory contains two files:
- **`parameters`** — Fortran namelist file defining the model and simulation settings
- **`seeds`** — Random number generator seeds

The default `parameters` file configures a half-filled Hubbard model ($U/t = 4$, $\beta t = 5$) on a 6×6 square lattice:

```fortran
&VAR_ham_name
ham_name = "Hubbard"
/

&VAR_lattice
L1           = 6
L2           = 6
Lattice_type = "Square"
Model        = "Hubbard"
/

&VAR_Model_Generic
N_SUN        = 2
N_FL         = 1
Dtau         = 0.1d0
Beta         = 5.d0
Projector    = .F.
Checkerboard = .T.
Symm         = .T.
/

&VAR_QMC
Nwrap   = 10
NSweep  = 20
NBin    = 5
Ltau    = 1
/

&VAR_errors
n_skip  = 1
N_rebin = 1
/

&VAR_Hubbard
Mz         = .T.
Continuous = .F.
ham_T      = 1.d0
ham_chem   = 0.d0
ham_U      = 4.d0
/
```

### 3. Run

```bash
$ALF_DIR/Prog/ALF.out
```

For MPI builds:

```bash
mpirun -np 4 $ALF_DIR/Prog/ALF.out
```

### 4. Analyze

After the simulation completes, the directory contains raw bin files. Run the analysis tool:

```bash
$ALF_DIR/Analysis/ana.out *    # analyze all observables
```

This produces Jackknife-analyzed output files (e.g. `Ener_scalJ` for total energy with error bars).

### 5. Check Results

The `info` file contains a summary of the run: parameters used, acceptance rates, precision, and walltime. Key scalar results appear in files like:

| File | Observable |
|------|------------|
| `Ener_scalJ` | Total energy (mean ± error) |
| `Kin_scalJ` | Kinetic energy |
| `Pot_scalJ` | Potential energy |
| `Part_scalJ` | Particle number |

Equal-time correlations (e.g. `SpinZ_eqJK` for spin structure factor in k-space) and time-displaced correlations (e.g. `Green_tau`) are also available when `Ltau=1`.

---

## Switching to HDF5

The example above uses plain-text output for simplicity. For anything beyond a first test, **HDF5 is the recommended output format**:

- Single compressed `data.h5` file instead of dozens of plain-text files
- Required by pyALF
- More efficient storage, especially for large lattices and time-displaced observables

To switch, rebuild with the `HDF5` flag (ALF auto-downloads and compiles HDF5 if needed):

```bash
source configure.sh GNU noMPI HDF5
make cleanlib cleanprog && make
```

Everything else stays the same — same `parameters` file, same run command. Only the analysis step changes:

```bash
# HDF5 analysis (instead of 'ana.out *')
$ALF_DIR/Analysis/ana_hdf5.out
```

Results are written to a `res/` subdirectory (e.g. `res/Ener_scalJ`).

---

## Next Steps

- [[Configuration]] — Understand all build and run options
- [[Tuning and Best Practices]] — Choose good simulation parameters
- [[Analysis Tools]] — Detailed analysis workflow
- [[Writing a New Model]] — Implement your own Hamiltonian
