# What is BESS Sizing Simulator

A project designed to provide computational and theoretical insights into battery sizing for ramp-rate control in Variable Renewable Energy (VRE) plants. It is based on the statistical analysis of power-output signals from wind (eolic) generation and power-demand time series.

## Objective

The main objective of this project is to determine the cost-effective sizing of Battery Energy Storage Systems (BESS) for ramp-rate control, ensuring adequate frequency support in VRE generation.  
We introduce a novel statistical approach to BESS sizing: using power-output time series to construct a probabilistic model of the BESS response, in terms of its probability density function (PDF) and cumulative distribution function (CDF).  
With this model, the required BESS power and energy capacity can be determined using the 99th percentile of the BESS-response CDF as the design criterion.

# Usage

The project currently consists of three fundamental components:  
1. Synthetic power-time-series characterization and generation  
2. BESS response simulation  
3. Numerical and analytical solutions for the BESS PDF based on the statistical properties of the power signal and the critical slope that triggers the battery response  

## Main Parameters

These are the user-defined parameters required for all functions and solvers to operate correctly:

- `N`: Number of timesteps for the synthetic time-series generator (recommended: >1,000,000).  
- `grid`: Resolution of the PDF solvers.  
- `Pmax`: Maximum power capacity of the plant.  
- `r`: Fraction of **Pmax** used to define the critical slope **a**, calculated as **a = r Pmax**. Default is 1% of the maximum capacity.  
- `beta`: Decay rate of the exponential tail of the Laplace distribution.

When using the Generalized Laplace (GL) distribution, the following parameters are also required:

- `c`: Weight ratio of the short-tail component in the GL distribution.  
- `zeta`: Scaling factor for the equivalent decay rate **beta_GL** of the short-tail term.

## VRE Synthetic Data

The increments of wind-power generation (Y) are known to follow a *Generalized Laplace* distribution.  
The functions `simulate_Y_P_SL` and `simulate_Y_P_GL` implement inverse-sampling methods to generate time series representing the increments of truncated Simple-Laplace and Generalized-Laplace distributions (Y), along with their cumulative paths (P).  
Currently, these functions only reproduce time series with the same marginal distribution.  
The synthetic-data generators are located in the `core_SynData.py` script.

## BESS PDF Solvers

Three solvers are available for computing the BESS PDF:

1. Algorithmic solution (empirical distribution)  
2. Numerical solution (Nystrom method)  
3. Analytical solution (Neumann-series expansion)

All solvers are located in the `core_Solvers.py` script under the names  
`algorithmic_solution()`, `numerical_solution()`, and `analytical_solution()`.

Each solver returns:

- **b_grid**: Array of BESS power values in \[0, b_max\].  
- **g(b)**: Array of the normalized BESS PDF.  
- **p0**: Point mass at zero (idle battery probability).  
- **p99(b)**: 99th percentile of the BESS-response CDF, representing the required BESS capacity.

The `algorithmic_solution()` additionally returns the simulated BESS response based on synthetic or empirical power-output time series.

## JIT Optimization

The project uses `numba` decorators for most functions due to the need for extremely long power time series in order to explore the full state space of increments.  
When modifying a function decorated with `@njit`, ensure it uses *no-Python* syntax; otherwise, numba will fall back or fail.  
The code also runs without numba—simply remove the decorators if needed.

The `core_helpers.py` script contains several numba-friendly utility functions that can be replaced by standard `numpy` equivalents when not using the JIT environment.
