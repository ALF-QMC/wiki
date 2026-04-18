# Bin Conversion

Converting plain-text simulation output to HDF5 format.

## Why Convert?

If you ran a simulation without HDF5 and want to switch to the HDF5 analysis pipeline (or simply consolidate output), you can convert existing plain-text bins to `data.h5`.

## convert_bins.py

The main conversion tool. Requires Python 3 with `h5py` and `f90nml`.

```bash
# In the simulation directory (where bin files and 'parameters' live):
$ALF_DIR/Analysis/convert_bins.py
```

This:
1. Reads the `parameters` file to extract simulation metadata
2. Converts all `*_scal`, `*_eq`, and `*_tau` bin files into a single `data.h5`
3. Moves old plain-text bin files to an `old_bins/` subdirectory

### Options

```bash
$ALF_DIR/Analysis/convert_bins.py --remove-old-bins   # delete old bins instead of moving them
```

### Batch Conversion

To convert all simulation directories in a tree at once:

```bash
$ALF_DIR/Analysis/convert_batch.sh
```

This finds all directories containing `Ener_scal` (indicating a completed run) and runs `convert_bins.py` in each.

Alternatively:

```bash
find . -name "Ener_scal" -execdir $ALF_DIR/Analysis/convert_bins.py \;
```

## Individual Converters

For finer control, the individual Fortran converters can be called:

| Program | Converts |
|---------|----------|
| `convert_scal.out` | `*_scal` files |
| `convert_latt.out` | `*_eq` and `*_tau` files (lattice observables) |
| `convert_local.out` | Local observables |

These are called internally by `convert_bins.py` and rarely need to be used directly.

## Notes

- The `parameters` file must be present in the same directory as the bin files.
- For Hubbard model runs with `Mz=.true.`, the converter applies a special fix (halving `N_SUN` and setting `N_FL=2`) to match the internal data layout.
