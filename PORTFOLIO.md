# Forestry Crop Suitability and Yield Analysis

[English](PORTFOLIO.md) | [한국어](PORTFOLIO.ko.md) · [README](README.md)

## At a glance

Suitability analysis and a map-based service concept for seven short-cycle
non-timber forest crops. New forest farmers must choose a crop and a site
first, but soil, climate, and production data sit in separate public sources,
and existing suitability datasets are hard to read without coding. The project
combines those sources, including GIS soil shapefiles, screens variables with
statistical tests, compares three classifiers per crop, and turns the result
into a map concept. Test AUC for above-mean production: 0.94 (chestnut), 0.91
(yam), 0.84 (blackberry), against 0.5 for chance.

Grand Prize (Korea Forest Service Commissioner Award, first place), 2024
Forestry Statistics Smart Use Competition, Korea Forest Service and Korea
Forestry Promotion Institute (13 Dec 2024). Team entry.

What made it hard:

- **Mixed data types.** Soil data is categorical (texture code, depth type,
  available-water class); climate data is continuous. One correlation matrix
  cannot screen both.
- **Small, crop-specific samples.** Production data is split across seven crops
  and many regions; the ROC curves imply about 2 to 5 positive cases per test
  set.
- **End users who do not interpret models.** The output is a GeoPandas heatmap
  with suitable / moderate / unsuitable grades that narrows sites for field
  review.

## Key decisions

**1. How many variables go into the models.**
Options: feed every column, or screen variables first.
Choice: screen soil variables with chi-square and ANOVA (p < 0.1) and climate
variables by correlation with production (|r| >= 0.2), with Levene's test as an
assumption check.
Why: with small samples, fewer candidates keep each model explainable variable
by variable.
Cost: loose thresholds and many tests, with no multiple-testing correction,
raise the chance of keeping noise variables.

**2. Which algorithm per crop.**
Options: one algorithm for all crops, or the best one per crop.
Choice: compare SVM, random forest, and logistic regression per crop. SVM was
chosen for yam; random forest for blackberry, fresh shiitake, deodeok, and
acanthopanax; logistic regression for chestnut and balloon flower.
Why: crops differ in which variables matter.
Cost: with 2 to 5 positive test cases, model differences sit within AUC noise,
so the choice is fragile.

**3. What the target is.**
Options: predict yield directly, or classify "suitable".
Choice: label each row as at or above the crop mean, keeping regression as a
separate path.
Why: a grower's first question is yes or no.
Cost: "above the mean" is a statistical cut-off, not an agronomic threshold,
and depends on which regions are in the data.

**4. How to sanity-check the variables.**
Options: trust the metrics alone, or compare top variables with cultivation
guidance.
Choice: compare with cultivation guidance. For chestnut, the leading soil
variable was the loam texture code, which matches standard advice.
Why: a quick check that the top variables make agronomic sense.
Cost: a sanity check, not validation.

## Results

| Crop | Chosen model | Test AUC | Approx. positives in test set |
|---|---|---:|---:|
| Chestnut | Logistic regression | 0.94 | 5 |
| Yam | SVM | 0.91 | 2 |
| Blackberry | Random forest | 0.84 | 4 |

The comparison point is the chance diagonal (AUC 0.5) on each ROC figure.

## Limitations

- AUCs come from test sets of about 2 to 5 positive cases, so per-crop model differences are within noise.
- AUCs survive for only three of the seven crops, with no per-model comparison table.
- No spatial or temporal holdout; nearby regions may appear in both train and test sets.
- "Suitable" is a statistical cut-off, and the map grades show association, not a prediction of cultivation success.

## Next steps

- Rebuild the analysis table with a versioned data dictionary.
- Evaluate with leave-one-region-out splits, report AUC with bootstrap
  intervals, and add majority-class and single-variable baselines.
- Replace the above-mean label with an agronomic threshold agreed with a domain
  expert, then compare the map against known cultivation sites.

Final report and presentation (Korean): `reports/`. ROC figures:
`images/roc_chestnut.png`, `images/roc_yam.png`, `images/roc_blackberry.png`.
