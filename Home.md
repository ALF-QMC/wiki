# ALF — Algorithms for Lattice Fermions

Welcome to the ALF wiki! This is the practical companion to the [formal documentation (PDF)](https://alf.physik.uni-wuerzburg.de/doc.pdf). Here you'll find installation guides, tutorials, parameter tuning advice, and developer resources.

**Project website**: https://alf.physik.uni-wuerzburg.de/

## Getting Started

- **[[Installation]]** — Build ALF from source on macOS or Linux
- **[[Quick Start]]** — Run your first simulation in minutes
  - [[Running with pyALF]] — Python interface (recommended for new users)
  - [[Running without pyALF]] — Direct Fortran workflow
- **[[Configuration]]** — Compiler options, MPI, HDF5, and build modes

## Using ALF

- **[[Production Run Cycle]]** — Systematic study of an existing model from setup to results
- **[[Writing a New Model]]** — Implement your own Hamiltonian using the ALF framework
  - [[Predefined Lattices]] — Square, Honeycomb, Bilayer, Triangular, Kagome, and more
  - [[Predefined Observables]] — Equal-time and time-displaced measurements
- **[[Analysis Tools]]** — Post-processing, binning, and analytic continuation
  - [[HDF5 Output Format]] — Structure of simulation output files
  - [[Bin Conversion]] — Converting and rebinning data
  - [[Analytic Continuation]] — Maximum entropy methods

## Tuning & Best Practices

Practical advice on choosing simulation parameters:

- **[[Tuning and Best Practices]]** — Overview and general guidance
  - [[Discretization]] — Dtau and Trotter error tradeoffs
  - [[Stabilization Parameters]] — Nwrap and numerical stabilization
  - [[HMC Parameters]] — Step size, leap-frog steps, mass matrix
  - [[Tempering]] — Parallel tempering configuration

## Operations

- **[[Running on Clusters]]** — Job scripts and tips for HPC systems
- **[[Troubleshooting and FAQ]]** — Common issues and solutions

## Development

- **[[Developer Guide]]** — Contributing to ALF
  - [[Test Suite]] — Running and writing tests
  - [[Code Architecture]] — Module structure and data flow
  - [[Release Process]] — Versioning and release cycle
- **[[Glossary]]** — QMC terminology and ALF-specific concepts