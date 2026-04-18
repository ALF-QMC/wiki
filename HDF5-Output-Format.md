# HDF5 Output Format

Structure of the HDF5 files produced by ALF simulations compiled with the `HDF5` flag.

## Output Files

| File | Content |
|------|---------|
| `data.h5` | All observable bins (scalar, equal-time, time-displaced) in a single compressed file |
| `confout_<N>.h5` | Auxiliary field configuration for MPI rank N (for restarts) |
| `info` | Human-readable run summary (same as without HDF5) |

## data.h5 Structure

The file is organized into HDF5 groups, one per observable. Each group contains the bin data as datasets. The group name matches the observable name (e.g. `Ener_scal`, `Green_eq`, `SpinZ_tau`).

Simulation parameters from the `parameters` file are stored as HDF5 attributes on the root group.

## Analyzing HDF5 Output

```bash
$ALF_DIR/Analysis/ana_hdf5.out              # all observables
$ALF_DIR/Analysis/ana_hdf5.out SpinZ_eq     # specific observable
```

Results are written to a `res/` subdirectory.

## Reading with Python

```python
import h5py

with h5py.File('data.h5', 'r') as f:
    # List all observables
    print(list(f.keys()))

    # Read a dataset
    ener = f['Ener_scal'][:]
    print(ener.shape)  # (NBin, ...)
```

## Compression

When ALF is compiled with `-DHDF5_ZLIB`, output data is compressed using zlib (deflate). This significantly reduces file sizes, especially for large lattices with time-displaced observables. Compression is transparent — tools like `h5py` decompress automatically.

If HDF5 was built without zlib support, a warning is printed during configuration:
> "Warning: HDF5 installed without compression capabilities. The output files will not be compressed!"
