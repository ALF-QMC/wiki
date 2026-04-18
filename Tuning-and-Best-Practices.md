# Tuning and Best Practices

Practical advice on choosing simulation parameters for reliable and efficient ALF runs.

## Philosophy

This section collects hard-won experience on what parameter choices work well and what to avoid. Unlike the [formal documentation](https://alf.physik.uni-wuerzburg.de/doc.pdf), which derives the algorithms, these pages focus on **practical guidance**: what to set, what to look for, and what can go wrong.

## Topics

- [[HMC Parameters]] — Hybrid Monte Carlo: step size, leap-frog steps, mass matrix (`Apply_B`)
- [[Stabilization Parameters]] — `Nwrap` and numerical stabilization frequency
- [[Discretization]] — `Dtau` choices and Trotter error tradeoffs
- [[Tempering]] — Parallel tempering: replica count, temperature grid, exchange rates

## General Advice

_TODO: Cross-cutting advice that applies regardless of the specific algorithm or model._
