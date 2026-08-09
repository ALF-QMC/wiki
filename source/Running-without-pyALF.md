# Running without pyALF

Running ALF directly using the compiled Fortran executable and parameter files. This gives full control over every aspect of the simulation.

## Directory Setup

A simulation directory needs three files:

```
my_sim/
├── parameters     # Fortran namelists defining the model and QMC parameters
├── seeds          # One random seed per MPI rank (one integer per line)
└── ALF.out        # The compiled executable (or a symlink to it)
```

A ready-made template is provided in `Scripts_and_Parameters_files/Start/`.

## The Parameters File

The `parameters` file uses Fortran namelist syntax. A minimal example for the Hubbard model:

```fortran
&VAR_ham_name
ham_name = "Hubbard"
/
&VAR_lattice
L1 = 6
L2 = 6
Lattice_type = "Square"
Model = ""
/
&VAR_Model_Generic
Dtau   = 0.1d0
Beta   = 10.d0
N_SUN  = 1
N_FL   = 2
Symm   = .true.
/
&VAR_QMC
NSweep     = 50
NBin       = 20
Nwrap      = 10
Ltau       = 1
LOBS_ST    = 1
LOBS_EN    = 1
CPU_MAX    = 24.d0
Sequential = .true.
/
&VAR_Hubbard
Ham_T  = 1.d0
Ham_U  = 4.d0
Mz     = .false.
/
&VAR_errors
n_skip = 0
N_rebin = 1
/
```

### Key Namelists

| Namelist | Purpose |
|----------|---------|
| `&VAR_ham_name` | Selects the Hamiltonian (e.g., `"Hubbard"`, `"Kondo"`, `"tV"`) |
| `&VAR_lattice` | Lattice type and dimensions (see [[Predefined Lattices]]) |
| `&VAR_Model_Generic` | Universal parameters: `Dtau`, `Beta`, `N_SUN`, `N_FL`, twist angles, projector settings |
| `&VAR_QMC` | QMC control: sweeps, bins, stabilization, update schemes (see [[Tuning and Best Practices]]) |
| `&VAR_<Model>` | Model-specific parameters (e.g., `&VAR_Hubbard` for hopping and interaction) |
| `&VAR_errors` | Analysis parameters: `n_skip`, `N_rebin`, `N_Cov` |

## The Seeds File

One integer per line, one per MPI rank. The template in `Scripts_and_Parameters_files/Start/seeds` provides over 2000 seeds — enough for typical production runs. If you run with more MPI ranks than seeds, ALF will generate additional seeds from the provided ones.

## Running

```bash
# Serial (single Markov chain)
./ALF.out

# MPI (one Markov chain per rank)
mpirun -np 20 ./ALF.out
```

ALF creates a `RUNNING` file at startup and removes it on clean exit. If `RUNNING` already exists, ALF refuses to start — this prevents accidental duplicate runs. Delete it manually if a previous run crashed.

## Output Files

### With HDF5

| File | Content |
|------|---------|
| `data.h5` | All observables (scalar, equal-time, time-displaced) in a single HDF5 file |
| `confout_N.h5` | Saved configuration for rank `N` (for restarts) |
| `info` | Run metadata: timings, acceptance rates, stabilization precision |

### Without HDF5

| File | Content |
|------|---------|
| `*_scal` | Scalar observables (one value per bin) |
| `*_eq` | Equal-time correlation functions |
| `*_tau` | Time-displaced correlation functions |
| `confout_N` | Saved configuration for rank `N` (for restarts) |
| `info` | Run metadata: timings, acceptance rates, stabilization precision |

## Restarts

ALF supports warm restarts: new bins are appended to existing output files, and the Markov chain continues from the saved configuration.

```bash
# Rename output configs to input configs
bash out_to_in.sh

# Resubmit — ALF detects confin_* and continues
./ALF.out
```

For tempering runs, use `out_to_in_temper.sh` which loops over all `Temp_*/` subdirectories.

## Batch Parameter Scans

`Scripts_and_Parameters_files/Start_chain.c` (a shell script despite the `.c` extension) automates parameter scans. It reads a `Sims` file where each row defines a parameter set:

```
Y  6  6  4.0  1.0  0.0  0.0  0.0  10.0  0.1  10  50  20  24.0
Y  8  8  4.0  1.0  0.0  0.0  0.0  10.0  0.1  10  50  20  24.0
stop
```

The first column (`Y`/`N`) controls whether to run that row. Remaining columns are substituted into placeholders in `parameters` and the job script via `sed`. Each row gets its own directory with a copy of the `Start/` template.

## Analysis

After a run completes, post-process with the analysis tools:

```bash
# HDF5 output
$ALF_DIR/Analysis/ana_hdf5.out

# Plain-text output
$ALF_DIR/Analysis/ana.out *
```

See [[Analysis Tools]] for details.
