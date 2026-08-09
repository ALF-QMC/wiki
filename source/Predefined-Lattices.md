# Predefined Lattices

Reference for the Bravais lattices available in ALF. These are implemented in `Prog/Predefined_Latt_mod.F90` and selected via the `Lattice_type` string in the `&VAR_lattice` namelist.

## Parameters

```fortran
&VAR_lattice
L1 = 6
L2 = 6
Lattice_type = "Square"
Model = ""
/
```

- `L1`, `L2` — lattice dimensions (number of unit cells in each direction)
- `Lattice_type` — string selecting the lattice (see table below)
- `Model` — optional model variant string (used by some Hamiltonians)

## Available Lattices

| `Lattice_type` | Orbitals per cell | Coord. vectors | Primitive vectors | Notes |
|----------------|:-----------------:|:--------------:|-------------------|-------|
| `"Square"` | 1 | 1 (1D) or 2 (2D) | $\mathbf{a}_1 = (1,0)$, $\mathbf{a}_2 = (0,1)$ | Set `L2=1` for a 1D chain |
| `"N_leg_ladder"` | `L2` | 1 | $\mathbf{a}_1 = (1,0)$ | `L2` orbitals per unit cell, `L1` rungs; 1D geometry |
| `"Bilayer_Square"` | 2 | 2 | $\mathbf{a}_1 = (1,0)$, $\mathbf{a}_2 = (0,1)$ | Two layers with 3D orbital positions |
| `"Triangular"` | 1 | 3 | $\mathbf{a}_1 = (1,0)$, $\mathbf{a}_2 = (1/2, \sqrt{3}/2)$ | 2D only (`L1,L2 > 1`) |
| `"Honeycomb"` | 2 | 3 | $\mathbf{a}_1 = (1,0)$, $\mathbf{a}_2 = (1/2, \sqrt{3}/2)$ | A/B sublattice |
| `"Bilayer_Honeycomb"` | 4 | 3 | $\mathbf{a}_1 = (1,0)$, $\mathbf{a}_2 = (1/2, \sqrt{3}/2)$ | Two honeycomb layers with 3D orbital positions |
| `"Pi_Flux"` | 2 | 4 | $\mathbf{a}_1 = (1,1)$, $\mathbf{a}_2 = (1,-1)$ | 2D only (`L1,L2 > 1`) |
| `"Kagome"` | 3 | 4 | $\mathbf{a}_1 = (1,0)$, $\mathbf{a}_2 = (1/2, \sqrt{3}/2)$ | 2D only (`L1,L2 > 1`) |

The total number of orbitals is `Ndim = L1 * L2 * Norb` where `Norb` is the number of orbitals per unit cell.

## Key Data Structures

After calling `Predefined_Latt`, the following are available:

| Variable | Type | Description |
|----------|------|-------------|
| `Latt` | `Type(Lattice)` | Bravais lattice: unit cell count `N`, neighbor list `nnlist`, distance map `imj` |
| `Latt_unit` | `Type(Unit_cell)` | Unit cell: `Norb`, coordination number `N_coord`, orbital positions `Orb_pos_p` |
| `List(I,1)` | Integer array | Unit cell index of site `I` |
| `List(I,2)` | Integer array | Orbital index of site `I` |
| `Invlist(uc,orb)` | Integer array | Site index of orbital `orb` in unit cell `uc` |
| `Ndim` | Integer | Total number of orbitals (`Latt%N * Latt_unit%Norb`) |

## Defining a Custom Lattice

If the predefined lattices don't fit your needs, you can define a custom lattice directly in your Hamiltonian's `Ham_Set` subroutine using the lattice library in `Libraries/Modules/lattices_v3_mod.F90`:

1. Create a `Type(Unit_cell)` specifying `Norb`, `N_coord`, orbital positions, and coordination vectors
2. Call `Make_Lattice(Latt, Latt_unit, L1, L2)` to build the full lattice
3. Construct the `List`/`Invlist` mapping arrays

See the existing Hamiltonians for examples of this pattern.
