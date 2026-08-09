# Developer Guide

Information for contributors and maintainers of the ALF codebase.

## Getting Started

See [CONTRIBUTING.md](https://github.com/ALF-QMC/ALF/blob/master/CONTRIBUTING.md) for the full community guidelines. Key points:

- ALF is released under **GPL v3** with an attribution clause.
- **Bug fixes and portability patches** are welcome from everyone via pull requests.
- **New features** should be discussed in a GitHub issue first.

## Access Levels

| Level | Who | How to get access |
|-------|-----|-------------------|
| User | Anyone | Clone the public repo |
| Contributor | Anyone | Submit a pull request |
| Developer (write access) | By application | Open a public issue + non-interference declaration |
| Core developer (owner) | By vote | Open a public issue + unanimous vote of existing owners |

## Development Workflow

1. Create a feature branch from `master`
2. Build with the `devel` flag for extra runtime checks: `source configure.sh GNU nompi devel`
3. Run the test suite (see [[Test Suite]])
4. Update `CHANGELOG.md` with your changes
5. Submit a pull request — CI runs automatically

### Building for Development

The `devel` flag in `configure.sh` enables:
- Array bounds checking
- Floating-point exception traps (NaN, overflow)
- Additional compiler warnings

```bash
source configure.sh GNU nompi devel
make cleanlib cleanprog
make -j5 program
```

### Code Style

- Fortran 2008 with submodule support
- Modules use `_mod` suffix, submodules use `_smod` suffix
- Hamiltonians are submodules of `Hamiltonian_main`
- Use `Kind=Kind(0.d0)` for double precision
- MPI/OpenMP code is guarded by preprocessor macros (`#ifdef MPI`, `#ifdef TEMPERING`)

## Sub-Pages

- [[Test Suite]] — Running and writing tests
- [[Code Architecture]] — Module structure and data flow
- [[Release Process]] — Versioning and release cycle
