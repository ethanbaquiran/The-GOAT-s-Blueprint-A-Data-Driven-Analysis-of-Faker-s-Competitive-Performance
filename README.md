# The GOAT's Blueprint: A Data-Driven Analysis of Faker's Competitive Performance

**By Ethan Baquiran** | UCSD DSC 80

---

## Introduction

Faker (Lee Sang-hyeok) is widely regarded as the greatest League of Legends player of all time. This project uses Oracle's Elixir match data from his **2015 and 2025 competitive seasons** to answer:

> **Is Faker's most impactful gameplay statistic in his strongest year (2015) still statistically significant in predicting match outcomes in 2025?**

The dataset contains 81 rows for 2015 and 187 rows for 2025, each representing one of Faker's individual game performances. Relevant columns:

| Column | Description |
|--------|-------------|
| `result` | Win (1) or loss (0) |
| `kills` | Number of kills |
| `deaths` | Number of deaths |
| `assists` | Number of assists |
| `kda` | (kills + assists) / deaths |
| `golddiffat15` | Gold difference at 15 minutes |
| `xpdiffat15` | XP difference at 15 minutes |
| `csdiffat15` | CS difference at 15 minutes |
| `damageshare` | % of team damage dealt by Faker |
| `visionscore` | Vision control score |

---

## Data Cleaning

We filtered both datasets to only Faker's rows, computed KDA as `(kills + assists) / deaths`, fixed boolean columns, and concatenated both years into a single DataFrame of 268 rows.

| kda | golddiffat15 | damageshare | visionscore | result | year |
|-----|-------------|-------------|-------------|--------|------|
| 1.25 | 152.0 | 0.434 | 0.0 | 0 | 2015 |
| 8.00 | 2628.0 | 0.449 | 0.0 | 1 | 2015 |
| 0.25 | -856.0 | 0.278 | 0.0 | 0 | 2015 |
| 2.75 | 415.0 | 0.251 | 0.0 | 1 | 2015 |
| 15.00 | 1151.0 | 0.444 | 0.0 | 1 | 2015 |

---

## Exploratory Data Analysis

### Univariate Analysis

The distribution of Faker's KDA in 2015 is right-skewed — most games cluster between 0-5 KDA, but wins pull the distribution toward higher values.

<iframe src="assets/kda_dist.html" width="800" height="450" frameborder="0"></iframe>

### Bivariate Analysis

KDA shows a dramatic difference between wins and losses — Faker averages 9.3 KDA in wins vs 1.7 in losses in 2015.

<iframe src="assets/kda_box.html" width="800" height="450" frameborder="0"></iframe>

### Interesting Aggregates

| year | result | kda | golddiffat15 | damageshare | xpdiffat15 | dpm | visionscore |
|------|--------|-----|-------------|-------------|------------|-----|-------------|
| 2015 | 0 | 1.731 | 2.667 | 0.310 | -80.722 | 587.131 | 0.000 |
| 2015 | 1 | 9.324 | 499.429 | 0.316 | 409.571 | 664.598 | 0.000 |
| 2025 | 0 | 1.772 | -284.435 | 0.241 | -256.797 | 582.114 | 35.145 |
| 2025 | 1 | 9.377 | 2.466 | 0.240 | 14.441 | 743.215 | 32.797 |

KDA remains consistently predictive across both years. Gold diff at 15 was massive in 2015 wins (+499) but dropped dramatically in 2025 (+2.5), showing how the meta shifted.

---

## Assessment of Missingness

Columns like `dragons` and `towers` are missing for all player rows — these are **NMAR** as Oracle's Elixir only records them for team summary rows.

For `golddiffat25` (missing 5 rows), we ran a permutation test:
- **Null:** Missingness is not dependent on match outcome
- **P-value:** 1.0000
- **Conclusion:** MCAR — games ending before 25 minutes happen regardless of outcome

---

## Hypothesis Testing

**Null Hypothesis:** The distribution of Faker's KDA is the same in wins and losses in 2025. Any observed difference is due to random chance.

**Alternative Hypothesis:** Faker's KDA is significantly higher in wins than losses in 2025.

**Test Statistic:** Difference in mean KDA (wins - losses)

**Observed difference:** 7.605 | **P-value:** < 0.001

<iframe src="assets/permutation.html" width="800" height="450" frameborder="0"></iframe>

We **reject the null hypothesis** — KDA is a statistically significant predictor of match outcomes in 2025, confirming it remains impactful from 2015.

---

## Framing a Prediction Problem

**Prediction Problem:** Predict whether Faker wins (`result`) based on his in-game performance statistics.

**Type:** Binary Classification

**Features:** KDA, gold diff at 15, XP diff at 15, CS diff at 15, kills at 15, deaths at 15, damage share, vision score, CSPM, DPM

**Metric:** Accuracy (63.1% win rate — slight imbalance but acceptable)

---

## Baseline Model

A Random Forest Classifier using 6 features achieved:
- **Training Accuracy:** 100%
- **Test Accuracy:** 94.7%

KDA dominated feature importance at 0.60 — confirming our EDA findings.

---

## Final Model

Adding `damageshare`, `visionscore`, `cspm`, and `dpm` with GridSearchCV tuning achieved:
- **Test Accuracy:** 95%
- **Mean CV Accuracy:** reported from cross-validation

<iframe src="assets/feature_importance.html" width="800" height="450" frameborder="0"></iframe>

KDA remains the dominant predictor at 0.49. DPM emerged as the second most important feature at 0.11.

---

## Fairness Analysis

**Question:** Does the model perform equally well for 2015 vs 2025 games?

- **2015 CV Accuracy:** 86.5% (+/- 6.8%)
- **2025 CV Accuracy:** 90.9% (+/- 2.6%)
- **Difference:** 4.4% — within fair threshold

<iframe src="assets/fairness.html" width="800" height="450" frameborder="0"></iframe>

**Conclusion:** The model is fair across years — it generalizes well across both of Faker's eras.
