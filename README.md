# The GOAT's Blueprint: A Data-Driven Analysis of Faker's Competitive Performance

**By Ethan Baquiran** | UCSD DSC 80

---

## Introduction

Faker (Lee Sang-hyeok) is widely regarded as the greatest League of Legends player of all time. With multiple World Championship titles spanning over a decade, he has redefined what it means to compete at the highest level of professional esports. But what actually makes Faker win? Is it raw mechanics, early game dominance, or sustained damage output?

This project uses Oracle's Elixir match data from Faker's **2015 and 2025 competitive seasons** to answer one central question:

> **Is Faker's most impactful gameplay statistic in his strongest year (2015) still statistically significant in predicting match outcomes in 2025?**

2015 is widely considered Faker's peak — the year he won Worlds and was virtually untouchable. By comparing it to 2025, a decade later, we can examine whether the same individual performance metrics that defined his dominance still hold predictive power in the modern meta.

The dataset contains **81 rows for 2015** and **187 rows for 2025**, each representing one of Faker's individual game performances. The key columns used in this analysis are:

| Column | Description |
|--------|-------------|
| `result` | Win (1) or loss (0) |
| `kills` | Number of kills |
| `deaths` | Number of deaths |
| `assists` | Number of assists |
| `kda` | (kills + assists) / deaths — overall performance score |
| `golddiffat15` | Gold difference vs opponent at 15 minutes |
| `xpdiffat15` | XP difference vs opponent at 15 minutes |
| `csdiffat15` | CS difference vs opponent at 15 minutes |
| `damageshare` | Percentage of team's total damage dealt by Faker |
| `visionscore` | Vision control score |

---

## Data Cleaning

Before analyzing, we cleaned the data in several ways. First, we filtered both the 2015 and 2025 datasets to only include rows where `playername` is "faker", giving us 81 and 187 rows respectively. We then computed a new `kda` column as `(kills + assists) / deaths`, replacing 0 deaths with 1 to avoid division by zero. Boolean columns like `result`, `firstblood`, `firstdragon`, `firstbaron`, and `firsttower` were converted from their raw format to proper integers. Finally, we added a `year` column to each dataset and concatenated them into a single DataFrame of 268 rows for combined analysis.

The head of the cleaned DataFrame is shown below:

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

We first examined the distribution of Faker's KDA across his 2015 games. The histogram below shows a right-skewed distribution — most games fall between 0 and 5 KDA, but there is a long tail of exceptional performances extending past 20. This skew is driven by wins, where Faker frequently posts dominant performances.

<iframe src="assets/kda_dist.html" width="800" height="450" frameborder="0"></iframe>

The distribution reveals something important: Faker's KDA is not normally distributed. A small number of games have extremely high KDA values, which are almost exclusively wins. This asymmetry motivates our use of the mean difference as a test statistic in our hypothesis test.

### Bivariate Analysis

Next we examined how KDA relates to match outcomes. The box plot below compares KDA distributions in wins vs losses for 2015. The difference is striking — Faker's median KDA in wins is roughly 10, while in losses it hovers near 1.5. The interquartile ranges barely overlap, suggesting KDA is a very strong separator between wins and losses.

<iframe src="assets/kda_box.html" width="800" height="450" frameborder="0"></iframe>

### Interesting Aggregates

The table below shows mean statistics grouped by year and result. It tells a compelling story about how the game has changed between 2015 and 2025.

| year | result | kda | golddiffat15 | damageshare | xpdiffat15 | dpm | visionscore |
|------|--------|-----|-------------|-------------|------------|-----|-------------|
| 2015 | 0 | 1.731 | 2.667 | 0.310 | -80.722 | 587.131 | 0.000 |
| 2015 | 1 | 9.324 | 499.429 | 0.316 | 409.571 | 664.598 | 0.000 |
| 2025 | 0 | 1.772 | -284.435 | 0.241 | -256.797 | 582.114 | 35.145 |
| 2025 | 1 | 9.377 | 2.466 | 0.240 | 14.441 | 743.215 | 32.797 |

Two things stand out. First, **KDA is remarkably consistent** — Faker averages roughly 9.3 in wins and 1.7 in losses in both years, suggesting this metric has remained a stable indicator of his performance across a decade. Second, **gold difference at 15 changed dramatically** — in 2015 wins, Faker averaged a +499 gold lead at 15 minutes, but in 2025 this dropped to just +2.5. This reflects how modern professional play has become much more contested in the early game, with teams neutralizing lane advantages more effectively.

---

## Assessment of Missingness

Not all columns in the dataset are complete. We identified two types of missingness.

**Columns missing all 187 rows in 2025:** Columns like `dragons`, `towers`, `void_grubs`, and many others are missing for all of Faker's rows. This is **NMAR (Not Missing At Random)** — Oracle's Elixir only records these statistics for team summary rows, not individual player rows. The data is missing because of the structure of how it was collected, not due to random chance or game outcomes.

**Columns missing 5 rows:** The `golddiffat25` column and related 25-minute snapshot columns are missing in 5 games. We hypothesize these games ended before the 25-minute mark, meaning no snapshot was ever recorded. To formally test this, we ran a permutation test:

- **Null Hypothesis:** The missingness of `golddiffat25` is not dependent on match outcome.
- **Test Statistic:** Difference in win rates between games where `golddiffat25` is missing vs not missing.
- **Observed Difference:** -0.032
- **P-value:** 1.0000

We **fail to reject** the null hypothesis. The missingness of `golddiffat25` is **MCAR (Missing Completely At Random)** — games ending before 25 minutes appear to happen regardless of whether Faker wins or loses. These 5 rows are dropped when using 25-minute features.

---

## Hypothesis Testing

From our EDA, KDA emerged as the most dramatically different statistic between wins and losses in 2015. We now formally test whether this relationship still holds in 2025 — the key question of our research.

**Null Hypothesis:** The distribution of Faker's KDA is the same in wins and losses in 2025. Any observed difference is due to random chance.

**Alternative Hypothesis:** Faker's KDA is significantly higher in wins than losses in 2025.

**Test Statistic:** Difference in mean KDA (wins − losses)

**Significance Level:** 0.05

We ran a permutation test with 10,000 iterations, shuffling the `result` labels and computing the KDA difference each time. The observed difference was **7.605** — Faker averages 9.38 KDA in wins and only 1.77 in losses in 2025.

<iframe src="assets/permutation.html" width="800" height="450" frameborder="0"></iframe>

The histogram shows the distribution of permuted differences centered around 0, with the red line marking our observed value of 7.605. Out of 10,000 permutations, **none** produced a difference as large as the observed value, giving a p-value of less than 0.001.

We **reject the null hypothesis**. Faker's KDA is statistically significantly higher in wins than losses in 2025, directly answering our research question: **yes**, the most impactful stat from 2015 remains statistically significant a decade later.

---

## Framing a Prediction Problem

Building on our hypothesis test, we frame a prediction problem: **can we predict whether Faker wins a game based on his in-game performance statistics?**

This is a **binary classification** problem where the response variable is `result` (1 = win, 0 = loss). We chose this variable because it directly answers our research question and is a concrete, meaningful outcome.

The features we use are all quantitative performance metrics recorded during or at the end of the game:

| Feature | Type | Description |
|---------|------|-------------|
| `kda` | Quantitative | Overall combat performance |
| `golddiffat15` | Quantitative | Early game gold advantage |
| `xpdiffat15` | Quantitative | Early game XP advantage |
| `csdiffat15` | Quantitative | Early game farming advantage |
| `killsat15` | Quantitative | Kills by 15 minutes |
| `deathsat15` | Quantitative | Deaths by 15 minutes |
| `damageshare` | Quantitative | Team damage contribution |
| `visionscore` | Quantitative | Map control |
| `cspm` | Quantitative | Farming efficiency |
| `dpm` | Quantitative | Sustained damage output |

We use **accuracy** as our evaluation metric. Faker wins 63.1% of his 2025 games (118 wins, 69 losses) — a slight class imbalance, but not severe enough to require special handling. We also report precision and recall via the confusion matrix.

---

## Baseline Model

Our baseline model is a **Random Forest Classifier** trained on Faker's 2025 games using 6 features: KDA, gold difference at 15, XP difference at 15, CS difference at 15, kills at 15, and deaths at 15. We split the data 80/20 into training and test sets.

The model achieved **100% training accuracy and 94.7% test accuracy**. The gap between training and test accuracy indicates some overfitting — the model memorized the training data but still generalizes reasonably well.

Feature importance analysis revealed that **KDA alone accounts for 0.60 of total importance**, more than all other five features combined. This strongly confirms our hypothesis test finding — KDA is the dominant predictor of match outcomes.

This baseline is a solid starting point but can be improved by adding more features, limiting tree depth to reduce overfitting, and tuning hyperparameters systematically.

---

## Final Model

To improve on the baseline, we added four new features — `damageshare`, `visionscore`, `cspm`, and `dpm` — and tuned hyperparameters using **GridSearchCV** with 5-fold cross-validation. These features were chosen because they capture different dimensions of Faker's performance: team contribution, map awareness, farming efficiency, and sustained damage.

The final model achieved **95% test accuracy** — a modest but meaningful improvement over the baseline's 94.7%. More importantly, the model is less overfit, with cross-validation confirming it generalizes well across different subsets of the data.

<iframe src="assets/feature_importance.html" width="800" height="450" frameborder="0"></iframe>

KDA remains the dominant predictor at 0.49 importance, but **DPM emerged as the second most important feature at 0.11** — suggesting that Faker's sustained damage output, not just his kill participation, plays a meaningful role in 2025 outcomes. This is consistent with how the modern meta rewards consistent damage dealers over high-variance carry styles.

---

## Fairness Analysis

Finally, we asked: does our model perform equally well for **2015 games** compared to **2025 games**? Since our research question explicitly compares these two eras, it's important to verify the model isn't biased toward one year's data patterns.

We used 5-fold cross-validation on each year separately:

- **2015 CV Accuracy:** 86.5% (± 6.8%)
- **2025 CV Accuracy:** 90.9% (± 2.6%)
- **Difference:** 4.4%

<iframe src="assets/fairness.html" width="800" height="450" frameborder="0"></iframe>

The 2025 model performs slightly better, likely because it has more training data (187 vs 81 games) and lower variance. The 4.4% difference falls within our fair threshold of 5%, so we conclude the **model is fair across years** — it generalizes well to both eras rather than being biased toward one.

This is an encouraging result. It means the performance patterns we identified — KDA as the dominant predictor, DPM as secondary — hold across both of Faker's eras, lending further support to our central finding.
