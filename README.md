# Hyvinvointialue Diabetes Risk Project

## Business question
Which wellbeing services counties (hyvinvointialueet) are showing the fastest deterioration in chronic disease burden relative to their socioeconomic trajectory?

## Why it matters
Finland's 21 hyvinvointialueet receive state funding partly through a need-adjusted coefficient (tarvevakioitu kerroin) calculated by THL from morbidity and socioeconomic registry data. That coefficient is backward-looking. This project explores whether regional socioeconomic trends can flag rising chronic disease risk earlier — before it fully shows up in the official statistics — as a decision-support layer alongside the existing funding formula.

## Data sources
- **THL Sotkanet API** — diabetes medication reimbursement rate (indicator 683), by hyvinvointialue, by year
- **Statistics Finland StatFin API** — median income, unemployment rate, % population 65+, by municipality
- **THL kunta → hyvinvointialue crosswalk** — used to aggregate municipality-level data to hyvinvointialue level

## Method
1. Pull diabetes indicator data by hyvinvointialue and year (Sotkanet)
2. Pull socioeconomic indicators by municipality and year (StatFin)
3. Aggregate municipality data to hyvinvointialue level (population-weighted)
4. Merge datasets, create lagged socioeconomic features
5. Label each hyvinvointialue-year as high/low risk based on year-over-year diabetes rate increase
6. Fit logistic regression on lagged socioeconomic features to predict risk label; interpret via odds ratios
7. Cluster hyvinvointialueet with k-means into risk profiles

## Status
🚧 In progress — data collection underway.

## Progress log

**1. Diabetes indicator (Sotkanet)**
- Used Sotkanet indicator 683: "% of population 40+ entitled to special (75%) reimbursement diabetes medication" — chosen as THL's own standard registry-based proxy for Type 2 diabetes prevalence
- Endpoint: `https://sotkanet.fi/rest/1.1/csv?indicator=683&years={year}&genders=total`
- **Note**: the API rejects comma-separated multi-year requests (400 error) and requires a browser-style `User-Agent` header (Colab's default `requests` UA gets a 403). Pulled one year at a time in a loop (2015–2023) and concatenated.
- Filtered to the 23 hyvinvointialue-level regions (21 counties + Helsinki + HUS) using the `/rest/1.1/regions` endpoint, matching on region `id` (not the per-category `code`), filtered where `category == 'HYVINVOINTIALUE'`
- Result: 207 rows (23 regions × 9 years)

**2. Income indicator (Statistics Finland / StatFin)**
- Used table 118w ("Income and income structure of household-dwelling units by region"), variable `tjt-ekvikturaha_med`: disposable monetary income per consumption unit, median — chosen over the plain (non-equivalized) median because it adjusts for household size/composition, making it comparable across regions with different demographics; it's also the standard EU/OECD measure for income and poverty statistics
- Confirmed via StatFin's table browser that hyvinvointialue (`HVA01`–`HVA21`, `HVA90`, `HVA91`) is available directly as a region option — this let us skip aggregating from municipality level entirely
- Endpoint: `https://pxdata.stat.fi/PxWeb/api/v1/en/StatFin/tjt/118w.px` (POST, json-stat2 format), parsed with `pyjstat`
- Result: 230 rows (23 regions × 10 years, 2015–2024)

**3. Merge**
- Joined diabetes data and income data on region name + year (income data's `Region` field needed the `HVA##` prefix stripped first to match)
- Result: 207 merged rows (23 regions × 9 overlapping years, 2015–2023)

**Next**: pulling % population 65+ from StatFin as a second socioeconomic predictor, then feature engineering (lagged variables, risk labels) and modeling.

## Results
_(to be filled in)_

## Limitations
_(to be filled in — e.g. small sample size across 21 regions, registry data undercounts diet-controlled diabetes cases)_