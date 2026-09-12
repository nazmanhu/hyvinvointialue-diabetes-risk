# Hyvinvointialue Diabetes Risk Project

## Business question
Which wellbeing services counties (hyvinvointialueet) carry a disproportionately high diabetes burden relative to their income level and population age structure — and can that relationship be modeled well enough to flag a "watch list" of regions for closer attention?

*(Note: the project originally set out to predict year-over-year spikes in diabetes burden from lagged socioeconomic data. That approach showed no meaningful predictive signal — see Method/Results below — so the question was reframed to a cross-sectional one: explaining relative burden across regions in a given year, which the data supports well.)*

## Why it matters
Finland's 21 hyvinvointialueet receive state funding partly through a need-adjusted coefficient (tarvevakioitu kerroin) calculated by THL from morbidity and socioeconomic registry data. This project is a lightweight, open-data replica of that logic at a smaller scale: showing that income and age structure alone explain a meaningful share of regional variation in diabetes burden, and using that relationship to flag regions carrying elevated risk — a possible decision-support layer alongside the existing funding formula.

## Data sources
- **THL Sotkanet API** — indicator 683: % of population 40+ entitled to special (75%) reimbursement diabetes medication, by hyvinvointialue, 2005–2023
- **Statistics Finland StatFin API**, table 118w — disposable monetary income per consumption unit (median), by hyvinvointialue, 2005–2024
- **Statistics Finland StatFin API**, table 11ra — share of population aged 65+ (%), by hyvinvointialue, 2005–2023

All three sources support the hyvinvointialue region breakdown natively, so no municipality-level aggregation was needed.

## Method
1. Pull diabetes indicator data by hyvinvointialue and year (Sotkanet)
2. Pull income and % 65+ indicators by hyvinvointialue and year (StatFin)
3. Merge all three datasets on region name and year
4. **First approach (dropped):** label each hyvinvointialue-year as high/low risk based on year-over-year diabetes rate increase, predict from lagged (prior-year) income and % 65+ via logistic regression. Cross-validated accuracy (56.8%) came in below the majority-class baseline (59%) — no real signal.
5. **Final approach:** label each hyvinvointialue-year as high/low risk based on whether its diabetes rate is in the top third *across regions in that year* (cross-sectional, not longitudinal). Predict from same-year income and % 65+ via logistic regression.
6. Cluster hyvinvointialueet with k-means (on diabetes rate, income, % 65+ jointly) into risk profiles for the most recent year.

## Results

**Logistic regression** (cross-sectional risk label, group k-fold CV by region, n=437 region-years):
- Mean CV accuracy: **70.2%**, consistently above the 64.5% majority-class baseline across all folds
- Odds ratios: **median income = 0.121** (higher income → much lower odds of high burden), **% 65+ = 2.646** (older population → notably higher odds of high burden)
- Income is the stronger driver of the two

**K-means clustering** (2023 snapshot, 4 clusters):
| Cluster | Profile | Diabetes rate (avg) | Median income (avg) | % 65+ (avg) | Regions |
|---|---|---|---|---|---|
| 2 | **Elevated burden — lower income, older population** | 13.2% | €26,405 | 28.9% | Kainuu, Kanta-Häme, Kymenlaakso, Lapland, North Karelia, North Savo, Päijät-Häme, Satakunta, South Karelia, South Ostrobothnia, South Savo |
| 1 | Moderate burden | 11.9% | €27,075 | 23.4% | Central Finland, Central Ostrobothnia, North Ostrobothnia, Ostrobothnia, Pirkanmaa, Southwest Finland |
| 3 | **Low burden — high income, younger population** | 10.2% | €30,910 | 19.2% | Central Uusimaa, City of Helsinki, East Uusimaa, Vantaa and Kerava, West Uusimaa |
| 0 | Outlier | 6.8% | €32,069 | 24.2% | Åland |

Cluster 2 is the project's "watch list" — mostly Eastern and Northern Finland, forming a clear regional pattern that matches well-documented Finnish health disparities. Cluster 3 (Uusimaa/Helsinki metro) is the clear low-risk group.

A horizontal bar chart of diabetes rate by region, colored by cluster, is included in `/outputs` as the primary visualization — a PCA scatter plot was also produced during exploration but excluded from the final writeup as less readable at a glance.

## Limitations
- Sample size is modest (23 regions, up to 19 years) — the original longitudinal (spike-prediction) approach didn't hold up under this constraint, which is why the project reports the cross-sectional reframing instead of a negative result
- Correlational, not causal — income and age structure are associated with diabetes burden but this doesn't establish mechanism
- Registry-based reimbursement data undercounts diet/lifestyle-controlled diabetes cases not receiving the specific reimbursed medication category
- Only two predictors used; other plausible drivers (healthcare access, urbanicity, ethnic/immigrant population composition) were not included
- Å land behaves as a structural outlier (small population, different administrative status) and skews cluster boundaries slightly

## Possible next steps
- Add a Power BI dashboard for interactive exploration (in progress)
- Incorporate THL's actual funding coefficient data to compare against this model's risk flags directly
- Extend with additional socioeconomic predictors (unemployment, education) or try XGBoost + SHAP for a richer, nonlinear model