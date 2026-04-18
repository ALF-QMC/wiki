# Troubleshooting and FAQ

Common issues and their solutions.

## Build Issues

### "Environment variable ALF_FC not set"
You forgot to source `configure.sh` before running `make`.
```bash
source configure.sh GNU noMPI    # or your desired configuration
make
```

### "Compiler 'xyz' does not exist"
The compiler set by `configure.sh` is not installed or not in your PATH. Install the compiler or load the appropriate module on your cluster.

### "Linear algebra libraries not found"
LAPACK/BLAS are not installed or not found by the compiler.
- **macOS:** `brew install lapack openblas`
- **Linux:** `sudo apt install liblapack-dev libblas-dev`
- **Cluster:** Load the MKL module (e.g., `module load mkl`)

### HDF5 download fails
The auto-download requires `curl` or `wget` and internet access. On compute nodes without internet, build HDF5 on a login node first:
```bash
source configure.sh GNU noMPI HDF5    # downloads and builds HDF5
```
Then use the same `configure.sh` invocation on compute nodes — the library will already be present.

### "HDF5 installed without compression capabilities"
The HDF5 build did not find zlib. Output will work but files won't be compressed. Install zlib development headers and rebuild HDF5:
- **macOS:** `brew install zlib`
- **Linux:** `sudo apt install zlib1g-dev`

Then delete the HDF5 directory for your compiler in `ALF/HDF5/` and re-run `configure.sh` with `HDF5`.

## Runtime Issues

### "Try with smaller Nwrap or dtau"
The Green's function deviation exceeds the internal threshold (10). This indicates numerical instability. Fixes:
1. **Reduce `Nwrap`** — stabilize more frequently (e.g., from 10 to 5)
2. **Reduce `Dtau`** — finer imaginary-time discretization
3. **Try a different stabilization scheme** — `LOG` mode extends accessible parameter ranges

See [[Stabilization Parameters]] for details.

### NaN or Inf in output
Usually a sign of severe numerical instability. Check:
- Is `Dtau` too large for the interaction strength?
- Is `Nwrap` too large?
- Are physical parameters reasonable (e.g., `ham_U` not astronomically large)?

### Simulation runs but results look wrong
- **Check the `info` file.** Look at acceptance rates — very low acceptance (<10%) means the Markov chain is stuck.
- **Check precision.** The `info` file reports the precision of the Green's function. Large values indicate numerical problems.
- **Validate against known results.** Run a small system where exact diagonalization results are available.

### Simulation is very slow
- **MPI scales with bins.** Using more MPI ranks than `NBin` wastes resources.
- **OpenMP scaling.** ALF uses OpenMP for linear algebra. Beyond 4–6 threads, returns diminish.
- **Checkerboard decomposition.** If applicable for your model, `Checkerboard = .T.` can significantly reduce the cost per sweep.

## Analysis Issues

### "confout" and "confout.h5" both exist
The `out_to_in.sh` script refuses to run if both HDF5 and plain-text configuration files coexist in the same directory. Remove the one that doesn't match your build.

### Missing `h5py` or `f90nml` for convert_bins.py
```bash
pip3 install h5py f90nml
```

## Frequently Asked Questions

### How do I restart a simulation?
Run `out_to_in.sh` (or `out_to_in_temper.sh` for tempering) to rename output configurations to input configurations, then relaunch. ALF automatically detects `confin_*` files and resumes.

### Can I change parameters between restarts?
Only parameters that don't affect the configuration format (e.g., `NSweep`, `NBin`). Changing `Dtau`, `Beta`, lattice size, or the model requires starting from scratch.

### How many bins/sweeps do I need?
- **NBin ≥ 20** for reliable Jackknife error estimates.
- **NSweep** should be large enough that consecutive bins are uncorrelated. Check by increasing `N_rebin` in `&VAR_errors` — if errors grow with rebinning, bins are correlated and `NSweep` should be increased.

### What is the sign problem?
Some models (e.g., frustrated systems, finite-density fermions) have a fluctuating sign of the fermion determinant, causing exponentially growing statistical errors. ALF reports the average sign in the `info` file. If it's much less than 1, the sign problem is severe and results at that parameter point may be unreliable.

### Where can I get help?
- [Discord server](https://discord.gg/VppWWEPMHa)
- Email: alf@physik.uni-wuerzburg.de
- [GitHub Issues](https://github.com/ALF-QMC/ALF/issues)
