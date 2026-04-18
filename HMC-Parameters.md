# HMC Parameters

Tuning the Hybrid Monte Carlo updating scheme in ALF.

## Parameters

| Parameter | Where Set | Typical Range | Description |
|-----------|-----------|---------------|-------------|
| `Nstp` | _TODO_ | 5–20 | Number of leap-frog integration steps |
| `dtHMC` | _TODO_ | 0.05–0.2 | Leap-frog step size |
| `Apply_B` (mass matrix) | _TODO_ | — | Preconditioning via mass matrix |

## Guidelines

_TODO: How parameters interact (Nstp × dtHMC ≈ trajectory length), target acceptance rate (~70–80%), how to diagnose issues._

## Model-Specific Notes

_TODO: Parameter choices that work well for specific models (Hubbard, Kondo, etc.)._

## Known Pitfalls

_TODO: Common mistakes and their symptoms._
