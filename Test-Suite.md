# Test Suite

Running and writing tests for ALF. The test suite lives in `testsuite/` and uses CMake + CTest.

## Running Tests

Tests are built and run separately from the main code. Three configurations are available:

### No-MPI tests

```bash
source configure.sh GNU nompi
make cleanlib cleanprog && make -j5 all
cd testsuite && mkdir -p test && cd test
cmake .. && make -j5 && ctest -j5 --output-on-failure
cd ../.. && rm -rf testsuite/test
```

### MPI tests

```bash
source configure.sh GNU mpi
make cleanlib cleanprog && make -j5 all
cd testsuite && mkdir -p test && cd test
cmake .. && make -j5 && ctest -j5 --output-on-failure
cd ../.. && rm -rf testsuite/test
```

### Tempering tests

```bash
source configure.sh GNU tempering
make cleanlib cleanprog && make -j5 all
cd testsuite && mkdir -p test && cd test
cmake .. && make -j5 && ctest -j5 --output-on-failure
cd ../.. && rm -rf testsuite/test
```

> **Tip:** VS Code tasks are provided for all three configurations — see the "testsuite nompi/mpi/tempering" tasks.

## Test Structure

```
testsuite/
├── CMakeLists.txt            # Top-level: enables sub-directories
├── matmod.tests/             # Matrix module unit tests
│   └── CMakeLists.txt
├── Prog.tests/               # QMC operator unit tests
│   └── CMakeLists.txt
├── test_vs_ed/               # QMC vs exact diagonalization regression tests
│   ├── test_specs.yaml       # Test parameters and reference values
│   └── compare_dirs.py       # Comparison utility
└── test_branch_parameters.json
```

### Matrix Module Tests (`matmod.tests/`)

Unit tests for the linear algebra infrastructure: matrix diagonalization, inversion, UDV decomposition. These catch low-level numerical issues early.

### Operator Unit Tests (`Prog.tests/`)

Tests for the QMC operator algebra: `Op_make`, `mmultL`/`mmultR` (operator × Green function), `Wrapup`/`Wrapdo` (time-slice propagation), `cgr` (Green function computation), UDV/QDRP stabilization.

### QMC vs Exact Diagonalization (`test_vs_ed/`)

Regression tests that run short QMC simulations and compare the total energy against exact diagonalization results. Configured via `test_specs.yaml`:

```yaml
hubbard_finT_chain:
  ed_energy: -1.47261997
  max_sigma: 3
  max_delta: 0.002
```

A test passes if the QMC energy agrees with the ED value within `max_sigma` standard deviations **and** `max_delta` absolute deviation.

## CI

GitHub Actions workflows run the test suite automatically:

- **`test-branch.yaml`**: Compares a branch against master using `alf_test_branch` (pyALF). Matrix of environments: Debian (GNU), Intel, PGI, macOS.
- **`test-vs-ed.yaml`**: QMC vs ED regression tests. Three stages: prepare → simulate (parallelized across environments) → analyze.

Both are `workflow_dispatch` (manually triggered).

## Writing New Tests

1. **Unit test**: Add a Fortran test program to `matmod.tests/` or `Prog.tests/`, register it in the corresponding `CMakeLists.txt` with `add_test()`
2. **Regression test**: Add a new entry to `test_vs_ed/test_specs.yaml` with the model parameters and ED reference energy
3. Run `ctest --output-on-failure` to verify
