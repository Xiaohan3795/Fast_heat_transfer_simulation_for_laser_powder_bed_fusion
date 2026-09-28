# Fast Heat Transfer Simulation for Laser Powder Bed Fusion

[![DOI](https://zenodo.org/badge/604094625.svg)](https://zenodo.org/badge/latestdoi/604094625)

This folder contains MATLAB code for transient heat-transfer simulation in laser powder bed fusion (LPBF), including:

- a full-order model (`FOM`) based on finite-element-style operators, and
- a fast surrogate model (`SURROGATE`) that uses adaptive projection bases and randomized sketching.

The main script runs both models, compares temperature histories, and reports accuracy and time reduction.

## Folder Contents

### Main workflow

- `simulator.m`  
  Main entry point. Loads case data, runs both FOM and surrogate, performs Picard iterations, and reports:
  - relative temperature error (`re_2norm`)
  - computational time reduction (`time_reduction`)

- `parameter_setup.m`  
  Central place for all simulation parameters (time step, source power, basis size, sketching sizes, tolerances, etc.).

- `simulation_prepare.m`  
  Precomputes boundary/node sets and constant matrices used throughout the solve:
  - convection matrix/vector
  - quadrature-evaluated basis operators for mass/stiffness/radiation terms

- `load_vector.m`  
  Builds the moving laser trajectory and load vectors over all time steps.

### Surrogate-model basis and sketching

- `make_gauss_orth.m`  
  Builds adaptive Gaussian trial functions and orthogonalizes them into a reduced basis.

- `pre_proj_sketch_new.m`  
  Performs randomized row selection and projects operators/vector quantities to reduced coordinates.

- `sketch_index_noproj.m`  
  Computes leverage-style sampling probabilities and returns selected indices/weights.

- `rsvd_subGaussian.m`, `rsvd_subGaussian_sketch.m`  
  Randomized SVD utilities used for basis generation and sketch-related subspaces.

### Projected operator assembly

- `proj_sketch_M.m`  
  Approximate reduced mass term.

- `proj_sketch_K.m`  
  Approximate reduced stiffness term.

- `proj_sketch_R.m`  
  Approximate reduced radiation-loss term.

### Material and constitutive helpers

- `D_matrices.m`  
  Builds diagonal temperature-dependent matrices for radiation, conductivity, and capacity terms.

- `kappa_find.m`  
  Thermal conductivity law (currently a placeholder polynomial).

- `rhoc_find.m`  
  Density and heat-capacity laws (currently placeholder linear forms).

### Quadrature and source assembly

- `Gaussian_quadrature_phi.m`  
  Volume quadrature evaluation of basis functions.

- `Gaussian_quadrature_phi_srf.m`  
  Surface quadrature evaluation of basis functions.

- `make_surface_integrals_Gaussian.m`  
  Assembles convection/radiation boundary integral terms.

- `make_surface_source.m`  
  Assembles moving Gaussian surface heat-source load vector.

## Required Input Data

`simulator.m` starts with:

- `load simple_case`

So a `simple_case.mat` file must be available on the MATLAB path (typically in this folder).  
The loaded data is expected to provide a mesh/operator struct named `msh` used across the code, with fields such as:

- geometry/topology: `vtx`, `simp`, `srf`, `csrf`, `srfarea`
- FEM operators: `phi_grad`, `phi_cons`, `Bgrad`, `bcons`
- quadrature containers: `Gq`, `Gqsrf`

If your case file uses a different variable name, either rename it to `msh` before running or update `simulator.m`.

## How to Run

1. Open MATLAB and set the current folder to this directory.
2. Ensure `simple_case.mat` is present and contains `msh`.
3. Run:

```matlab
simulator
```

## Key Parameters to Tune

Edit `parameter_setup.m` to control behavior:

- `dt`, `tn`: temporal resolution and number of time steps
- `fm`, `v`, `d2`: laser heat source strength, scan speed, beam width
- `dim_Psi`, `n_mu`, `n_u`, `n_t`: reduced basis and surrogate activation controls
- `skm`, `skk`, `sks`, `sp`: sketching sample sizes and sampling ratio
- `e`, `run_max`: Picard tolerance and iteration cap

## Outputs

After execution, `simulator.m` reports:

- `re_2norm`: relative L2 error (%) between FOM and surrogate temperature fields
- `time_reduction`: estimated percentage speed-up from surrogate modeling

It also stores full temperature histories in memory:

- `temp_FOM`
- `temp_S`

## Notes

- Material laws in `kappa_find.m` and `rhoc_find.m` are simplified placeholders.  
  Replace them with calibrated LPBF material models for physically realistic predictions.
- The scripts use `clear all; clc;` in `simulator.m`; remove those lines if you prefer preserving the workspace.

## Citation

This repository contains the MATLAB implementation accompanying the paper:

Li, X., and Polydorides, N. (2023). *Fast heat transfer simulation for laser powder bed fusion*. *Computer Methods in Applied Mechanics and Engineering*, 412, 116107. https://doi.org/10.1016/j.cma.2023.116107

If you use this code in your research, please cite the paper above.
