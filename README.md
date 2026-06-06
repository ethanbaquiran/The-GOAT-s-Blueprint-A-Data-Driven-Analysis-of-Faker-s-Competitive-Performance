# Faker: The Timeless Legend — Can 2015's Best Stat Still Predict Wins in 2025?

**By Ethan Baquiran** | UCSD DSC 80 Final Project

🌐 **[View the full report →](https://ethanbaquiran.github.io/The-GOAT-s-Blueprint-A-Data-Driven-Analysis-of-Faker-s-Competitive-Performance/)**

---

## Overview

Faker (Lee Sang-hyeok) is widely regarded as the greatest League of Legends player of all time, having won the World Championship in 2013, 2015, 2016, 2023, and 2024. This project uses professional match data from Oracle's Elixir to compare his performance across two seasons separated by a decade.

**Central question:** Was KDA — Faker's most impactful performance stat in 2015 — still a statistically significant predictor of his match outcomes in 2025?

---

## Dataset

Data sourced from [Oracle's Elixir](https://oracleselixir.com/), filtered to Faker's individual player rows:

| Season | Games |
| --- | --- |
| 2015 | 81 |
| 2025 | 187 |
| **Total** | **268** |

Key features used: `kda`, `golddiffat15`, `xpdiffat15`, `csdiffat15`, `killsat15`, `deathsat15`, `damageshare`, `visionscore`, `cspm`, `dpm`, `result`

> Note: `visionscore` was not tracked by Oracle's Elixir in 2015 and is zero for all 2015 rows. It is only used as a feature in models trained on 2025 data.

---

## Project Structure

```
├── template.ipynb          # Full analysis notebook (submit as PDF to Gradescope)
├── index.md                # Website source (rendered at the link above)
├── _config.yml             # Jekyll site configuration
├── assets/
│   ├── kda_dist.html           # Univariate: KDA distribution (2015)
│   ├── golddiff_dist.html      # Univariate: Gold diff at 15 distribution (2015)
│   ├── kda_box.html            # Bivariate: KDA by result (2015)
│   ├── permutation.html        # Hypothesis test permutation distribution
│   ├── feature_importance.html # Final model feature importances
│   ├── confusion_matrix.html   # Final model confusion matrix
│   └── fairness.html           # Fairness analysis bar chart
```

---

## Analysis Summary

### Step 1 — Introduction
Framed the question around Faker's cross-era consistency, with the Oracle's Elixir dataset as the source.

### Step 2 — Data Cleaning & EDA
- Computed KDA as `(kills + assists) / max(deaths, 1)`
- Fixed boolean columns (`result`, `firstblood`, etc.)
- Added a `year` column for cross-era comparison
- Univariate: KDA is right-skewed in 2015 — floor is low, ceiling is extraordinary
- Bivariate: KDA separates wins/losses far more clearly than any other feature
- Aggregate: KDA win/loss gap is nearly identical in 2015 and 2025 (~7.6 in both years)

### Step 3 — Assessment of Missingness
- **NMAR:** Team-level columns (`dragons`, `towers`, etc.) are structurally absent from player rows
- **MCAR:** `golddiffat25` is missing in 5 games (ended before 25 min); permutation test confirms missingness is unrelated to match outcome (p ≈ 1.0)
- **MAR:** `golddiffat25` missingness depends on `|golddiffat15|` — games ending early tend to have more extreme early gold leads (p < 0.05)

### Step 4 — Hypothesis Testing
- **Null:** KDA distribution is the same in wins and losses in 2025
- **Alternative:** KDA is significantly higher in wins
- **Result:** Observed difference = 7.605; p-value < 0.0001 (10,000 permutations) → **reject null**

### Step 5 — Prediction Problem
Binary classification: predict `result` (win/loss) from Faker's in-game stats. Evaluated by accuracy (63.1% baseline win rate).

### Step 6 — Baseline Model
Random Forest on 6 features (`kda`, `golddiffat15`, `xpdiffat15`, `csdiffat15`, `killsat15`, `deathsat15`):
- Training accuracy: 100%
- Test accuracy: 94.7%
- KDA dominates at 0.60 feature importance

### Step 7 — Final Model
Added `damageshare`, `visionscore`, `cspm`, `dpm`; tuned with `GridSearchCV` (5-fold CV):
- Test accuracy: ~95%
- Mean CV accuracy: ~90.9% (±0.026)
- KDA remains top feature at ~0.49; DPM second at ~0.11

### Step 8 — Fairness Analysis
Does the model generalize equally across eras?
- 2015 CV accuracy: 86.5% | 2025 CV accuracy: 90.9% | Difference: 4.4%
- Permutation test (500 iterations): p > 0.05 → **model is fair across years**

---

## Conclusion

KDA, Faker's most dominant stat in 2015, remains the strongest predictor of his match outcomes in 2025. Despite a decade of meta changes, rule updates, and an entirely new professional landscape, the fundamental relationship between elite individual performance and winning is unchanged. The model trained across both eras confirms this with comparable accuracy in both years.

---

## How to Run

1. Download the 2015 and 2025 Oracle's Elixir CSV files and place them in the project root
2. Open `template.ipynb` in Jupyter
3. Run all cells top-to-bottom
4. The final cell exports all interactive charts to `assets/`

**Dependencies:** `pandas`, `numpy`, `plotly`, `scikit-learn`

```bash
pip install pandas numpy plotly scikit-learn
```
