# Discretization

Choosing the imaginary-time discretization `Dtau` and managing Trotter errors. This applies to **every** ALF simulation.

## Parameters

Set in the `&VAR_Model_Generic` namelist in the `parameters` file.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `Dtau` | `0.1` | Imaginary-time step size |
| `Beta` | `5.0` | Inverse temperature ($\beta = 1/T$) |

The number of imaginary-time slices is computed automatically:

$$L_\text{Trot} = \text{nint}(\beta / \Delta\tau)$$

For projective (zero-temperature) simulations with `Projector = .T.` and projection parameter `Theta`, the total number of slices is $L_\text{Trot} + 2 \times \text{nint}(\Theta / \Delta\tau)$.

## Guidelines

### Choosing Dtau

The Trotter decomposition introduces a systematic error of order $\mathcal{O}(\Delta\tau^2)$ in the partition function and observables. This error is **not** a statistical error — it is a bias that does not decrease with more sweeps or bins.

- **Smaller `Dtau`** = smaller Trotter error, but more time slices → more expensive.
- **Larger `Dtau`** = cheaper, but larger systematic bias.
- **`Dtau = 0.1`** is a common starting point for moderate interaction strengths ($U/t \sim 1$–$4$).
- **For strong coupling** ($U/t > 6$) or high precision, reduce to `Dtau = 0.05` or smaller.

### Dtau Extrapolation

For publication-quality results, run at multiple `Dtau` values and extrapolate to $\Delta\tau \to 0$:

1. Run at e.g. `Dtau = 0.2, 0.1, 0.05`
2. Plot the observable vs. $\Delta\tau^2$
3. Fit a line and extrapolate to $\Delta\tau^2 = 0$

Since the error is $\mathcal{O}(\Delta\tau^2)$, a linear fit in $\Delta\tau^2$ is appropriate. If the results at your chosen `Dtau` already agree within error bars across different `Dtau` values, the Trotter error is negligible and extrapolation is not needed.

### Interaction with Other Parameters

- **Nwrap:** The stabilization interval is `Nwrap × Dtau`. If you halve `Dtau`, you can double `Nwrap` and maintain the same stability. See [[Stabilization Parameters]].
- **Beta:** Reducing `Dtau` at fixed `Beta` doubles `Ltrot`, roughly doubling the cost per sweep. The cost scales as $\mathcal{O}(L_\text{Trot} \times N_\text{dim}^3)$ for dense systems.
- **Checkerboard decomposition:** When `Checkerboard = .T.`, the per-time-slice cost is reduced, making smaller `Dtau` more affordable.

## Model-Specific Notes

- **Hubbard model:** `Dtau = 0.1` is adequate for $U/t \leq 4$. For $U/t = 8$ or larger, consider `Dtau = 0.05`.
- **Kondo model:** The Kondo coupling can require finer discretization. Test convergence with `Dtau` explicitly.
- **Models with continuous fields (HMC):** `Dtau` affects both the Trotter error and the HMC force computation. Smaller `Dtau` generally leads to smoother forces and better HMC acceptance.

## Known Pitfalls

- **Dtau too large → biased results.** Unlike statistical errors, Trotter errors do not average out. Results may look precise (small error bars) but be systematically wrong.
- **Ltrot not an integer.** ALF rounds $\beta / \Delta\tau$ to the nearest integer. Choose `Dtau` and `Beta` such that the ratio is close to an integer to avoid surprises (e.g., the actual `Beta` differs slightly from what you intended).
- **Cost scaling.** Halving `Dtau` doubles `Ltrot` and roughly doubles the wall time. Consider this before going to very small `Dtau` on large lattices.
