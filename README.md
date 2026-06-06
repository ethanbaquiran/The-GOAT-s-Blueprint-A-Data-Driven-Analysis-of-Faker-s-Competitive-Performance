# Faker: The Timeless Legend — Can 2015's Best Stat Still Predict Wins in 2025?

## A DSC 80 Final Project analyzing Faker's performance across 2015 and 2025 — Ethan Baquiran

[View on GitHub](https://github.com/ethanbaquiran/The-GOAT-s-Blueprint-A-Data-Driven-Analysis-of-Faker-s-Competitive-Performance)

**Name:** Ethan Baquiran

---

## Introduction

League of Legends (LoL) is a globally dominant multiplayer online battle arena (MOBA) developed by Riot Games. It has one of the most storied professional esports histories in the world, with international tournaments drawing tens of millions of viewers. The dataset used in this project comes from [Oracle's Elixir](https://oracleselixir.com/), a comprehensive repository of professional LoL esports match data — specifically covering games from the **2015** and **2025** seasons.

In the decade between these two years, one player has remained synonymous with greatness: **Faker** (Lee Sang-hyeok), the mid laner for T1 (formerly SKT T1), widely regarded as the greatest LoL player of all time. Faker won Worlds in 2013, 2015, 2016, 2023, and 2024. He was dominant in 2015 — and he was *still* competing at the highest level in 2025 at age 29.

**The central question driving this analysis:**
> **Was KDA — Faker's most impactful performance stat in 2015 — still a statistically significant predictor of his match outcomes a decade later in 2025?**

This question matters because the game has changed dramatically over 10 years: faster pacing, new objectives (void grubs, Rift Herald), major meta shifts, and an entirely different professional landscape. If the stat most predictive of Faker's wins in 2015 still holds up in 2025, it reveals something profound about both the consistency of elite play and the enduring structure of League of Legends itself.

The dataset contains one row per player per game. **It contains 81 Faker games from 2015 and 187 from 2025 (268 total after filtering to player rows only).** For Faker's individual rows, the most relevant columns are:

| Column         | Description                                              |
| -------------- | -------------------------------------------------------- |
| `playername`   | Player's in-game name (filtered to "faker")              |
| `result`       | Match outcome: 1 = win, 0 = loss                         |
| `kills`        | Number of kills in the game                              |
| `deaths`       | Number of deaths in the game                             |
| `assists`      | Number of assists in the game                            |
| `kda`          | (kills + assists) / max(deaths, 1) — our primary feature |
| `golddiffat15` | Gold difference vs. opponent at 15 minutes               |
| `xpdiffat15`   | XP difference vs. opponent at 15 minutes                 |
| `csdiffat15`   | CS difference vs. opponent at 15 minutes                 |
| `damageshare`  | Fraction of team's total damage Faker dealt              |
| `visionscore`  | Vision control and map information contribution          |
| `dpm`          | Damage per minute (sustained output over the game)       |
| `cspm`         | CS per minute (farming efficiency)                       |

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

The raw data was filtered to Faker's individual player rows from the 2015 and 2025 seasons. Key cleaning steps:

- **KDA computed** as `(kills + assists) / max(deaths, 1)` to avoid division-by-zero in deathless games.
- **Boolean columns** (`result`, `firstblood`, `firstdragon`, etc.) were cast to their correct types.
- **Year column** added to enable cross-era comparison.
- **Rows with missing `golddiffat15`** dropped when using 15-minute features (these are games that ended before 15 minutes).

The resulting dataset has **81 games from 2015** and **187 from 2025**. Below is the head of the cleaned DataFrame:

| kda    | golddiffat15 | damageshare | visionscore | result | year |
| ------ | ------------ | ----------- | ----------- | ------ | ---- |
| 3.000  | 312          | 0.312       | 0           | 1      | 2015 |
| 1.500  | -204         | 0.271       | 0           | 0      | 2015 |
| 12.000 | 887          | 0.391       | 0           | 1      | 2015 |
| 2.000  | -511         | 0.248       | 0           | 0      | 2015 |
| 9.000  | 623          | 0.365       | 0           | 1      | 2015 |

Note: `visionscore` is 0 for all 2015 rows — this metric was not tracked by Oracle's Elixir until 2017.

### Univariate Analysis

The histogram below shows the distribution of Faker's KDA across his 2015 games. The distribution is heavily right-skewed — most games cluster between 2–6, but there is a long tail of dominant performances where his KDA exceeded 15 or even 20. This asymmetry is characteristic of elite carry players: the floor is mediocre, but the ceiling is extraordinary.

[KDA Distribution (2015)](assets/kda_dist.html)

The histogram below shows Faker's gold difference at 15 minutes in 2015. Unlike KDA, this is more symmetrically distributed and centered near 0 — even Faker loses laning phase sometimes, and gold leads are constrained by professional-level play on both sides.

[Gold Difference at 15 Distribution (2015)](assets/golddiff_dist.html)

### Bivariate Analysis

The box plot below splits Faker's KDA by match result in 2015. The separation is striking: in losses, the median KDA hovers around 1.5–2, while in wins it climbs to 6–10+. This single visualization foreshadows the hypothesis test — KDA is not random noise but a deeply outcome-linked variable.

[KDA by Game Result (2015)](assets/kda_box.html)

### Interesting Aggregates

The table below shows Faker's average stats grouped by year and result. Two things stand out immediately: (1) KDA remains the largest differentiator between wins and losses in *both* eras, and (2) DPM (damage per minute) is substantially higher across the board in 2025, reflecting the faster-paced modern meta.

| year | result | kda    | golddiffat15 | damageshare | xpdiffat15 | dpm   | visionscore |
| ---- | ------ | ------ | ------------ | ----------- | ---------- | ----- | ----------- |
| 2015 | 0      | 1.730  | -358.2       | 0.261       | -412.1     | 312.4 | 0.0         |
| 2015 | 1      | 9.320  | 502.8        | 0.341       | 531.7      | 408.9 | 0.0         |
| 2025 | 0      | 2.410  | -289.4       | 0.248       | -318.5     | 441.2 | 32.8        |
| 2025 | 1      | 10.015 | 441.3        | 0.312       | 422.9      | 589.7 | 38.4        |

---

## Assessment of Missingness

### NMAR Analysis

Columns like `dragons`, `towers`, and `void_grubs` are missing for **all of Faker's rows**. We believe this is **NMAR (Not Missing At Random)**. In the Oracle's Elixir dataset, these columns are only recorded in *team summary rows*, not individual player rows — so the missingness depends on the *type* of row (player vs. team), which is not captured by any other column in the data. No additional variable we could observe would make this missingness explainable; it is structural to how the data was collected.

If we could obtain a column indicating `row_type` (player vs. team summary), the missingness would become MAR, since team-level stats would always be present for team rows and always absent for player rows.

### Missingness Dependency Test 1: `golddiffat15` missingness vs. `result` — MCAR

We tested whether the missingness of `golddiffat15` in 2025 depends on `result` (win/loss).

- **Null Hypothesis:** Missingness of `golddiffat15` is independent of match outcome.
- **Alternative Hypothesis:** Missingness of `golddiffat15` depends on match outcome.
- **Test Statistic:** Difference in win rates between missing and non-missing groups.
- **Significance Level:** 0.05

After 1,000 permutations, the p-value was **> 0.05**. We **fail to reject** the null hypothesis. Games missing `golddiffat15` are simply games that ended before 15 minutes — and those short games happen regardless of whether Faker wins or loses. This is **MCAR (Missing Completely at Random)**.

### Missingness Dependency Test 2: `golddiffat15` missingness vs. `damageshare` — MAR

We then tested whether the missingness of `golddiffat15` depends on `damageshare`, a continuous feature. Games ending before 15 minutes tend to be early stomps, which may produce different damage share patterns.

- **Null Hypothesis:** Missingness of `golddiffat15` is independent of `damageshare`.
- **Alternative Hypothesis:** Missingness of `golddiffat15` depends on `damageshare`.
- **Test Statistic:** Absolute difference in mean `damageshare` between missing and non-missing groups.
- **Significance Level:** 0.05

After 1,000 permutations, the p-value was **< 0.05**. We **reject** the null hypothesis. `golddiffat15` is **MAR (Missing At Random)** on `damageshare` — games that end before 15 minutes tend to have different damage share distributions than full-length games.

---

## Hypothesis Testing

**Research Question:** Is KDA — the stat most strongly correlated with Faker's wins in 2015 — still a statistically significant predictor of match outcomes in 2025?

From EDA, KDA showed the largest win/loss gap of any feature in 2015 (1.73 in losses vs. 9.32 in wins). We now formally test whether this relationship persists a decade later.

- **Null Hypothesis:** The distribution of Faker's KDA is the same in wins and losses in 2025. Any observed difference is due to random chance.
- **Alternative Hypothesis:** Faker's KDA is significantly higher in wins than losses in 2025.
- **Test Statistic:** Difference in mean KDA (wins − losses) in 2025.
- **Significance Level:** 0.05

**Results:**

- Mean KDA in wins (2025): **10.015**
- Mean KDA in losses (2025): **2.410**
- Observed difference: **7.605**
- P-value (10,000 permutations): **< 0.0001**

[Permutation Test: KDA Difference](assets/permutation.html)

The permutation distribution above shows the null distribution across 10,000 shuffled label assignments. The red vertical line marks the observed difference of 7.605 — not a single permutation came close to matching it.

**Conclusion:** We **reject the null hypothesis**. Faker's KDA is a statistically significant predictor of match outcomes in 2025. A permutation test was the right choice here: it makes no distributional assumptions about KDA, and directly tests whether the observed gap could arise by random chance under the null. The result is unambiguous: the most impactful stat from Faker's 2015 season remains his most powerful performance signal a decade later.

---

## Framing a Prediction Problem

**Prediction Problem:** Predict whether Faker **wins** (`result = 1`) a game based on his in-game performance statistics.

This is a **binary classification** problem. The response variable is `result` (win = 1, loss = 0). We chose this because it directly answers our research question about which statistics predict match outcomes.

We use features available during or at the end of the game: `kda`, `golddiffat15`, `xpdiffat15`, `csdiffat15`, `killsat15`, `deathsat15`, `damageshare`, `visionscore`, `cspm`, and `dpm`. All are statistics recorded *during* the game — there is no data leakage, as these would all be known by the time we make a prediction.

We evaluate using **accuracy** as our primary metric. Faker wins approximately 63% of his 2025 games, so a naive "always predict win" baseline achieves ~63% — our models must meaningfully beat this. We also report precision, recall, and the confusion matrix to give a fuller picture.

---

## Baseline Model

The baseline model uses a **Random Forest Classifier** trained on **6 features**:

- `kda` (quantitative) — motivated by our hypothesis test showing it's the strongest win predictor
- `golddiffat15` (quantitative) — the most common proxy for mid-lane laning advantage
- `xpdiffat15`, `csdiffat15`, `killsat15`, `deathsat15` (all quantitative) — additional early-game snapshot stats

All 6 features are quantitative with no ordinal or nominal encoding required.

**Baseline Performance:**

- Training Accuracy: **100%**
- Test Accuracy: **~95%**

The 100% training accuracy signals overfitting — the model has memorized the training set. Feature importance confirms **KDA dominates**, accounting for more predictive weight than all other features combined. These weaknesses motivate the final model: we need hyperparameter tuning, cross-validation, and a richer feature set to get a more generalizable model.

---

## Final Model

**New Features Added:**

- `damageshare` — fraction of team damage Faker dealt; measures how much he carried
- `visionscore` — tactical map control and information advantage
- `cspm` — farming efficiency across the full game (not just at 15 min)
- `dpm` — sustained damage output throughout the game

Each feature captures a different dimension of mid-lane performance. KDA reflects peak moments; DPM reflects consistency; vision score reflects awareness; CS/XP differences reflect laning mastery. Together they give the model a much richer picture than the baseline's features.

**Model:** Random Forest Classifier with `GridSearchCV` (5-fold CV) over:

```python
param_grid = {
    'n_estimators': [50, 100, 200],
    'max_depth': [3, 5, 10, None],
    'min_samples_split': [2, 5, 10]
}
```

**Final Model Performance:**

- Test Accuracy: **~95%**
- Mean Cross-Validation Accuracy: **~88% (±0.04)**

The feature importance plot below shows that **KDA dominates at ~0.49 importance**, directly confirming our research question. DPM emerged as the second most important feature (~0.11), a new signal specific to the 2025 meta where overall damage output is higher.

[Feature Importance — Final Model](assets/feature_importance.html)

The confusion matrix below summarizes predictions on the held-out test set:

[Confusion Matrix — Final Model](assets/confusion_matrix.html)

The final model improved on the baseline by: (1) adding DPM, damage share, and vision score as features; (2) using GridSearchCV with cross-validation to select hyperparameters and reduce overfitting; and (3) evaluating with 5-fold CV rather than a single train/test split for more reliable accuracy estimates.

---

## Fairness Analysis

**Question:** Does our model perform equally well for Faker's 2015 games compared to his 2025 games?

Since our research question specifically compares Faker across eras, fairness across years is essential. If the model only works for one era, our cross-era comparisons are invalid.

- **Group X:** 2015 games
- **Group Y:** 2025 games
- **Evaluation Metric:** Accuracy (5-fold cross-validation within each group)

- **Null Hypothesis:** The model's accuracy is the same for 2015 and 2025 games. Any observed difference is due to random chance.
- **Alternative Hypothesis:** The model's accuracy differs significantly between 2015 and 2025 games.
- **Test Statistic:** Difference in accuracy (2015 accuracy − 2025 accuracy)
- **Significance Level:** 0.05

We use a **permutation test**: shuffle the year labels 1,000 times and recompute the accuracy difference each time to build a null distribution. The p-value is the fraction of permutations that produce a difference as extreme as the one observed.

**Results:**

- 2015 CV Accuracy: **86.5% (±~0.06)**
- 2025 CV Accuracy: **90.9% (±~0.05)**
- Observed Difference: **−4.4%**
- P-value (permutation test, n=1,000): **≥ 0.05 → Fail to reject null**

[Fairness Analysis: Accuracy by Year + Permutation Test](assets/fairness.html)

**Conclusion:** We **fail to reject the null hypothesis**. The 4.4% gap is consistent with random variation under the null. Our model is **fair across years** — it generalizes comparably to both eras rather than being biased toward one. The slightly higher 2025 accuracy likely reflects that the modern meta produces sharper, more extreme performance signals, not model bias. This supports the validity of our central finding: KDA has been, and remains, the defining metric of Faker's legendary performance.

---
