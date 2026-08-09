# Tempering

Parallel tempering configuration in ALF.

## Overview

Parallel tempering (replica exchange) runs multiple simulations at different parameter values simultaneously and periodically proposes swaps between neighboring replicas. This helps overcome energy barriers and reduces autocorrelation in systems that are difficult to sample at a single parameter point.

Tempering in ALF requires MPI. Configure with:

```bash
source configure.sh GNU Tempering HDF5
```

This sets both `-DMPI` and `-DTEMPERING` preprocessor flags.

## Parameters

Set in the `&VAR_TEMP` namelist in the `parameters` file.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `N_exchange_steps` | `6` | Number of exchange (swap) moves attempted per exchange event |
| `N_Tempering_frequency` | `10` | Frequency of exchange events (in sweeps) |
| `mpi_per_parameter_set` | `2` | Number of MPI ranks per replica |
| `Tempering_calc_det` | `.true.` | Include the fermion determinant ratio in the exchange acceptance. Set to `.false.` if the varied parameter only enters the bosonic action $S_0$ |

### Example

```fortran
&VAR_TEMP
N_exchange_steps       = 6
N_Tempering_frequency  = 10
mpi_per_parameter_set  = 2
Tempering_calc_det     = .T.
/
```

## Directory Structure

Tempering runs use a specific directory layout. See `Scripts_and_Parameters_files/Start_Tempering/` for a template:

```
Run/
├── parameters          # shared parameter file (includes &VAR_TEMP)
├── seeds               # random seeds
├── out_to_in_temper.sh # restart helper
├── Temp_0/
│   └── parameters      # parameters for replica 0
├── Temp_1/
│   └── parameters      # parameters for replica 1
└── ...
```

Each `Temp_N/` directory contains a `parameters` file with the parameter values specific to that replica (e.g., different `Beta`, coupling strength, or chemical potential). The shared `parameters` file in the root sets global options including `&VAR_TEMP`.

## Running

The total number of MPI ranks must be a multiple of `mpi_per_parameter_set`:

```bash
# 4 replicas × 2 MPI ranks each = 8 total
mpirun -np 8 $ALF_DIR/Prog/ALF.out
```

ALF splits the MPI communicator: `replica_index = rank / mpi_per_parameter_set`. Each replica writes output to its `Temp_N/` subdirectory.

## Guidelines

### Temperature Grid

- **Overlap is essential.** Adjacent replicas must have sufficient overlap in their energy distributions for exchanges to be accepted. If replicas are too far apart in parameter space, exchange acceptance drops to zero.
- **Denser near phase transitions.** In the vicinity of a phase transition, energy distributions change rapidly. Place replicas closer together there.
- **Start with a few replicas** and check exchange acceptance. Add more if acceptance is low.

### Exchange Acceptance

- **Target: ~20–50% exchange acceptance** between neighboring replicas.
- Check `Acceptance Tempering` in the `info` file (written to each `Temp_N/info`).
- If acceptance is too low: add intermediate replicas or reduce the parameter spacing.
- `N_exchange_steps` controls how many swap attempts per exchange event. Increasing it gives more chances for exchanges but adds overhead.

### Fermion Determinant

- **`Tempering_calc_det = .true.`** (default): the exchange probability includes the ratio of fermion determinants. This is correct when the varied parameter affects the fermionic action (e.g., varying coupling strength or chemical potential).
- **`Tempering_calc_det = .false.`**: only the bosonic action $S_0$ enters the exchange probability. Use this when the varied parameter only affects the Ising field dynamics, not the fermion determinant. This is much cheaper per exchange.

## Restarts

Use the tempering-specific restart script:

```bash
bash out_to_in_temper.sh
```

This loops over all `Temp_*/` directories and renames `confout_*` → `confin_*` in each.

## Known Pitfalls

- **Langevin + Tempering incompatibility.** Langevin mode automatically sets `N_exchange_steps = 0`, effectively disabling tempering. Use sequential or HMC updates with tempering.
- **MPI rank count mismatch.** The total number of MPI ranks must be exactly divisible by `mpi_per_parameter_set`. ALF will error out otherwise.
- **Missing Temp_N directories.** Each replica needs its own `Temp_N/` directory with a valid `parameters` file before launch.
- **Insufficient replicas.** With too few replicas (or too large parameter spacing), the simulation degenerates into independent runs with no useful exchanges.
