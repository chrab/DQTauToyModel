# Copilot Instructions

## Project Overview

A Jupyter notebook toy model for time-dependent water abundance in a protoplanetary disk warm layer, driven by a time-varying UV field. Motivated by JWST observations of DQ Tau-like binary systems. The notebook was originally written by Claude AI and is intended for interactive exploration.

## Environment

Managed with conda. The environment name is `dqtaubinder`.

```bash
conda env create -f environment.yml
conda activate dqtaubinder
jupyter notebook water_ode.ipynb
```

The project is also runnable on Binder without local setup.

## Architecture

All code lives in a single notebook: **`water_ode.ipynb`**. The notebook is structured as numbered sections:

1. **Reaction rates** — `k_photodiss(G)` and `k_formation(T, nH)` compute photodissociation and formation rates.
2. **UV field prescriptions** — `uv_field(t_days, mode, ...)` supports `'sine'`, `'step'`, and `'periodic_burst'` modes.
3. **ODE solver** — `solve_water_ode(...)` integrates the two-level ODE using **explicit Euler** (not `scipy.integrate`).
4. **Timescale diagnostics** — `print_timescales(...)` for quick sanity checks.
5–7. **Static runs, plots, and parameter grids** — publication-quality figures and response-amplitude surveys over density/temperature space.
8. **Observed DQ Tau driver** — `solve_water_ode_observed(...)` drives the ODE with a phase-folded accretion luminosity template interpolated from `laccr.dat`.
9. **Interactive explorer** — `ipywidgets` sliders for live parameter exploration.

## Key Physical Parameters (DQ Tau defaults)

| Parameter | Value | Meaning |
|-----------|-------|---------|
| `nH` | `1e11` cm⁻³ | Warm molecular layer density |
| `T` | `300` K | Temperature |
| `G0` | `1e2` | Quiescent UV field (Draine units) |
| `G_burst` | `10` | Burst enhancement factor |
| `period_days` | `15.8` | DQ Tau orbital period |
| `eps_O` | `2.4e-4` | Available oxygen abundance (n_O/n_H) |
| `k0` | `1e-9` s⁻¹ | Photodissociation rate per Draine unit |

`eps_O = 2.4e-4` accounts for solar oxygen (`4.9e-4`) minus refractory silicate lock-up and CO; consistent with ProDiMo disk chemistry models.

## Data File

**`laccr.dat`** — 71 rows of phase-folded accretion data from Tofflemire+ 2025 (ApJ 985, 224). Columns: orbital phase, accretion luminosity L_acc [L_sun]. Phase 1 corresponds to the first burst.

## ODE Formulation

```
dy/dt = k_form(T, nH) * (eps_O - y) - k_pd(G(t)) * y
```

where `y = n(H2O)/n_H`. The integrator is deliberate explicit Euler with a fixed `dt_days=0.05` step — keep this consistent when adding new solver variants.

## Conventions

- UV field `G` is always in **Draine (1978)** units.
- Time is always in **days** at the interface; rates are converted to s⁻¹ internally (`1 day = 86400 s`).
- Abundance `x` or `y` is the fractional water abundance `n(H2O)/n_H`.
- Solver functions return `(t_days, x, G_at_t)` tuples.
- Amplitude scatter in `solve_water_ode_observed` is applied in **dex** (log10 space) to the UV field.
