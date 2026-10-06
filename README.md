# Neighborhood Conditions and NYC Restaurant Health Inspections

Do neighbourhood conditions show up in a restaurant's health grade? This project builds the dataset
and analysis plan to find out. It joins New York City's restaurant inspection records with
sanitation complaints from 311 and census demographics, producing a clean, one-row-per-restaurant
dataset ready for modelling.

**Scope, stated plainly:** this is the data-preparation and proposal stage of a team project. The
pipeline and the cleaned dataset are complete. The statistical analysis was designed and presented,
not run.

Columbia University, Applied Machine Learning II (APANPS5205), August 2026.
Team: Nicole Liu, Aziza Temirova, Tristan Patterson and Yuchun Wu.
Slides: [ML II.pptx (1).pdf](ML%20II.pptx%20(1).pdf) · pipeline: [data_preparation.ipynb](data_preparation.ipynb).

## Research questions

1. Do inspection scores and letter grades differ across the five boroughs?
2. Are neighbourhood sanitation conditions, measured by sanitation-related 311 requests, related to
   inspection outcomes?
3. Are demographics (median household income, poverty rate, population density, educational
   attainment) associated with inspection scores?
4. Which neighbourhood characteristics are the strongest predictors of inspection performance?

## The pipeline

Three public sources, merged on ZIP code:

| Source | Raw size | What's kept |
|---|---|---|
| [DOHMH Restaurant Inspection Results](https://data.cityofnewyork.us/Health/DOHMH-New-York-City-Restaurant-Inspection-Results/43nn-pn8j) | 296,044 violation-level rows | each restaurant's most recent routine (cycle) inspection, Jan 2023 – Jun 2026 |
| [NYC 311 Service Requests](https://data.cityofnewyork.us/Social-Services/311-Service-Requests-from-2020-to-Present/erm2-nwe9) | 21.7M requests | 1.15M sanitation complaints (rodents tracked separately), counted per ZIP |
| [ACS 5-year estimates](https://data.census.gov/) | 4 tables at ZCTA level | population, density, median income, poverty rate, % bachelor's or higher |

How the inspection data gets from raw to one row per restaurant:

| Step | Rows |
|---|---|
| Violation-level rows in the window, with a score | 258,567 |
| Valid NYC boroughs and ZIP codes | 255,712 |
| Routine cycle inspections only (scores comparable across restaurants) | 198,551 |
| Collapsed to inspection level | 59,841 |
| **Most recent inspection per restaurant** | **21,472** |

Details worth knowing:

- **Violation-level to inspection-level.** The raw file has one row per violation, so inspections are
  collapsed first, creating `n_violations` and `n_critical_violations`.
- **Grades derived from scores.** 12.5% of posted letter grades are still pending, so
  `grade_from_score` uses the official DOHMH cutoffs (A 0–13, B 14–27, C 28+) and is never missing.
- **Robust complaint matching.** 311 category names changed over the years (and again in a 2026 schema
  rename), so sanitation complaints are matched by keyword rather than exact label.
- **Memory-safe.** The multi-gigabyte 311 export is read in chunks of 4 columns and aggregated on the
  fly, so the raw rows are never all in memory.
- **Non-residential ZIPs flagged.** 254 restaurants (1.2%) sit in office-tower or airport ZIPs with
  little or no resident population.

Outputs: `cleaned_restaurant_inspections.csv` (21,472 rows × 24 columns, the primary analysis file) and
`cleaned_neighborhood_zip_level.csv` (217 ZIP codes × 14 columns).

## A first look

Lower scores mean fewer violation points. Medians are identical across boroughs, but Queens averages
the highest (worst) scores:

| Borough | Restaurants | Mean score | Median |
|---|---|---|---|
| Manhattan | 8,399 | 12.99 | 12 |
| Brooklyn | 5,515 | 12.88 | 12 |
| Queens | 4,825 | 14.25 | 12 |
| Bronx | 1,948 | 12.87 | 12 |
| Staten Island | 785 | 13.56 | 12 |

## Planned analysis

Designed and presented, not yet run:

1. **Borough comparison:** one-way ANOVA and Kruskal–Wallis tests (RQ1).
2. **Correlation analysis:** Pearson and Spearman correlations with a multicollinearity screen (RQ2–3).
3. **Multiple linear regression:** score on sanitation and demographics, with borough and cuisine
   controls and ZIP-clustered standard errors (RQ2–3).
4. **Logistic regression:** probability of an A grade, the outcome the public actually sees.
5. **Feature importance:** standardized coefficients compared with random-forest importance (RQ4).
6. **Geographic visualization:** maps of residuals to reveal spatial patterns the model misses.

**My part:** the preliminary data exploration, the planned statistical analysis, and the expected
contributions and limitations.

## Limitations

- ZIP Code Tabulation Areas approximate USPS ZIP codes; boundaries don't align perfectly.
- 311 volume partly reflects how readily residents report problems, not only true conditions.
- Observational, aggregated data: any finding is an association at the neighbourhood level, not a claim
  about individual restaurants (the ecological fallacy).

## Running it

Download the raw files listed at the top of the notebook into `data_raw/` (the raw data is not included
here: the 311 export alone is several gigabytes), then run `data_preparation.ipynb`. It needs only
`pandas` and `numpy`, and writes the cleaned files to `data_clean/`. The first cell mounts Google
Drive for Colab; to run locally, skip that cell and start from the folder that contains `data_raw/`.
