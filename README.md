# CUSUM-TBEA code repository

This is the code file for the paper **"CUSUM schemes for simultaneously monitoring correlated time between events and amplitude data with application to the production-critical equipment unit"**.

Time: 2026.10.29


## Code File and Description

### 1. Distribution / copula primitives

**File name: abfrommusd**
**Description:** This MATLAB function converts a prescribed mean and standard deviation into the two-parameter form of the marginal distribution of `X` or `T`. Four families are supported through the switch `dis`: 1 = Gamma, 2 = Lognormal, 3 = Normal, 4 = Weibull (the shape parameter is obtained by `fsolve`, the scale is then `mu/gamma(1/a+1)`). It is used to parameterise the in-control (Phase I) and out-of-control (shifted) marginals in every ARL and control-limit calculation.

**File name: idfTX**
**Description:** This MATLAB function evaluates the inverse CDF (quantile function) of the marginal distribution of `X` or `T`, i.e. it maps a uniform pseudo-observation to the corresponding data value (`gaminv`, `logninv`, `norminv`, `wblinv`). It is the bridge between the copula-generated dependent uniforms and the actual observations.

**File name: claytontheta, francktheta, gumbeltheta**
**Description:** These three MATLAB functions convert Kendall's tau into the dependence parameter theta of the Clayton, Frank and Gumbel copulas, respectively. The Clayton and Gumbel relations are closed-form; the Frank relation is solved numerically because it requires the Debye function.

**File name: debie**
**Description:** This MATLAB function computes the Debye function `D(theta) = (1/theta) * integral_0^theta t/(exp(t)-1) dt` by numerical integration. It is the building block of the Frank copula parameterisation and is called by `francktheta` and `francktau`.

**File name: francktau**
**Description:** This MATLAB function performs the reverse conversion for the Frank copula, i.e. it returns Kendall's tau corresponding to a given Frank theta by inverting `tau = 1 + 4(debie(theta)-1)/theta`.

**File name: tautheta**
**Description:** This MATLAB script tabulates and prints the pre-computed dependence parameters theta of the three copula families for tau = 0.1, 0.2, ..., 0.9. These are the values reported in Table 2 of the paper and are the values hard-coded inside the ARL scripts.

### 2. Statistics and control limits

**File name: pdfZ**
**Description:** This MATLAB function evaluates the probability density of the joint monitoring statistic `Z` at a specified UCL by Monte-Carlo simulation. The arguments allow the copula family (`C`: 8 = Clayton, 9 = Frank, 10 = Gumbel), the statistic (`ZZ`: 5 = `Z1 = X1 - T1`, 6 = `Z2 = X1/T1`, 7 = `Z3 = X1 + 1/T1`), the marginals of `X` and `T`, Kendall's tau and the shifted means. The density is obtained by histogramming 10^7 simulated dependent samples and interpolating at the UCL, and it is the routine used to derive the control limits.

**File name: Ez_calculate**
**Description:** This MATLAB script computes the mathematical expectation `E[Z]` of the monitoring statistic by numerically integrating `z * pdfZ(...)` over the support of `Z`. The comments at the bottom of the file list the switch codes for the copula type, the statistic, the two marginal distributions, the correlation and the shifted means, so that the expectation can be re-evaluated for any scenario.

**File name: Var_zvalue**
**Description:** This MATLAB script computes the variance and the standard deviation of the monitoring statistic by numerically integrating `z^2 * pdfZ(...)` and using `Var(Z) = E[Z^2] - (E[Z])^2`. Together with `Ez_calculate` it provides the moments needed to set the in-control centre line and the control limits.

**File name: optCUSUMExpK_TBEA**
**Description:** This MATLAB function determines the upper control limit of the one-sided CUSUM-TBEA chart by the bisection method: for a given reference value `k` and a target in-control average run length `arl0`, the ARL obtained from the Monte-Carlo routine is evaluated at the midpoint of the bracketing interval `[lb, ub]` and the interval is updated until the ARL matches the target (or after 40 iterations). The function returns the optimal UCL, the corresponding ARL and a convergence flag.

**File name: optCUSUMExpK_TBEA_data**
**Description:** This MATLAB script is the driver of the optimisation above. It sets the design and Phase I parameters used in the paper (`k = 0.15`, `arl0 = 200`, `muX0 = 4946`, `muT0 = 58.9`), calls `optCUSUMExpK_TBEA` and prints the resulting UCL, ARL and flag. It produces the `k+` and `H+` pairs reported in Tables 3 and S1.

### 3. Average run length (ARL) by Monte-Carlo simulation

**Naming convention of the ARL scripts.** The file name is `rlU_cusum_tbea` + copula code + distribution code. The first letter of the copula code is `C` for the Clayton copula and `F` for the Frank copula; the last two letters give the marginals of `X` and `T` in this order (`g` = Gamma, `n` = Normal, `w` = Weibull), e.g. `CSgg` = Clayton copula with Gamma `X` and Gamma `T`, `FSnw` = Frank copula with Normal `X` and Weibull `T`. Inside each script the line that is active (`x = X1 - T1`, `x = X1/T1` or `x = X1 + 1/T1`) selects the statistic `Z1`, `Z2` or `Z3`; the other two options are kept commented out so that any of them can be switched in.

**File name: rlU_cusum_tbeaCSgg, rlU_cusum_tbeaCSgw, rlU_cusum_tbeaCSng, rlU_cusum_tbeaCSnw, rlU_cusum_tbeaCSwg, rlU_cusum_tbeaCSww**
**Description:** These six MATLAB functions calculate the average run length of the one-sided CUSUM-TBEA chart under the **Clayton** copula, for the six combinations of the marginal distributions of `X` and `T` (Gamma-Gamma, Gamma-Weibull, Normal-Gamma, Normal-Weibull, Weibull-Gamma, Weibull-Weibull). Each function draws dependent samples from the copula, updates the CUSUM statistic `C = max(0, C + (x - EZ) - K)` until it exceeds the UCL, and returns the mean run length over the Monte-Carlo replicates. The Kendall's tau, the reference value `k`, the UCL and the shifted means are set in the header of each file; the constants `EZ` are the expectations of the corresponding statistic.

**File name: rlU_cusum_tbeaFSgg, rlU_cusum_tbeaFSgw, rlU_cusum_tbeaFSng, rlU_cusum_tbeaFSnw, rlU_cusum_tbeaFSwg, rlU_cusum_tbeaFSww**
**Description:** These six MATLAB functions calculate the average run length of the one-sided CUSUM-TBEA chart under the **Frank** copula, for the same six combinations of the marginal distributions of `X` and `T`. They are the counterparts of the Clayton versions above and are used to assess the sensitivity of the run-length performance to the dependence model (Table 7 of the paper). In `rlU_cusum_tbeaFSwg` the Kendall's tau is set to the value 0.4657 estimated from the real data.

**File name: rlUewmatbeaCSwg**
**Description:** This MATLAB function calculates the average run length of the one-sided EWMA-TBEA chart by Monte-Carlo simulation under the Clayton copula with Weibull `X` and Gamma `T`, using the recursion `E_t = max(Z0, lambda*Z_t + (1-lambda)*E_{t-1})` with `Z = X1 + 1/T1`. It provides the benchmark run lengths against which the proposed CUSUM-TBEA chart is compared in the ARL comparison tables.

### 4. Design optimisation over the shift scenarios

**File name: minZ1_arl, minZ2_arl, minZ3_arl**
**Description:** These three MATLAB scripts perform the sensitivity analysis of the design for the statistics `Z1`, `Z2` and `Z3`. They contain the tabulated ARL values of all candidate `(k, UCL)` combinations for every pair of shifts `(rho1, rho2)`, then search, for each shift scenario, the combination that minimises the ARL and record the corresponding `(k, UCL)`. The result is written to `min_index_values.xlsx` and displayed as a table, giving the approximately optimal designs and the optimal ARLs reported in Tables 5, 6 and S2-S7.

### 5. Figures

**File name: Scatter_X_T**
**Description:** This MATLAB script generates **Figure 1** of the paper, the scatter plot of the maintenance cost `X` against the time interval `T` between two consecutive breakdowns. The 30 Phase I observations (10/03/2012 - 09/11/2016) are drawn as solid black circles and the 14 Phase II observations (11/01/2017 - 27/12/2018) as open circles; the axes run from 0 to 100 days and 0 to 9000 euros with light grey grid lines, Times New Roman and italic axis labels, and the figure is exported as vector SVG and as a 600 dpi PNG. The script is deliberately written to be backward compatible with MATLAB releases older than R2014b (no `yline`, `readmatrix`, `exportgraphics` or `MarkerEdgeWidth`; `line` is used instead of `plot` so that a locally stored `plot.m` cannot shadow the built-in function).

**File name: PhaseII_Shewhart**
**Description:** This MATLAB script generates the **Shewhart TBEA** panel of **Figure 2** of the paper. It reads the Phase II monitoring series from `REALDATA.xlsx`, applies the in-control centre line and the upper control limit obtained in Phase I, and draws the SCI-style chart (blue line with open circle markers, red dashed UCL labelled on the right, grey centre line, no title, Times New Roman) and exports it as a vector SVG (`Shewhart_Control_Chart3.svg`). The parameters currently in the file are the ones of statistic `Z3` (centre line `mu0 = 2.08818`, UCL `3.26562`).

**File name: PhaseII_CUSUM**
**Description:** This MATLAB script generates the **CUSUM TBEA** panel of **Figure 2**. It reads the Phase II series from `REALDATA.xlsx`, accumulates the one-sided CUSUM statistic `S_t = max(0, S_{t-1} + Z_t - mu0 - k)` with the paper's reference value `k = 0.15` and decision interval `H = 1.20480`, highlights the out-of-control observations, and exports the chart as a vector SVG (`CUSUM_Control_Chart2.svg`). The centre line `mu0 = 1.03278` currently in the file is the Phase I estimate of `E[Z2]`; the commented lines above it hold the corresponding estimates for `Z1` and `Z3`.

**File name: PhaseII_EWMA**
**Description:** This MATLAB script generates the **EWMA TBEA** panel of **Figure 2**. It reads the Phase II series from `REALDATA.xlsx`, computes the one-sided EWMA statistic `E_t = max(Z0, lambda*Z_t + (1-lambda)*E_{t-1})` with the smoothing constant `lambda = 0.7` and decision interval `H = 2.6168`, marks the out-of-control observations and exports the chart as a vector SVG (`EWMA_Control_Chart3.svg`). The parameters currently in the file are the ones of statistic `Z3` (`Z0 = 2.08818`).

### 6. Data files

**File name: REALDATA.xlsx**
**Description:** This Excel workbook holds the Phase II monitoring data of the production-critical equipment unit. `Sheet1` contains, in one column, the series of the monitoring statistic that is fed to the Phase II chart scripts (the file currently holds the 14 values of `Z3 = X1 + 1/T1`); `Sheet2` lists, for each of the 14 Phase II failures, the time interval `T`, the maintenance cost `X`, the normalised values `T/mu_T0` and `X/mu_X0` and the three monitoring statistics `Z1`, `Z2` and `Z3`. When a different statistic is monitored, the corresponding column of `Sheet2` is placed in `Sheet1` and the centre line, control limit and reference value of the script are updated accordingly.

### 7. Real data example - distribution selection and correlation test

**File name: compute_table13_14**
**Description:** This MATLAB function performs the goodness-of-fit assessment of the copula dependence model on the Phase I sample (n = 30). It builds the PIT pseudo-observations `u = rank(T)/(n+1)`, `v = rank(X)/(n+1)`, estimates the dependence parameter of the Frank and Clayton copulas by maximum likelihood, and reports the log-likelihood, AIC, BIC, the Rosenblatt Cramer-von Mises statistic and its parametric-bootstrap p-value (2000 replicates with the parameter re-estimated at every replicate); it further evaluates the lower-tail agreement `lambda(q) = C_n(q,q)/q` together with its simulation-based p-value and the median of `lambda(q)`. These are the results of Tables 13 and 14 of the paper. Only base MATLAB is used, and the bootstrap p-values reproduce the published values up to Monte-Carlo error.

**File name: compute_table15**
**Description:** This MATLAB function performs the goodness-of-fit assessment of the marginal distributions fitted to the Phase I sample: Gamma, Weibull and Normal for `T`, and Gamma, Weibull and Normal for `X`, all by maximum likelihood with the location fixed at 0 for the Gamma and Weibull cases. For every fitted model it reports the K-S, A-D and Cramer-von Mises statistics together with AIC and BIC, and writes the table to `Table15_marginal_GOF.csv`. These are the results of Table 15 of the paper.

**File name: RMI**
**Description:** This MATLAB script evaluates the relative mean inefficiency (RMI) of the Shewhart, EWMA and CUSUM TBEA charts over a grid of mean shifts (`delta_X` from 1.0 to 2.0 in steps of 0.1 and `delta_T` from 1.00 down to 0.50 in steps of 0.05). It starts from the tabulated ARL matrices of the three schemes, forms the pointwise minimum ARL over the three charts at every shift combination, computes the relative inefficiency `(ARL_chart - ARL_best)/ARL_best`, averages it over all 121 combinations to obtain the RMI of each scheme and exports everything to `RMI_Z1_results.xlsx` (sheets: RMI summary, minimum ARL, and the relative inefficiency of each scheme).

**File name: RMI_results**
**Description:** This Excel workbook stores the output of the RMI analysis (`RMI.m`), namely the RMI of the Shewhart, EWMA and CUSUM TBEA charts and the corresponding pointwise minimum ARL matrix.

---

## Folder structure

```
cusum_TBEA/
├── Distribution / copula primitives
│   abfrommusd.m, idfTX.m, claytontheta.m, francktheta.m,
│   gumbeltheta.m, debie.m, francktau.m, tautheta.m
├── Statistics and control limits
│   pdfZ.m, Ez_calculate.m, Var_zvalue.m,
│   optCUSUMExpK_TBEA.m, optCUSUMExpK_TBEA_data.m
├── ARL by Monte-Carlo simulation
│   rlU_cusum_tbeaCS*.m (6 files), rlU_cusum_tbeaFS*.m (6 files),
│   rlUewmatbeaCSwg.m
├── Design optimisation
│   minZ1_arl.m, minZ2_arl.m, minZ3_arl.m
├── Figures
│   Scatter_X_T.m, PhaseII_Shewhart.m, PhaseII_CUSUM.m, PhaseII_EWMA.m
├── REALDATA.xlsx
└── real data example/
    ├── Distribution selection and correlation test/
    │   compute_table13_14.m, compute_table15.m
    └── RMI/
        ├── RMI.m
        └── RMI_results.xlsx
```

## How to run

1. Add the repository folder (or the relevant sub-folder) to the MATLAB path, or open it as the current folder.
2. **Control limits:** run `optCUSUMExpK_TBEA_data.m` to obtain the UCL of the CUSUM-TBEA chart for the target ARL.
3. **Run lengths:** run any `rlU_cusum_tbeaCS*.m` / `rlU_cusum_tbeaFS*.m` file with the shift levels and the control limit of interest.
4. **Optimal designs:** run `minZ1_arl.m` (or `minZ2_arl.m`, `minZ3_arl.m`) to obtain the recommended `(k, UCL)` for each shift scenario.
5. **Figures:** run `Scatter_X_T.m` for Figure 1 and the three `PhaseII_*.m` scripts for Figure 2; the SVGs are written next to the scripts.
6. **Real data example:** run `compute_table13_14.m`, `compute_table15.m` and `RMI/RMI.m` for the tables and the RMI comparison.

## Software requirements

MATLAB R2019b or later is recommended. The Monte-Carlo and optimisation scripts, as well as the two goodness-of-fit scripts, use only base MATLAB (the Gamma, Weibull and Normal densities and CDFs are written out explicitly, so no Statistics and Machine Learning Toolbox is required). `Scatter_X_T.m` is backward compatible with releases older than R2014b, while `PhaseII_Shewhart.m`, `PhaseII_CUSUM.m` and `PhaseII_EWMA.m` require R2016b or later because they call `yline`, `readmatrix` and `exportgraphics`.

## Notes

* `rlU_cusum_tbeaCSnw.asv` is an automatic MATLAB backup file and is not part of the code; it is excluded from the repository through `.gitignore`.
* In each ARL script the active statistic (`Z1`, `Z2` or `Z3`) is selected by the single uncommented line `x = X1 ...`; the two remaining alternatives are kept in the file for convenience.
* The Monte-Carlo results depend on the random seed; the published values were obtained with the settings stored in the files.
