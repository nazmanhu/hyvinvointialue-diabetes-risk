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
🚧 In progress — repo scaffold created, data pull next.

## Results
_(to be filled in)_

## Limitations
_(to be filled in — e.g. small sample size across 21 regions, registry data undercounts diet-controlled diabetes cases)_
