# Stabilization Parameters

Numerical stabilization settings in ALF. These apply to **every** simulation regardless of the update scheme.

> **Reference:** The stabilization algorithm is described in the [documentation (PDF)](https://alf.physik.uni-wuerzburg.de/doc.pdf), Section on numerical stabilization.

## Parameters

### Nwrap

Set in the `&VAR_QMC` namelist in the `parameters` file.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `Nwrap` | `10` (in pyALF/examples; `0` internally if not set) | Number of imaginary-time slices between QR stabilizations |

The Green's function is recomputed from scratch after every `Nwrap × Dtau` interval of imaginary time. The total number of stabilization checkpoints is `Ltrot / Nwrap` (rounded up if `Ltrot` is not evenly divisible).

### Stabilization Scheme

Set at compile time via `configure.sh`:

```bash
source configure.sh GNU noMPI STAB3    # or STAB1, STAB2, LOG
```

| Scheme | Flag | Description |
|--------|------|-------------|
| Default | *(none)* | Standard stabilization — generally works well |
| STAB1 | `-DSTAB1` | Older UDV decomposition |
| STAB2 | `-DSTAB2` | UDV with additional normalizations |
| STAB3 | `-DSTAB3` | Newest: separates large and small scales |
| LOG | `-DSTABLOG` | Logarithmic storage of internal scales — extends accessible parameter ranges |

## Guidelines

### Choosing Nwrap

- **Smaller `Nwrap` = more stable but slower.** Each stabilization costs a full Green's function recomputation.
- **Larger `Nwrap` = faster but risks numerical errors.** The propagated Green's function drifts further from the exact value between stabilizations.
- **`Nwrap = 10`** is a good default for moderate interaction strengths and typical `Dtau` values.
- **Reduce `Nwrap`** (e.g., to 5 or even 1) when:
  - Working at strong coupling (large `ham_U`)
  - Using large `Dtau`
  - The `info` file reports poor precision
  - You see the error message "Try with smaller Nwrap or dtau"
- **`Nwrap` must be ≥ 1.** Setting it to 0 is not valid.

### Choosing a Stabilization Scheme

- **Start with the default** (no STAB flag). It is fast and works well for most problems.
- **Use `LOG`** when you need to reach large $\beta$ (low temperature) or strong interactions. It solves NaN issues by storing internal scales in logarithmic form, extending the accessible parameter range at some computational overhead.
- **STAB1 and STAB2** are older schemes kept for backward compatibility. STAB1 uses UDV decompositions; STAB2 adds additional normalizations. They are generally not recommended for new simulations.
- **STAB3** separates large and small scales explicitly. It can be useful for specific problems but is not the default.

### Interaction with Dtau

`Nwrap` and `Dtau` together determine the imaginary-time interval between stabilizations: $\Delta\tau_\text{stab} = \texttt{Nwrap} \times \texttt{Dtau}$. If you reduce `Dtau` (for Trotter error reasons), you can often afford a proportionally larger `Nwrap` while maintaining the same stability.

## Diagnostics

The `info` file reports the **precision of the Green's function** — the maximum deviation between the propagated and freshly computed Green's function at stabilization checkpoints. Values should be small (e.g., $< 10^{-6}$). If the precision is poor:

1. Reduce `Nwrap`
2. Reduce `Dtau`
3. Switch to `LOG` stabilization

If the deviation exceeds 10, ALF prints the error: `'Try with smaller Nwrap or dtau.'`

## Known Pitfalls

- **Nwrap too large for strong coupling.** At large interaction strengths, scales separate rapidly in imaginary time. The default `Nwrap = 10` may be too aggressive.
- **Changing stabilization scheme requires recompilation.** The scheme is set via preprocessor flags, not at runtime.
- **Ltrot not divisible by Nwrap.** This is handled (the last interval is shorter), but can lead to slightly uneven stabilization. Choose `Dtau` and `Beta` such that `Ltrot = Beta/Dtau` is a nice multiple of `Nwrap` when possible.
