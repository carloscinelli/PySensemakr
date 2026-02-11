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
for linear regression models based on the omitted variable bias (OVB) framework developed
in @cinelli2020making. In observational studies, researchers typically adjust for observed
covariates in a regression model and assume that no unobserved confounders remain. Since
this assumption is untestable, sensitivity analysis asks how strong unobserved confounders
would need to be to alter the conclusions of a study. `PySensemakr` provides tools that
make this question precise and easy to answer.

Specifically, the package allows researchers to (i) compute sensitivity statistics for
routine reporting, including the partial $R^2$ of the treatment with the outcome and the
robustness value ($RV_q$), which measures the minimum confounding strength---in terms of
the proportion of residual variance explained---required to change a study's conclusions;
(ii) derive bounds on the strength of unobserved confounders using observed covariates as
benchmarks, asking, for example, how strong a confounder would need to be relative to an
observed covariate such as gender or income to overturn a result; and (iii) produce contour
plots and extreme scenario plots that provide a visual summary of the sensitivity of causal
estimates to unobserved confounding. `PySensemakr` is built on top of `statsmodels`
[@seabold2010statsmodels] and requires only standard regression output, without assumptions
on the functional form of the treatment assignment mechanism or on the distribution of the
unobserved confounders.

# Statement of Need

The most widely used strategy for drawing causal inferences from observational data is to
adjust for observed covariates in a regression model. The key untestable assumption underlying
this approach is that there are no unobserved confounders---variables that jointly affect both
the treatment and the outcome and that are not included in the model. Since this assumption
cannot be verified from the data at hand, the credibility of causal claims rests on how
robust they are to potential violations. Sensitivity analysis formalizes this question:
rather than assuming confounders are absent, it quantifies how strong they would need to be
to change the conclusions of a study.

@cinelli2020making proposes a modern suite of sensitivity analysis tools based on a
partial $R^2$ parameterization of omitted variable bias. The key idea is to describe the
strength of unobserved confounders in terms of the partial $R^2$---the share of residual
variance they explain---of the confounder with the treatment and the outcome, after
accounting for observed covariates. This parameterization has several advantages:
(i) it does not require assumptions on the functional form of the treatment assignment
mechanism nor on the distribution of the unobserved confounders,
(ii) it naturally handles multiple confounders, possibly acting non-linearly,
(iii) it exploits expert knowledge to bound sensitivity parameters using observed covariates
as benchmarks, and (iv) it can be easily computed using only standard regression output.
The methodology lends itself naturally to *routine reporting*---the practice of including
sensitivity statistics alongside standard regression tables, much like one reports
$p$-values or confidence intervals---because it produces a small number of interpretable
summary measures that can be reported in any empirical paper.

While `sensemakr` implementations already exist for R [@cinelli2020sensemakr] and Stata, Python
has lacked an equivalent tool, despite being one of the most widely used languages for data
analysis across the social sciences, biomedical research, and industry.
`PySensemakr` fills this gap, bringing the full `sensemakr` methodology to the Python
ecosystem. Its target audience includes applied researchers in the social sciences,
epidemiology, and any field where OLS regression is used to estimate causal effects from
observational data, as well as data scientists in industry who need to assess the robustness
of their findings.

# State of the Field

The R version of `sensemakr` [@cinelli2020sensemakr] has been widely adopted across
disciplines---including political science, economics, epidemiology, and
education---and has been cited in thousands of empirical studies. A Stata version
is also available. `PySensemakr` is a dedicated, full-featured Python implementation
of the complete `sensemakr` methodology for OLS regression. It provides the full suite
of sensitivity statistics (partial $R^2$, robustness values $RV_q$ and $RV_{q,\alpha}$),
the complete benchmarking apparatus, all bias-adjustment functions, and all
visualization tools (contour plots and extreme scenario plots), with an API that
closely mirrors the R version.

Other Python packages provide related but distinct functionality. For instance,
general-purpose causal inference libraries may include basic sensitivity checks, but none
implements the complete partial $R^2$-based framework of @cinelli2020making, which includes
sensitivity statistics, formal benchmarking bounds, bias-adjusted inference, and specialized
visualizations as a unified toolkit. We chose to build a standalone package rather than
contribute to an existing library because the methodology requires a tightly integrated set
of functions---from computing sensitivity statistics, to bounding the bias using benchmark
covariates, to producing specialized diagnostic plots---that would not fit naturally as
an extension of a general-purpose regression or plotting library. The standalone design also
allows `PySensemakr` to maintain API parity with the R version, easing adoption for
researchers who work across both languages.

# Software Design

`PySensemakr` takes a fitted `statsmodels` `OLSResults` object as input and computes
all relevant sensitivity statistics from the standard regression output. A key design
decision was to build on `statsmodels` rather than requiring users to re-specify their
models, allowing `PySensemakr` to integrate seamlessly into existing regression
workflows. The package is organized into the following modules:

- **`main`**: The `Sensemakr` class serves as the primary interface. It takes a fitted
  model, a treatment variable, and optional benchmark covariates, and computes sensitivity
  statistics and bounds. It provides `summary()` and `plot()` methods for reporting results.
- **`sensitivity_statistics`**: Functions for computing the partial $R^2$ of the treatment
  with the outcome, the robustness value $RV_q$ (the minimum confounding strength required
  to reduce the estimate by a fraction $q$), and the robustness value $RV_{q,\alpha}$
  (which additionally accounts for statistical significance).
- **`sensitivity_bounds`**: Functions for computing bounds on the bias-adjusted estimates,
  using observed covariates as benchmarks for the plausible strength of unobserved confounders.
- **`sensitivity_plots`**: Functions for producing contour plots of bias-adjusted estimates
  and $t$-values as functions of the sensitivity parameters $R^2_{D \sim Z | \mathbf{X}}$
  and $R^2_{Y \sim Z | \mathbf{X}, D}$, as well as extreme scenario plots.
- **`bias_functions`**: Core functions for computing the bias, bias-adjusted estimates,
  standard errors, and $t$-values.

This modular design mirrors the R version's architecture, making it straightforward for
users familiar with the R package to transition to Python. Internally, all computations rely
on closed-form expressions derived from the OVB framework, avoiding simulation or resampling,
which makes the package fast and deterministic. All numerical results have been validated
against the R implementation to ensure correctness.

## Example: Sensitivity Analysis for the Darfur Data

We illustrate a typical workflow using the dataset of @hazlett2020angry, who studies the
effect of being directly exposed to violence (`directlyharmed`) on attitudes toward peace
(`peacefactor`) among Darfurian refugees, controlling for covariates such as age, gender,
occupation, past political participation, and household size. The analysis proceeds in
three steps: fit an OLS regression, construct a `Sensemakr` object specifying the treatment
and benchmark covariates, and then call `summary()` and `plot()` to obtain all results.

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

The unadjusted OLS estimate of the treatment effect is $0.097$ ($SE = 0.023$, $t = 4.18$).
The `summary()` method reports the key sensitivity statistics. The partial $R^2$ of the
treatment with the outcome is $2.2\%$, meaning that an extreme confounder explaining $100\%$
of the residual outcome variance would need to explain at least $2.2\%$ of the residual
treatment variance to fully account for the observed effect. The robustness value is
$RV_q = 13.9\%$: unobserved confounders that explain less than $13.9\%$ of the residual
variance of both the treatment and the outcome are not strong enough to reduce the point
estimate to zero. The robustness value accounting for statistical significance is
$RV_{q,\alpha} = 7.6\%$: confounders explaining less than $7.6\%$ of the residual variance
of both the treatment and the outcome cannot bring the estimate to a range where it is
no longer statistically distinguishable from zero at the $5\%$ level.

The `summary()` method also reports bounds on the bias-adjusted estimates using observed
covariates as benchmarks for confounding strength. Using `female` as a benchmark, if an
unobserved confounder were as strongly associated with the treatment and outcome as `female`
is ($1\times$ `female`), the bias-adjusted estimate would be $0.075$ ($t = 3.44$, $95\%$
CI: $[0.032, 0.118]$). Even at $2\times$ `female`, the adjusted estimate remains positive
and significant ($0.053$, $t = 2.60$). At $3\times$ `female`, the adjusted confidence
interval begins to include zero ($0.030$, $95\%$ CI: $[-0.006, 0.067]$), indicating the
level of confounding needed to overturn the finding. These benchmarking results allow
researchers to make calibrated judgments: rather than asking whether confounding exists in
the abstract, they can assess whether confounders plausibly as strong as key observed
covariates are likely to be present in their application.

\autoref{fig:contour} shows the contour plots produced by `sense.plot()`. The left panel
displays contour lines of the bias-adjusted estimate as a function of the sensitivity
parameters $R^2_{D \sim Z | \mathbf{X}}$ and $R^2_{Y \sim Z | \mathbf{X}, D}$; the right
panel shows the corresponding $t$-value contours. The red diamonds mark the benchmark bounds
based on `female`. \autoref{fig:extreme} shows the extreme scenario plot, which displays the
bias-adjusted estimate under the worst-case confounding scenario as a function of the
confounder's association with the treatment, for different assumptions about its association
with the outcome ($R^2_{Y \sim Z | \mathbf{X}, D}$ at $100\%$, $75\%$, and $50\%$). Together,
these plots allow researchers to communicate the sensitivity of their findings in a single,
interpretable figure.

![Sensitivity contour plots for the Darfur data. The left panel shows contour lines for the bias-adjusted estimate of the effect of being directly harmed on attitudes toward peace, as a function of the partial $R^2$ of unobserved confounding with the treatment and the outcome. The right panel shows the corresponding contour lines for the $t$-value. The red diamonds indicate the bounds based on the observed covariate `female`.\label{fig:contour}](output_4_0.png){width=50%} ![](output_5_0.png){width=50%}

![Extreme scenario plot for the Darfur data, showing the bias-adjusted estimate under the worst-case scenario for confounders of varying strength.\label{fig:extreme}](output_6_0.png){width=70%}

# Research Impact

The sensitivity analysis methodology implemented in `PySensemakr` [@cinelli2020making] has
had broad impact across the empirical sciences. The original theoretical paper has been
cited in thousands of empirical studies, and the R implementation of `sensemakr`
[@cinelli2020sensemakr] has been used in published work spanning political science,
economics, epidemiology, sociology, education, and environmental science, among other fields.
The methodology has also been incorporated into textbooks and graduate-level courses on
causal inference, and the robustness value has become a standard reporting measure in
several applied disciplines.

By providing a Python implementation, `PySensemakr` extends the reach of these tools to the
large and growing community of researchers and data scientists who use Python as their
primary computing environment. The package includes a comprehensive test suite validated
against the R implementation, ensuring numerical equivalence across platforms. `PySensemakr`
is installable via `pip`, is hosted on PyPI, and includes documentation and example notebooks
to facilitate adoption.

# AI Usage Disclosure

Generative AI tools were used to assist with code maintenance tasks (e.g., updating deprecated
API calls for compatibility with newer versions of dependencies). All AI-generated changes
were reviewed and tested by the authors.

# Acknowledgements

We thank Chad Hazlett and Jeremy Ferwerda for their contributions to the theoretical
development and to the R and Stata implementations of `sensemakr`, which served as the
foundation for this Python package.

# References
