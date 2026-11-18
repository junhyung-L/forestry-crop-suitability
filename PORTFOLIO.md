# Forestry Crop Suitability Analysis: Matching Models to Crops and Conditions

[English](PORTFOLIO.md) | [한국어](PORTFOLIO.ko.md) · [README](README.md)

## Summary

Prospective foresters must choose a crop and a site first, but soil, climate,
and production data sit in separate public sources, and existing suitability
datasets are hard to read without coding. The project combines those sources,
including GIS soil shapefiles, for seven short-rotation forest crops, screens
variables with statistical tests, compares three classifiers per crop, and
turns the result into a map-based recommendation concept. Grand Prize (Korea
Forest Service Commissioner Award, first place), 2024 Forestry Statistics Smart
Use Competition, Korea Forest Service and Korea Forestry Promotion Institute
(13 Dec 2024). Team entry.

## Challenge

- **Mixed data types.** Soil data is categorical (texture code, depth type,
  available-water class); climate data is continuous. One correlation matrix
  cannot screen both.
- **Small, crop-specific samples.** Production data is split across seven crops
  and many regions; the retained ROC curves imply about 2 to 5 positive cases
  per test set.
- **A user who does not read models.** The output is a GeoPandas heatmap with
  suitable / moderate / unsuitable grades that narrows sites for field review.

## Decisions

**1. Screen variables first, or feed every column to the models.**
Choice: screen soil variables with chi-square and ANOVA (p < 0.1) and climate
variables by correlation with production (|r| >= 0.2), with Levene's test as an
assumption check. Why: with small samples, fewer candidates keep each model
explainable variable by variable. Cost: loose thresholds and many tests, with
no multiple-testing correction, raise the chance of keeping noise variables.

**2. One algorithm for all crops, or the best one per crop.**
Choice: compare SVM, random forest, and logistic regression per crop. The
report picked SVM for yam; random forest for blackberry, fresh shiitake,
deodeok, and acanthopanax; logistic regression for chestnut and balloon flower.
Why: crops differ in which variables matter. Cost: with 2 to 5 positive test
cases, model differences sit within AUC noise, so the choice is fragile.

**3. Classify "suitable", or predict yield directly.**
Choice: label each row as at or above the crop mean, keeping regression as a
separate path. Why: a grower's first question is yes / no. Cost: "above the
mean" is a statistical cut-off, not an agronomic threshold, and depends on
which regions are in the data.

**4. Check results against domain knowledge.** Top variables were compared
with cultivation guidance; for chestnut, the leading soil variable was the
loam texture code, which matches standard advice. A sanity check, not
validation.

## Results

| Crop | Model chosen in the report | ROC-image AUC | Approx. positives in test set |
|---|---|---:|---:|
| Chestnut | Logistic regression | 0.94 | 5 |
| Yam | SVM | 0.91 | 2 |
| Blackberry | Random forest | 0.84 | 4 |

The comparison point is the chance diagonal (AUC 0.5) on each figure.

## Limitations

- AUCs come from very small test sets and are not rerun here.
- AUCs for the other four crops and the per-model comparison table are not retained.
- No spatial or temporal hold-out; nearby regions may appear in both train and test sets.
- Correlations and importances show association, not that changing a condition changes yield.
- The recommendation map is a design concept; no deployed service, and the grades do not predict cultivation success.

## Next steps

- Rebuild the analysis table with a versioned data dictionary.
- Evaluate with leave-one-region-out splits, report AUC with bootstrap
  intervals, and add majority-class and single-variable baselines.
- Replace the above-mean label with an agronomic threshold agreed with a domain
  expert, then compare the map against known cultivation sites.

## Evidence

- Final report and presentation (Korean): `reports/`
- ROC figures: `images/roc_chestnut.png`, `images/roc_yam.png`,
  `images/roc_blackberry.png`
- [Project review](docs/PROJECT_REVIEW.md)
