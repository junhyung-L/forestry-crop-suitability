# Forestry Crop Suitability Analysis: Matching Models to Crops and Conditions

[English](PORTFOLIO.md) | [한국어](PORTFOLIO.ko.md) · [README](README.md)

## Summary

Prospective foresters have to choose a crop and a site before anything else,
but soil, climate, and production data sit in separate public sources, and the
existing suitability datasets are hard to read without coding. This project
combined those sources for seven short-rotation forest crops and screened
variables with statistical tests. It then compared three classifiers per crop
and turned the result into a map-based recommendation concept. It received the
Grand Prize (Korea Forest Service Commissioner Award, first place) at the 2024
Forestry Statistics Smart Use Competition, Korea Forest Service and Korea
Forestry Promotion Institute (13 Dec 2024).

## Challenge

- **Mixed data types.** Soil data is categorical (texture code, depth type,
  available-water class), while climate data is continuous. One correlation
  matrix cannot screen both.
- **Small, crop-specific samples.** Production data is split across seven
  crops and many regions. The retained ROC curves imply only about 2 to 5
  positive cases per test set.
- **A user who does not read models.** The output had to be something a
  non-specialist could act on: a GeoPandas heatmap with suitable / moderate /
  unsuitable grades that narrows sites for field review.

## My contribution

This was a team competition entry. The data-source note in `data/` carries my
name, but this repository does not record the per-person split of the analysis,
so the decisions below are described as the team's.

## Decisions

**1. Screen variables before modelling, or feed every column to the models.**
Options: use all soil and climate columns, or filter them first. The report
filtered soil variables with chi-square and ANOVA (p < 0.1) and climate
variables by correlation with production (|r| >= 0.2), with Levene's test as an
assumption check. With small samples, fewer candidate variables make each
model easier to explain variable by variable. The cost: loose thresholds and
many tests raise the chance of keeping noise variables, and no multiple-testing
correction was applied.

**2. One algorithm for all crops, or the best one per crop.**
Options: pick one model family and apply it everywhere, or compare SVM, random
forest, and logistic regression for each crop. The report chose per crop: SVM
for yam; random forest for blackberry, fresh shiitake, deodeok, and
acanthopanax; logistic regression for chestnut and balloon flower. Crops differ
in which variables matter, so one model family need not suit all of them. The cost: with 2 to 5
positive test cases, the differences between models are within the noise of
the AUC estimate, so the per-crop choice is fragile.

**3. Define "suitable" as a class, or predict yield directly.**
Options: regress production, or label each row as at or above the crop mean.
The classification path uses the mean-production label because a grower's
first question is yes / no, and regression is kept as a separate path. The
cost: "above the mean" is a statistical cut-off, not an agronomic threshold,
and it depends on which regions are in the data.

**4. Check results against domain knowledge.** The top variables were compared
with cultivation guidance. For chestnut, the leading soil variable was the loam
texture code, which matches standard advice. This is a sanity check, not
validation.

## Results

| Crop | Model chosen in the report | Retained ROC-image AUC | Approx. positives in test set |
|---|---|---:|---:|
| Chestnut | Logistic regression | 0.94 | 5 |
| Yam | SVM | 0.91 | 2 |
| Blackberry | Random forest | 0.84 | 4 |

The comparison point is the chance diagonal (AUC 0.5) on each figure. AUCs for
the other four crops and the per-model comparison table are not retained.

## What the result does not show

- The AUCs come from very small test sets and are not rerun here; they are
  historical artefacts, not benchmarks.
- No spatial or temporal hold-out: nearby regions may appear in both train and
  test sets.
- Correlations and model importances describe association. They do not show
  that changing a soil or climate condition changes yield.
- The recommendation map is a design concept. No deployed service is verified,
  and the grades do not predict cultivation success.

## What I would do next

- Rebuild the analysis table with a data dictionary and keep it versioned.
- Evaluate with leave-one-region-out splits and report AUC with bootstrap
  intervals, plus a majority-class and single-variable baseline.
- Replace the above-mean label with an agronomic threshold agreed with a
  domain expert, then compare the map against known cultivation sites.

## Evidence

- Final report and presentation (Korean): `reports/`
- ROC figures: `images/roc_chestnut.png`, `images/roc_yam.png`,
  `images/roc_blackberry.png`
- [Project review](docs/PROJECT_REVIEW.md)
