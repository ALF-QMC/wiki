# ALF — Algorithms for Lattice Fermions

Welcome to the ALF wiki! This is the practical companion to the [formal documentation (PDF)](https://alf.physik.uni-wuerzburg.de/doc.pdf). Here you'll find installation guides, tutorials, parameter tuning advice, and developer resources.

**Project website**: https://alf.physik.uni-wuerzburg.de/

<!-- Automatic Table of Contents -->

```{toc}
```

---

<!-- Manual Table of Contents -->

## Getting Started

- **[Installation](./Installation.md)** — Build ALF from source on macOS or Linux
- **[Quick Start](./Quick-Start.md)** — Run your first simulation in minutes
  - **[Running with pyALF](./Running-with-pyALF.md)** — Python interface (recommended for new users)
  - **[Running without pyALF](./Running-without-pyALF.md)** — Direct Fortran workflow
- **[Configuration](./Configuration.md)** — Compiler options, MPI, HDF5, and build modes

## Using ALF

- **[Production Run Cycle](./Production-Run-Cycle.md)** — Systematic study of an existing model from setup to results
- **[Writing a New Model](./Writing-a-New-Model.md)** — Implement your own Hamiltonian using the ALF framework
  - **[Predefined Lattices](./Predefined-Lattices.md)** — Square, Honeycomb, Bilayer, Triangular, Kagome, and more
  - **[Predefined Observables](./Predefined-Observables.md)** — Equal-time and time-displaced measurements
- **[Analysis Tools](./Analysis-Tools.md)** — Post-processing, binning, and analytic continuation
  - **[HDF5 Output Format](./HDF5-Output-Format.md)** — Structure of simulation output files
  - **[Bin Conversion](./Bin-Conversion.md)** — Converting and rebinning data
  - **[Analytic Continuation](./Analytic-Continuation.md)** — Maximum entropy methods

## Tuning & Best Practices

Practical advice on choosing simulation parameters:

- **[Tuning and Best Practices](./Tuning-and-Best-Practices.md)** — Overview and general guidance
  - **[Discretization](./Discretization.md)** — Dtau and Trotter error tradeoffs
  - **[Stabilization Parameters](./Stabilization-Parameters.md)** — Nwrap and numerical stabilization
  - **[HMC Parameters](./HMC-Parameters.md)** — Step size, leap-frog steps, mass matrix
  - **[Tempering](./Tempering.md)** — Parallel tempering configuration

## Operations

- **[Running on Clusters](./Running-on-Clusters.md)** — Job scripts and tips for HPC systems
- **[Troubleshooting and FAQ](./Troubleshooting-and-FAQ.md)** — Common issues and solutions

## Development

- **[Developer Guide](./Developer-Guide.md)** — Contributing to ALF
  - **[Test Suite](./Test-Suite.md)** — Running and writing tests
  - **[Code Architecture](./Code-Architecture.md)** — Module structure and data flow
  - **[Release Process](./Release-Process.md)** — Versioning and release cycle
- **[Glossary](./Glossary.md)** — QMC terminology and ALF-specific concepts