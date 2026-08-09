# Running with pyALF

[pyALF](https://github.com/ALF-QMC/pyALF) is a Python interface that wraps ALF's parameter generation, compilation, execution, and analysis into a single scripted workflow. It is the recommended way to run ALF for most users.

Full documentation: [pyALF docs](https://alf.physik.uni-wuerzburg.de/pyalf-doc/source/front.html)

> **pyALF requires HDF5.** ALF automatically downloads and compiles HDF5 on first compilation (~15 minutes).

## Installation of pyALF

```bash
pip install pyALF
```

## Minimal Example

```python
from py_alf import ALF_source, Simulation

# Point to your existing ALF source directory
alf_src = ALF_source(alf_dir="/path/to/ALF")

# Create a simulation with custom parameters
sim = Simulation(
    alf_src,
    "Hubbard",                      # Hamiltonian name
    {"Lattice_type": "Square"},     # Override default parameters
    machine='GNU'                   # Compiler: 'GNU', 'intel', or 'PGI'
)

sim.compile()       # configures and compiles ALF (first run also builds HDF5)
sim.run()           # runs the simulation
sim.analysis()      # post-processes results
```

pyALF creates a directory tree, writes the `parameters` file, compiles ALF (if needed), runs the simulation, and calls the analysis programs — all from the Python script.

## Working with Results

After analysis, read results into a Pandas DataFrame:

```python
obs = sim.get_obs()

# Access the internal energy and its error
obs.iloc[0][['Ener_scal0', 'Ener_scal0_err']]
```

Simulations can be resumed by calling `sim.run()` again — new bins are appended to existing data.

## Further Reading

The [pyALF documentation](https://alf.physik.uni-wuerzburg.de/pyalf-doc/source/front.html) covers parameter scans, parallel runs, Jupyter notebooks, and advanced configuration. This wiki focuses on the underlying ALF code; see [[Running without pyALF]] for the direct Fortran workflow.
