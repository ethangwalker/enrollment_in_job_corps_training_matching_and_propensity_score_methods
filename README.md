# Matching and Propensity Score Methods: Revisiting LaLonde with Dehejia & Wahba

Can non-experimental methods recover an experimental treatment effect? This project re-estimates the effect of the **National Supported Work (NSW)** job-training program on 1978 earnings using Dehejia and Wahba's (1999) non-experimental sample. The sample pairs the NSW treated group with controls from the **Panel Study of Income Dynamics (PSID)**. Each estimate is compared against the experimental benchmark ATT of **$1,794.34**.

The analysis follows an unconfoundedness-based design. It diagnoses covariate imbalance, estimates a propensity score, imposes overlap and compares a set of matching and propensity-score estimators with naive OLS.

> Problem Set 3 for ECON 440 (Unconfoundedness-Based Designs), written in Quarto and R.

---

## Key Findings

- **Naive regression fails badly.** The unadjusted difference in means is **−$15,581**. The OLS-based ATT estimator built from a controls-only outcome regression is **−$547**, which has the wrong sign.
- **Matching recovers the right neighborhood.** One-to-one covariate matching on the full sample gives **$1,474**, within a few hundred dollars of the benchmark.
- **Trimming to the overlap region helps.** The Dehejia–Wahba overlap rule drops 503 PSID controls whose propensity scores fall outside the treated range. All three main estimators then land between **$1,398 and $2,120**.
- **Thin overlap leaves the results imprecise and fragile.** Standard errors run from about $1,900 to $2,900, compared with $633 in the experiment. Estimates also swing widely with specification: switching to a Mahalanobis metric gives $154, and a quadratic p-score regression gives −$1,604.

The broader takeaway is that matching and propensity-score methods do not fix bad data. They make the limits of the data visible in a way that OLS does not.

---

## Results

ATT of the NSW program on 1978 earnings (standard errors in parentheses):

| Estimator | ATT | SE |
|---|---:|---:|
| **Without imposing overlap** (n = 1,088) | | |
| Unadjusted difference of means | −15,580.89*** | (1,222.93) |
| OLS on treatment + covariates | −1,504.30 | (1,257.62) |
| OLS outcome model on controls → impute Y(0) for treated | −547.06 | (1,334.44) |
| Covariate matching (1 match, Abadie–Imbens) | 1,473.78 | (1,952.24) |
| **After imposing overlap – main results** (n = 585) | | |
| Covariate matching (1 match, Abadie–Imbens) | 1,482.05 | (1,923.32) |
| P-score matching (1 match) | 1,398.43 | (2,356.13) |
| OLS on cubic p-score for controls | 2,119.70 | (2,879.92) |
| **Sensitivity analysis** | | |
| Covariate matching (1 match, Mahalanobis) | 153.65 | (2,019.96) |
| Covariate matching (2 matches, Abadie–Imbens) | 1,247.11 | (1,992.85) |
| Covariate matching (3 matches, Abadie–Imbens) | 876.67 | (1,767.45) |
| P-score matching (2 matches) | 1,489.65 | (2,822.75) |
| P-score matching (3 matches) | 1,180.62 | (2,769.90) |
| OLS on linear p-score for controls | 2,580.95 | (2,600.49) |
| OLS on quadratic p-score for controls | −1,603.87 | (1,967.71) |

*Experimental benchmark: $1,794.34 (SE $633). Matching SEs are the Abadie–Imbens SEs from the `Matching` package. SEs for the OLS-based ATT estimators come from 1,000 bootstrap replications. \*p<0.10, \*\*p<0.05, \*\*\*p<0.01.*

---

## Methodology

1. **Feature construction.** Create unemployment indicators `u74` and `u75`, equal to 1 when earnings in that year were zero.
2. **Balance diagnostics.** Compute Welch t-tests of covariate means by treatment status. Every covariate except `hispanic` is significantly imbalanced. Treated individuals are younger, less educated, less often married and earned roughly $18K less in each pre-treatment year.
3. **Estimand choice.** Target the **ATT** rather than the ATE, because the PSID controls mostly look nothing like program participants.
4. **Reference OLS estimates.** Run the unadjusted difference, OLS with covariates and an OLS imputation estimator.
5. **Covariate matching.** Use nearest-neighbor matching on all ten covariates (`Matching::Match`, `estimand = "ATT"`).
6. **Propensity score.** Fit the Dehejia–Wahba (1999, Table 3, note *e*) logit specification: age, age², education, education², married, nodegree, black, hispanic, re74, re75, re74², re75² and `u74 × black`.
7. **Overlap.** Plot p-score histograms by group, then apply the DW trimming rule, which keeps observations with p̂ inside the [min, max] range of the treated. Re-check balance and examine p-score quantiles within the trimmed sample.
8. **Main estimators within overlap.** Compare covariate matching, p-score matching and an OLS regression on a cubic in the p-score.
9. **Sensitivity analysis.** Vary the distance metric, the number of matches (M = 1, 2, 3) and the functional form of the p-score regression.
10. **Write-up.** Discuss what would change with experimental controls and summarize the lessons for unconfoundedness-based designs.

---

## Repository Structure

```
.
├── matching_and_propensity_score_methods.qmd   # Quarto source: analysis + write-up
├── matching_and_propensity_score_methods.pdf   # Rendered report (16 pages, code appendix included)
├── nsw_dw_psid_small_econ440.xls               # Data (NSW treated + PSID controls)
└── README.md
```

---

## Data

`nsw_dw_psid_small_econ440.xls` has **1,088 observations**: **185 NSW treated** and **903 PSID controls**.

| Variable | Description |
|---|---|
| `data_id` | Source sample (`NSW` / `PSID`) |
| `treat` | 1 = received NSW job training, 0 = PSID comparison |
| `age` | Age in years |
| `education` | Years of schooling |
| `black`, `hispanic` | Race/ethnicity indicators |
| `married` | Married indicator |
| `nodegree` | No high school degree indicator |
| `re74`, `re75` | Real earnings in 1974 and 1975 (pre-treatment) |
| `re78` | Real earnings in 1978 (**outcome**) |

`u74` and `u75` are derived in the analysis.

---

## Reproducing the Analysis

**Requirements**
- R (≥ 4.1, for the native `|>` pipe and `\(x)` lambdas)
- [Quarto](https://quarto.org/) with a LaTeX distribution for PDF output (e.g. `quarto install tinytex`)
- R packages:

```r
install.packages(c("tidyverse", "readxl", "Matching", "knitr"))
```

**Render**

```bash
quarto render matching_and_propensity_score_methods.qmd
```

The `.xls` file must sit in the same directory as the `.qmd`. Code chunks are hidden in the report body, and the complete code is reprinted in the PDF's appendix. Bootstrap standard errors use fixed seeds (1–4), so the results reproduce exactly.

---

## References

- Dehejia, R. H., & Wahba, S. (1999). Causal Effects in Nonexperimental Studies: Reevaluating the Evaluation of Training Programs. *Journal of the American Statistical Association*, 94(448), 1053–1062.
- LaLonde, R. J. (1986). Evaluating the Econometric Evaluations of Training Programs with Experimental Data. *American Economic Review*, 76(4), 604–620.
- Abadie, A., & Imbens, G. W. (2006). Large Sample Properties of Matching Estimators for Average Treatment Effects. *Econometrica*, 74(1), 235–267.
- Sekhon, J. S. (2011). Multivariate and Propensity Score Matching Software with Automated Balance Optimization: The Matching Package for R. *Journal of Statistical Software*, 42(7).

---

**Author:** Ethan Walker