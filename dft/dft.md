---
layout: page
title: DFT Notes
subtitle: Working notes on density functional theory and its practical knobs
permalink: /dft/
math: true
---

This page exists partly to test that math, code, tables and lists all render
correctly through the site's page layout. Everything below is real, so it can
stay if it's useful.

## The Hohenberg–Kohn idea

The ground-state electron density $$n(\mathbf{r})$$ determines the external
potential up to a constant, and therefore determines everything else about the
system. That replaces a wavefunction in $$3N$$ variables with a scalar field in
three, which is what makes first-principles calculations on real solids
tractable at all.

The energy is a functional of the density,

$$
\begin{equation}
E[n] = T_s[n] + \int v_{\text{ext}}(\mathbf{r})\, n(\mathbf{r})\, d^3r
     + E_{\text{H}}[n] + E_{xc}[n],
\end{equation}
$$

and it is minimized by the true ground-state density.

## The Kohn–Sham construction

Rather than approximate the kinetic energy of the interacting system directly,
Kohn and Sham map it onto a fictitious system of non-interacting electrons that
reproduces the same density:

$$
\begin{equation}
\left[ -\tfrac{1}{2}\nabla^2 + v_{\text{eff}}(\mathbf{r}) \right]
  \psi_i(\mathbf{r}) = \varepsilon_i\, \psi_i(\mathbf{r}),
\end{equation}
$$

with the effective potential

$$
v_{\text{eff}}(\mathbf{r}) = v_{\text{ext}}(\mathbf{r})
  + \int \frac{n(\mathbf{r}')}{|\mathbf{r} - \mathbf{r}'|} d^3r'
  + v_{xc}(\mathbf{r}),
\qquad
v_{xc}(\mathbf{r}) = \frac{\delta E_{xc}[n]}{\delta n(\mathbf{r})}.
$$

Because $$v_{\text{eff}}$$ depends on the density it produces, the equations are
solved self-consistently.

## Where the approximation lives

Everything hard is pushed into $$E_{xc}[n]$$, which is not known exactly. The
common choices trade cost against which errors they make:

| Rung | Depends on | Typical failure |
|---|---|---|
| LDA | $$n$$ | Overbinds; underestimates lattice constants |
| GGA (PBE) | $$n, \nabla n$$ | Underbinds; overestimates volumes |
| meta-GGA (SCAN) | $$n, \nabla n, \tau$$ | Costlier; numerically sensitive |
| Hybrid (HSE) | + exact exchange | Expensive for large cells |

For equation-of-state work at high pressure, the LDA/GGA spread is a useful
informal error bar: the two tend to bracket experiment.

## Practical convergence knobs

- **Plane-wave cutoff** $$E_{\text{cut}}$$ — the basis includes all
  $$\mathbf{G}$$ with $$\tfrac{1}{2}|\mathbf{k}+\mathbf{G}|^2 < E_{\text{cut}}$$.
  Converge total energy *differences*, not absolute energies.
- **k-point sampling** — denser meshes matter far more for metals than for
  insulators, where the Brillouin-zone integrand is smooth.
- **Smearing** — Methfessel–Paxton for metals, tetrahedron for accurate
  densities of states, Gaussian with small width for insulators.

A minimal VASP `INCAR` for a static run:

```
ENCUT  = 600
PREC   = Accurate
EDIFF  = 1E-8
ISMEAR = -5
LREAL  = .FALSE.
```

The tight `EDIFF` matters more than it looks: stress and elastic constants are
derivatives of the energy, so they inherit and amplify any sloppiness in the
self-consistency loop.
