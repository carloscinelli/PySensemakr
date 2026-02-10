---
title: '`PySensemakr`: Sensitivity Analysis Tools for OLS in Python'
tags:
  - Python
  - causal inference
  - sensitivity analysis
  - omitted variable bias
  - regression
authors:
  - name: Zhehao Zhang
    affiliation: 1
  - name: Nathan LaPierre
    affiliation: 2
  - name: Brian Hill
    affiliation: 2
  - name: Carlos Cinelli
    orcid: 0000-0002-2021-7739
    corresponding: true
    affiliation: 1
affiliations:
 - name: University of Washington, United States
   index: 1
 - name: University of California, Los Angeles, United States
   index: 2
date: 10 February 2026
bibliography: paper.bib
---

# Summary

`PySensemakr` is a Python package that implements a suite of sensitivity analysis tools
that extends the traditional omitted variable bias (OVB) framework for linear regression models,
as developed in @cinelli2020making. The package allows researchers to
(i) compute sensitivity statistics for routine reporting, such as the partial $R^2$
of the treatment with the outcome and the robustness value,
(ii) derive bounds on the strength of unobserved confounders using observed covariates as benchmarks,
and (iii) produce sensitivity contour plots and extreme scenario plots that provide a
visual summary of the sensitivity of causal estimates to unobserved confounding.
`PySensemakr` is built on top of `statsmodels` [@seabold2010statsmodels] and requires
only standard regression output, without further assumptions on the functional form of
the treatment assignment mechanism or on the distribution of the unobserved confounders.
The goal of `PySensemakr` is to make sensitivity analysis an accessible,
routine part of empirical research in Python, thereby enabling a more transparent
and disciplined discussion regarding the causal interpretation of regression estimates.

# Statement of Need

The most common strategy for drawing causal inferences from observational data is to
adjust for observed covariates in a regression model, while making the untestable assumption
that there are no unobserved confounders. Since this assumption cannot be verified with
the data at hand, sensitivity analysis---which asks how strong unobserved confounders
would need to be in order to change the conclusions of a study---plays a central role
in determining the credibility of causal claims based on observational data.

@cinelli2020making proposes a modern suite of sensitivity analysis tools based on a
partial $R^2$ parameterization of omitted variable bias. These tools
(i) do not require assumptions on the functional form of the treatment assignment
mechanism nor on the distribution of the unobserved confounders,
(ii) naturally handle multiple confounders, possibly acting non-linearly,
(iii) exploit expert knowledge to bound sensitivity parameters using observed covariates
as benchmarks, and (iv) can be easily computed using only standard regression output.
The methodology thus lends itself naturally to *routine reporting*---the practice of
including sensitivity statistics alongside standard regression tables, much like one
would report $p$-values or confidence intervals.

While `sensemakr` implementations already exist for R [@cinelli2020sensemakr] and Stata, Python
has lacked an equivalent tool, despite being one of the most widely used languages for data
analysis across the social sciences, biomedical research, and industry.
`PySensemakr` fills this gap, bringing the full `sensemakr` methodology to the Python ecosystem.

# State of the Field

The R version of `sensemakr` [@cinelli2020sensemakr] has been widely adopted across
disciplines---including political science, economics, epidemiology, and
education---and has been cited in thousands of empirical studies. A Stata version
is also available. `PySensemakr` is a dedicated, full-featured Python implementation
of the complete `sensemakr` methodology for OLS regression. It provides the full suite
of sensitivity statistics (partial $R^2$, robustness values $RV_q$ and $RV_{q,\alpha}$),
the complete benchmarking apparatus, all bias-adjustment functions, and all
visualization tools (contour plots and extreme scenario plots), with an API that
closely mirrors the R version. `PySensemakr` brings this same dedicated toolkit to the
Python ecosystem, lowering the barrier for researchers who wish to incorporate formal
sensitivity analysis into Python-based regression workflows.

# Software Design

`PySensemakr` takes a fitted `statsmodels` `OLSResults` object as input and computes
all relevant sensitivity statistics from the standard regression output. The package
is organized into the following modules:

- **`main`**: The `Sensemakr` class serves as the primary interface. It takes a fitted
  model, a treatment variable, and optional benchmark covariates, and computes sensitivity
  statistics and bounds. It provides `summary()` and `plot()` methods for reporting results.
- **`sensitivity_statistics`**: Functions for computing the partial $R^2$ of the treatment
  with the outcome, the robustness value $RV_q$ (the minimum confounding strength required
  to reduce the estimate by a fraction $q$), and the robustness value $RV_{q,\alpha}$
  (accounting for statistical significance).
- **`sensitivity_bounds`**: Functions for computing bounds on the bias-adjusted estimates,
  using observed covariates as benchmarks for the plausible strength of unobserved confounders.
- **`sensitivity_plots`**: Functions for producing contour plots of bias-adjusted estimates
  and $t$-values as functions of the sensitivity parameters $R^2_{D \sim Z | \mathbf{X}}$
  and $R^2_{Y \sim Z | \mathbf{X}, D}$, as well as extreme scenario plots.
- **`bias_functions`**: Core functions for computing the bias, bias-adjusted estimates,
  standard errors, and $t$-values.

\autoref{fig:contour} illustrates the contour plot output for the example dataset of
@hazlett2020angry, in which the estimate and the $t$-value of the treatment effect are
displayed as functions of the hypothetical confounding strength. \autoref{fig:extreme}
shows the corresponding extreme scenario plot.

![Sensitivity contour plots for the Darfur data. The left panel shows contour lines for the bias-adjusted estimate of the effect of being directly harmed on attitudes toward peace, as a function of the partial $R^2$ of unobserved confounding with the treatment and the outcome. The right panel shows the corresponding contour lines for the $t$-value. The red diamonds indicate the bounds based on the observed covariate `female`.\label{fig:contour}](output_4_0.png){width=50%} ![](output_5_0.png){width=50%}

![Extreme scenario plot for the Darfur data, showing the bias-adjusted estimate under the worst-case scenario for confounders of varying strength.\label{fig:extreme}](output_6_0.png){width=70%}

The following code illustrates a typical workflow:

```python
import sensemakr as smkr
import statsmodels.formula.api as smf

# Load data and fit regression model
darfur = smkr.load_darfur()
model = smf.ols(formula='peacefactor ~ directlyharmed + age + farmer_dar + '
                'herder_dar + pastvoted + hhsize_darfur + female + village',
                data=darfur).fit()

# Run sensitivity analysis
sense = smkr.Sensemakr(model=model,
                       treatment="directlyharmed",
                       benchmark_covariates=["female"],
                       kd=[1, 2, 3])

# Print and plot results
sense.summary()
sense.plot()
```

# Research Impact

The sensitivity analysis methodology implemented in `PySensemakr` [@cinelli2020making] has
had broad impact across the empirical sciences. The original theoretical paper has been
cited over 1,800 times, and the R implementation of `sensemakr` [@cinelli2020sensemakr]
has been used in published studies spanning political science, economics, epidemiology,
sociology, education, and environmental science, among other fields.
By providing a Python implementation, `PySensemakr` extends the reach of these
tools to the large and growing community of researchers and data scientists who
use Python as their primary computing environment, thereby helping to make formal
sensitivity analysis a routine component of empirical research workflows.

# AI Usage Disclosure

Generative AI tools were used to assist with code maintenance tasks (e.g., updating deprecated
API calls for compatibility with newer versions of dependencies). All AI-generated changes
were reviewed and tested by the authors.

# Acknowledgements

We thank Chad Hazlett and Jeremy Ferwerda for their contributions to the theoretical
development and to the R and Stata implementations of `sensemakr`, which served as the
foundation for this Python package.

# References
