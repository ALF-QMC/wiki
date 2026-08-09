# Glossary

Terminology used in ALF and auxiliary-field quantum Monte Carlo.

| Term | Definition |
|------|------------|
| **AFQMC** | Auxiliary-field quantum Monte Carlo — the family of methods ALF implements |
| **Amplitude** | Width of the box distribution for proposing continuous auxiliary field updates (`&VAR_QMC`) |
| **Beta** | Inverse temperature $\beta = 1/T$ (`&VAR_Model_Generic`) |
| **Bin** | A block of `NSweep` Monte Carlo sweeps. Observables are averaged within each bin; error bars come from bin-to-bin fluctuations (Jackknife) |
| **Bravais lattice** | A periodic lattice defined by primitive vectors and a unit cell. See [[Predefined Lattices]] |
| **Bulk** | Twist-angle implementation flag: `.true.` = vector potential in bulk, `.false.` = boundary condition (`&VAR_Model_Generic`) |
| **Calc_Fl** | Boolean array indicating which flavors require explicit Green function computation |
| **Channel** | String (`'P'`, `'PH'`, `'PP'`, `'PH_C'`, `'P_PH'`, `'T0'`) selecting the analytic continuation kernel. See [[Analytic Continuation]] |
| **Checkerboard** | Decomposition of the hopping matrix into commuting sub-blocks for efficient exponentiation (`&VAR_Model_Generic`) |
| **confin / confout** | Configuration input/output files. `confout_N` saves the auxiliary field state for MPI rank `N`; renamed to `confin_N` for restarts |
| **CPU_MAX** | Wall-clock time limit in hours. ALF exits cleanly when this is reached (`&VAR_QMC`) |
| **Dtau** | Imaginary-time discretization step $\Delta\tau$. Controls the Trotter error (`&VAR_Model_Generic`) |
| **Green's function** | The central quantity: $G_{ij} = \langle c_i c_j^\dagger \rangle$. Updated incrementally and recomputed periodically for stability |
| **HMC** | Hybrid (Hamiltonian) Monte Carlo — a global update scheme using molecular dynamics trajectories. See [[HMC Parameters]] |
| **Hubbard-Stratonovich** | Transformation that decouples the interaction into auxiliary fields, enabling determinantal QMC |
| **Langevin** | Stochastic update scheme for continuous auxiliary fields. Cannot be combined with sequential updates |
| **LOBS_ST / LOBS_EN** | Start and end time slices for measurements (`&VAR_QMC`) |
| **Ltau** | Flag: `1` = compute time-displaced correlations, `0` = equal-time only (`&VAR_QMC`) |
| **Ltrot** | Total number of imaginary-time slices: $L_\text{trot} = \text{nint}(\beta/\Delta\tau) + 2\Theta_\text{trot}$ |
| **N_FL** | Number of fermion flavors. Propagation is block-diagonal in flavor (`&VAR_Model_Generic`) |
| **N_SUN** | Number of colors in SU(N) symmetry. Propagation is color-independent (`&VAR_Model_Generic`) |
| **NBin** | Number of bins per simulation (`&VAR_QMC`) |
| **Ndim** | Total number of orbitals: (unit cells) × (orbitals per cell) |
| **NSweep** | Number of MC sweeps per bin (`&VAR_QMC`) |
| **Nwrap** | Number of time slices between QR stabilizations. See [[Stabilization Parameters]] |
| **Op_T** | Hopping (kinetic) operators $e^{T_n}$ — defined per Hamiltonian |
| **Op_V** | Interaction operators $e^{V_n(\tau)}$ — coupled to auxiliary fields |
| **Phi_X / Phi_Y** | Twist angles in units of flux quanta (`&VAR_Model_Generic`) |
| **Projector** | If `.true.`, use projective (zero-temperature) QMC instead of finite-temperature (`&VAR_Model_Generic`) |
| **RUNNING** | Lock file created by ALF at startup, removed on clean exit. Prevents duplicate runs |
| **Sequential** | Local (single-field) update scheme — the default starting point. See [[Tuning and Best Practices]] |
| **Sign problem** | When the fermion determinant weight becomes negative (or complex), sampling efficiency degrades exponentially with system size and inverse temperature |
| **Sweep** | One pass through all auxiliary fields on all time slices |
| **Symm** | Symmetric Trotter decomposition flag. When `.true.`, the Green function is symmetrized before measurements (`&VAR_Model_Generic`) |
| **Tempering** | Parallel tempering — exchanges configurations between parameter sets to improve ergodicity. See [[Tempering]] |
| **Theta** | Projection parameter $\Theta$ for projective QMC; controls ground-state projection accuracy (`&VAR_Model_Generic`) |
| **Thtrot** | Projection parameter in units of $\Delta\tau$: $\Theta_\text{trot} = \text{nint}(\Theta/\Delta\tau)$ |
| **Trotter error** | Systematic error from the imaginary-time discretization, $O(\Delta\tau^2)$ or $O(\Delta\tau)$ depending on the decomposition. See [[Discretization]] |
| **UDV** | Matrix decomposition $M = U \cdot D \cdot V$ used for numerical stabilization of the Green's function |
| **Warmup** | Initial sweeps discarded to let the Markov chain equilibrate before measurements begin |
| **WF_L / WF_R** | Left and right trial wave functions for projective QMC |
