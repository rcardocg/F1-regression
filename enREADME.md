# How can I determine the outcome of a Formula 1 race?

Linear regression project on F1 race results ("Formula 1 World Championship 1950-2024" Kaggle dataset).

---

# Review of the report's justification and conclusions

> The "Metrics", "Justification", and "Conclusions" sections of the original PDF contain statements that **do not correspond to what was actually executed in the notebook**. Below is a breakdown for each section, detailing what the original text stated (BEFORE), why it is incorrect, and the proposed text to replace it (AFTER).

## Context: what the notebook actually did

The notebook trained **only two models**:

1. **Simple model** = `grid` + `total_pit_time` (2 predictor variables; 3 columns including the constant). It does not include `count_pit_stops`.
2. **Complex model** = `grid` + `total_pit_time` + 10 dummy variables (Red Bull, Ferrari, Mercedes, Sauber; Verstappen, Russell, Norris, Piastri, Magnussen, Sargeant) = 12 predictors (13 columns including the constant).

`count_pit_stops` **was only used in the p-values analysis (section 6.2); it was never included in any trained model.** Furthermore, the correlation between `total_pit_time` and `count_pit_stops` was never calculated.

| Metric (test) | Simple model (grid + total_pit_time) | Complex model (+ 10 dummies) |
|---|---|---|
| R² | 0.356 | 0.401 |
| Adjusted R² (train) | 0.361 | 0.414 |
| MAE | 3.921 | 3.740 |
| RMSE | 4.943 | 4.766 |
| Accuracy* | 0.830 | 0.837 |
| Precision** | 0.597 | 0.634 |

\*Accuracy defined in the notebook as `1 - MAE/range` (this is not classification accuracy).
\*\*Precision defined as the correlation coefficient between actual and predicted values (this is not classification precision). These definitions are non-standard and should not be interpreted as such.

## Metrics

**BEFORE (incorrect):** The PDF claims that *four* models were evaluated (2, 3, 12, and 13 variables) comparing the presence of `count_pit_stops`, citing an adjusted R² of **0.426** for the 3-variable model and **0.426/0.427** for the 13-variable model, with MAE 3.81/3.63, RMSE 4.83/4.63, precision 0.65, and accuracy 0.84. It also claims that `total_pit_time` and `count_pit_stops` are "highly correlated" and that removing either does not change the metrics.

**Why it is incorrect:** Such models and figures never existed. No model included `count_pit_stops`; the 0.426/0.427 R² values do not appear in any notebook output. The supposed correlation/redundancy between the two pit variables was never calculated or tested.

**AFTER (proposed):** Two models were trained and evaluated on train/val/test splits (see table). The complex model outperforms the simple one across all metrics: R² of 0.401 vs. 0.356 on test, MAE of 3.740 vs. 3.921, and RMSE of 4.766 vs. 4.943, with stable values between validation and testing (R² ~0.37-0.40), indicating a lack of significant overfitting. The claim that `total_pit_time` and `count_pit_stops` are redundant **is completely unfounded**: their correlation was not measured, nor was a model compared using one or the other. If this claim is to be maintained, a 3-variable model with `count_pit_stops` must be executed and verified (e.g., using VIF).

## Justification

**BEFORE (incorrect):** "The choice of the 3 and 13 variable models is justified because they include `count_pit_stops` and show better performance; the variables `total_pit_time` and `count_pit_stops` provide similar information, and it is key to include at least one."

**Why it is incorrect:** The central premise is false: the 3-variable model in the notebook (which actually consists of `grid` + `total_pit_time`) **does not include** `count_pit_stops`, and there is no "with vs. without" comparison of said variable. The justification is based on results that were not obtained.

**AFTER (proposed):**
*   The simple model is justified by the variables with the highest statistical significance and interpretability: `grid` (p ≈ 3.2×10⁻⁹⁰, the most influential, consistent with qualifying strongly predicting the outcome) and `total_pit_time` (p ≈ 4.9×10⁻²¹), both having a significant effect on the final position.
*   The complex model adds the driver and team dummies that were significant (p < 0.05) **and** active in recent seasons (≥ 2024): Red Bull, Ferrari, Mercedes, Sauber, and the drivers Verstappen, Russell, Norris, Piastri, Magnussen, and Sargeant.
*   `count_pit_stops` was significant during the p-value screening (p ≈ 3.4×10⁻⁴⁷), but it was not incorporated into any trained model; **there is no evidence in the notebook** that it is redundant with `total_pit_time`.
*   Interpretative caveat: the coefficient for `total_pit_time` is **negative** (−0.026/−0.025), meaning that more time in the pits leads to a better (lower) position. This is counterintuitive and suggests confounding or selection bias (e.g., drivers who DNF do not return to the pits, or pit stops are associated with the strategies/pace of top teams). This must be discussed with caution before attributing a causal effect to it.

## Conclusions

**BEFORE (incorrect):** "The 13-variable model improves upon the 3-variable model, but not enough to justify the complexity; the simple 3-variable model efficiently captures the relationship. `total_pit_time` and `count_pit_stops` describe the same phenomenon and eliminating either does not modify the results."

**Why it is incorrect:** The "3-variable model with `count_pit_stops`" was not executed, making the comparison and conclusion of redundancy unsustainable.

**AFTER (proposed):**
*   The complex model (12 predictors) does show a real, albeit modest, improvement over the simple one (2 predictors): R² 0.401 vs. 0.356 on test (approx. +12%), MAE 3.740 vs. 3.921 (−5%), and RMSE 4.766 vs. 4.943 (−4%). The gain exists but is limited.
*   `grid` is the dominant factor: by itself, it explains most of the predictive capacity, aligning with the reality of the sport (starting position is the best pre-race indicator).
*   The R² ≈ 0.40 implies that more than 60% of the variability remains unexplained. Relevant factors are missing (weather, incidents, strategy, reliability, driver/team form), meaning the model identifies **indicators** of the outcome, but does not allow for precise prediction.
*   The claim regarding the redundancy of `total_pit_time` and `count_pit_stops` **is not proven** in the work performed: their correlation was never calculated, nor were models compared that included them separately. Choosing `total_pit_time` over `count_pit_stops` should be based on additional analysis, not on an unmeasured correlation.
*   Recommendation: incorporate qualifying data, form streaks, weather, and mechanical status, and explicitly verify the contribution of the pit variables (e.g., with a 3-variable model and VIF) before claiming their redundancy.

## Note on the notebook

The notebook's "Simple Model (3 variables)" actually uses 2 predictors (`grid`, `total_pit_time`) plus the constant; the "Complex model (13 variables)" uses 12 predictors plus the constant. The 19 engineered features (e.g., `avg_pit_time`, `pit_time_log1p`, `grid_x_pits`, `pit_stops_per_10laps`) were built but **none were used in the final models**; they were only useful as an exploratory stage during variable screening.
