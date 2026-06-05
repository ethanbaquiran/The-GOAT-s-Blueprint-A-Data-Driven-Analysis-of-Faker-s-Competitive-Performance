---
layout: home
---

# The GOAT's Blueprint: A Data-Driven Analysis of Faker's Competitive Performance

**By Ethan Baquiran** | UCSD DSC 80

---

## Introduction

This project analyzes **Faker** (Lee Sang-hyeok), widely regarded as the greatest 
League of Legends player of all time, using Oracle's Elixir match data from his 
**2015 and 2025 competitive seasons**.

**Research Question:** Is Faker's most impactful gameplay statistic in his strongest 
year (2015) still statistically significant in predicting match outcomes in 2025?

The dataset contains 81 rows for 2015 and 187 rows for 2025, each representing one 
of Faker's individual game performances. Relevant columns:

| Column | Description |
|--------|-------------|
| `result` | Win (1) or loss (0) |
| `kills` | Kills |
| `deaths` | Deaths |
| `assists` | Assists |
| `kda` | (kills + assists) / deaths |
| `golddiffat15` | Gold difference at 15 minutes |
| `xpdiffat15` | XP difference at 15 minutes |
| `csdiffat15` | CS difference at 15 minutes |
| `damageshare` | % of team damage dealt |
| `visionscore` | Vision control score |
