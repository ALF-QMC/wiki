# Running on Clusters

Job scripts and practical tips for running ALF on HPC systems. Example scripts are provided in `Scripts_and_Parameters_files/`.

## General Advice

- **ALF does not profit from hyper-threading.** Set `--ntasks-per-core=1` or equivalent.
- **MPI parallelizes over bins.** Each MPI rank runs an independent Markov chain. Use as many MPI ranks as bins (`NBin`), not more.
- **OpenMP** is enabled by default for GNU and Intel builds. Set `OMP_NUM_THREADS` (or `--cpus-per-task` in SLURM) to control threads per rank.
- **Thread pinning matters.** Intel MPI benefits from pinning settings (see examples below). For OpenMPI, use `--bind-to core`.
- **Restarts:** Use `out_to_in.sh` to rename `confout_*` → `confin_*` before resubmitting. ALF detects `confin_*` and continues from the saved configuration.
- **Tempering runs** use `out_to_in_temper.sh` which loops over `Temp_*/` subdirectories.

## Cluster-Specific Recipes

### Fritz (NHR@FAU)

```bash
#!/bin/bash -l
#SBATCH --partition=<PARTITION>
#SBATCH --nodes=<Nnodes>
#SBATCH --ntasks-per-node=<NtaskPnode>
#SBATCH --cpus-per-task=<Nthreads>
#SBATCH --time=24:00:00
#SBATCH --export=NONE

unset SLURM_EXPORT_ENV
module load intel intelmpi mkl

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
export KMP_AFFINITY=granularity=fine,compact
export I_MPI_PIN_CELL=core
export I_MPI_PIN_DOMAIN=auto:cache
export I_MPI_PIN_ORDER=scatter

bash ./out_to_in.sh
srun ./ALF.out
```

Configure with: `source configure.sh FRITZ MPI HDF5`

### JURECA (JSC)

```bash
#!/bin/bash -x
#SBATCH --partition=batch
#SBATCH --nodes=<Nnodes>
#SBATCH --ntasks-per-node=<NtaskPnode>
#SBATCH --cpus-per-task=<Nthreads>
#SBATCH --time=24:00:00

module load Intel IntelMPI imkl

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
export KMP_AFFINITY=granularity=fine,compact
export I_MPI_PIN_CELL=core
export I_MPI_PIN_DOMAIN=auto:cache3
export I_MPI_PIN_ORDER=scatter

./out_to_in.sh >/dev/null 2>&1
srun ./ALF.out
```

24 cores per node. Useful thread counts: 1, 2, 4, 6, 12.

Configure with: `source configure.sh JUWELS MPI HDF5` (loads modules automatically).

### SuperMUC-NG (LRZ)

```bash
#!/bin/bash
#SBATCH --partition=general
#SBATCH --nodes=<Nnodes>
#SBATCH --ntasks-per-node=<NtaskPnode>
#SBATCH --ntasks-per-core=1
#SBATCH --cpus-per-task=<Nthreads>
#SBATCH --time=24:00:00
#SBATCH --account=<projectID>

module load slurm_setup
module load hdf5/1.10.7-intel21

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
export KMP_AFFINITY=granularity=fine,compact
export I_MPI_PIN_CELL=core
export I_MPI_PIN_DOMAIN=auto:cache3
export I_MPI_PIN_ORDER=scatter
unset FORT_BLOCKSIZE

bash ./out_to_in.sh
mpiexec -n $SLURM_NTASKS ./ALF.out
```

48 cores per node. Useful thread counts: 1, 2, 4, 6, 12, 24.

Configure with: `source configure.sh SuperMUC-NG MPI` (loads modules and HDF5 automatically).

## Batch Job Chains

ALF provides `Start_chain.c` for submitting parameter scans. It reads a `Sims` file where each row defines a set of parameter substitutions, then for each row:

1. Creates a run directory
2. Copies `parameters` and `seeds` from `Start/`
3. Substitutes placeholder variables in `parameters` and the job script
4. Submits with `sbatch`

This is useful for systematic scans over coupling constants, temperatures, or lattice sizes.

## Adapting for Your Cluster

To adapt the provided scripts:

1. Copy the closest existing job file from `Scripts_and_Parameters_files/`
2. Update the `module load` lines for your environment
3. Set the correct partition, account, and time limits
4. Adjust `--ntasks-per-node` and `--cpus-per-task` for your hardware
5. If using Intel MPI, keep the pinning variables. For OpenMPI, use `--bind-to core --map-by slot`

If your cluster is not yet listed in `configure.sh`, you have two options:

- **Quick start:** Use one of the generic compiler entries (`GNU` or `Intel`) and load modules manually before sourcing `configure.sh`.
- **Permanent setup:** Add a custom `MACHINE` block to `configure.sh`. Search for an existing machine (e.g., `FRITZ`) and duplicate its block, then adjust the module loads, compiler paths, and library flags for your environment. This way you can simply run `source configure.sh MYCLUSTER MPI HDF5` and the correct environment is set up automatically.
