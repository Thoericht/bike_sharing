# Model Card: XGBoost Optuna

> **Status:** Final regression model 
> **Created:** 2026-08-24  
> **Model type:** XGBoost Regressor with Optuna hyperparameter optimization  
> **Primary target:** `cnt` – number of bike rentals per hour

## 1. Short description

The model predicts the hourly number of bike rentals (`cnt`). It is based on an XGBoost Regressor and uses temporal, meteorological, and lag/rolling features. Hyperparameters were optimized with Optuna.

The model was selected as the final model because XGBoost achieved the best performance among the compared models in the time-based hold-out test. Modeling was implemented in a pipeline with preprocessing and the regressor.

## 2. Intended use

### Intended use cases

- Forecasting hourly demand for bike rentals.
- Supporting capacity, staffing, and availability planning.
- Analyzing the most important drivers of demand.
- Benchmarking against simpler baseline and reference models.

### Out-of-scope use cases

- No causal claims about the effect of individual variables.
- No automated control without domain plausibility checks.
- No extrapolation far beyond the time period or value range observed in the dataset.
- No use as the sole decision basis during exceptional events, structural market changes, or missing input data.

## 3. Dataset

The model is trained on the **UCI Bike Sharing Dataset** (Fanaee-T, 2013), which contains hourly and daily counts of rental bikes from the Capital Bikeshare system in Washington, D.C., for the years 2011 and 2012, together with corresponding weather and seasonal information. citeweb:2

**Dataset characteristics:**

- **Period:** 2011–2012  
- **Granularity:** hourly (used here) and daily  
- **Instances:** 17,389 total records (hourly + daily)  
- **Features:** 13 core variables in the raw files  
- **Missing values:** none reported in the original dataset citeweb:2

**Key variables used:**

- `instant`: record index  
- `dteday`: date  
- `season`: categorical (1:winter, 2:spring, 3:summer, 4:fall)  
- `yr`: year (0: 2011, 1: 2012)  
- `mnth`: month (1–12)  
- `hr`: hour (0–23)  
- `holiday`: binary (holiday vs. non-holiday, derived from DC holiday schedule)  
- `weekday`: day of the week  
- `workingday`: binary (1 if neither weekend nor holiday, else 0)  
- `weathersit`: categorical weather situation (1–4, from clear to severe weather)  
- `temp`: normalized temperature in Celsius  
- `atemp`: normalized feeling temperature in Celsius  
- `hum`: normalized humidity  
- `windspeed`: normalized wind speed  
- `casual`: count of casual users  
- `registered`: count of registered users  
- `cnt`: total count of rentals (target variable) citeweb:2

The raw hourly data (`hour.csv`) were preprocessed to create additional time-based and lag/rolling features (e.g., `lag_1`, `lag_24`, `lag_168`, `rolling_24h`, `rolling_168h`, `weekday_hour_avg`, `trend_diff`) and then split chronologically into a training set (2011-01-01 to 2012-08-06, 13,891 rows) and a hold-out test set (2012-08-07 to 2012-12-31, 3,488 rows). This time-based split ensures that no future information leaks into the training process and reflects a realistic deployment scenario where the model predicts future demand based on past observations. citefile:1

**License:** The dataset is licensed under a Creative Commons Attribution 4.0 International (CC BY 4.0) license, allowing sharing and adaptation for any purpose with appropriate credit. citeweb:2

## 4. Data

### Dataset and time period

The processed data contain hourly observations. The data were sorted chronologically and split into a time-based training and test set:

| Split | Time period | Size |
|---|---:|---:|
| Train | 2011-01-01 to 2012-08-06 | 13,891 rows |
| Test | 2012-08-07 to 2012-12-31 | 3,488 rows |

The time-based split prevents later observations from being used in training to predict earlier time points.

### Target variable

- **Name:** `cnt`
- **Type:** numeric, continuous
- **Unit:** number of rentals per hour
- **Task:** regression

### Input features

The final feature set `FEATURES_V01` includes:

- Calendar and time features: `season`, `yr`, `mnth`, `hr`, `time_of_day`, `holiday`, `weekday`, `workingday`.
- Weather features: `weathersit`, `temp`, `temp_diff`, `hum`, `windspeed`.
- Lagged target information: `lag_1`, `lag_24`, `lag_168`.
- Rolling / aggregated features: `rolling_24h`, `rolling_168h`, `weekday_hour_avg`, `trend_diff`.

The lagged and rolling features are intended to capture short-term, daily, and weekly demand patterns. When computing them, it must be ensured that only information available at prediction time is used.

## 5. Preprocessing

Preprocessing is part of an `sklearn` pipeline:

| Feature type | Processing |
|---|---|
| Categorical features | One-hot encoding with `handle_unknown='ignore'` |
| Binary features | Imputation with most frequent value |
| Numerical features | Imputation with median |
| Model | XGBoost Regressor |

Imputation is performed inside the pipeline to avoid leakage from the test set into training.

## 6. Model architecture

The final model consists of:

1. Loading the engineered features.
2. Type-specific preprocessing via a `ColumnTransformer`.
3. XGBoost Regressor to model nonlinear relationships and interactions.
4. Output of a numeric prediction for `cnt`.

### Final hyperparameters

Optuna ran 60 trials. The best result was obtained in trial 41 with a cross-validation RMSE of **50.604**.

| Hyperparameter | Value |
|---|---:|
| `learning_rate` | 0.0344179611 |
| `max_depth` | 3 |
| `min_child_weight` | 6 |
| `subsample` | 0.9327237907 |
| `colsample_bytree` | 0.7421855502 |
| `reg_alpha` | 2.2742507754 |
| `reg_lambda` | 5.3217869666 |
| `n_estimators` | 1,353 |

Optimization used `TimeSeriesSplit(n_splits=5)` with RMSE as the optimization objective. The test set was not used for hyperparameter selection.

## 7. Evaluation

### Time-based hold-out test

Final evaluation was performed on the chronologically later test period from 2012-08-07 to 2012-12-31.

| Metric | Value | Interpretation |
|---|---:|---|
| RÂ² | 0.957450 | Proportion of variance explained in the test period |
| MAE | 29.120710 | Average absolute error in rentals per hour |
| RMSE | 45.482281 | Error metric with stronger weighting of large deviations |

XGBoost achieved the lowest RMSE and highest RÂ² among the feature-engineered models:

| Model | RÂ² | MAE | RMSE |
|---|---:|---:|---:|
| XGBoost | 0.957450 | 29.120710 | 45.482281 |
| Random Forest | 0.954892 | 28.902847 | 46.829660 |
| Linear Regression | 0.903318 | 47.994169 | 68.559353 |

### Time-series cross-validation

The five cross-validation folds show varying model performance over time:

| Fold | RÂ² | MAE | RMSE |
|---:|---:|---:|---:|
| 1 | 0.665 | 60.071 | 86.353 |
| 2 | 0.928 | 25.395 | 39.563 |
| 3 | 0.906 | 23.871 | 36.368 |
| 4 | 0.869 | 39.694 | 67.143 |
| 5 | 0.934 | 37.671 | 55.674 |
| **Mean** | **0.860** | **37.340** | **57.020** |

The clearly weaker performance in the first fold indicates that predictive performance changes over time. Model performance and potential data or concept drift should therefore be monitored regularly.

## 8. Interpretation

XGBoost can model complex nonlinear relationships between time of day, weekday, weather, lag features, and demand. The high test performance should not be interpreted as evidence of causal relationships.

The feature importance computed in the notebook was shown in grouped form. For communicating results, a distinction should be made between:

- **Global model interpretation:** Which feature groups are important for predictions overall?
- **Local explanation:** Why does a specific hour receive a particular forecast?
- **Causality:** Which variable would cause a change in demand? This question is not answered by feature importance or SHAP alone.

When interpreting lag and rolling features, it is especially important to check whether they might contain information that would not be available in time in later production use.

## 9. Strengths

- Very good performance in the time-based hold-out test.
- Models nonlinearities and interactions.
- Accounts for short-term, daily, and weekly demand patterns.
- Hyperparameter optimization with time-aware cross-validation.
- Preprocessing and model are combined reproducibly in a pipeline.
- Better than Random Forest and linear regression in the test period.

## 10. Limitations and risks

- Test performance refers to a specific historical period and may degrade under drift.
- The first cross-validation fold shows clearly higher errors than later folds.
- RMSE and MAE are not necessarily equally informative across all demand ranges.
- Extreme values and exceptional events can contribute disproportionately to RMSE.
- One-hot encoding can lead to limited representation for new categories, even if unknown categories are technically ignored.
- Lagged or rolling features can lead to erroneous predictions if not updated in time.
- Feature importance is not a causal analysis.
- The model card does not include a separate uncertainty or prediction interval assessment.

## 11. Fairness and impact

The model forecasts aggregated demand and does not make person-level decisions. Classic fairness metrics for individual or group discrimination are therefore only partially applicable.

Possible indirect impacts should still be checked, for example if forecasts lead to systematic under-supply of certain neighborhoods, time windows, or user groups. Evaluation should be performed along relevant operational segments, such as:

- Time of day.
- Day of week.
- Weather condition.
- Season.
- Location or area, if such information is used later.

## 12. Reproducibility

For reproducible execution, at least the following information should be versioned:

- Raw data and processing steps.
- Feature engineering code.
- Training and test time periods.
- Python, scikit-learn, XGBoost, and Optuna versions.
- Random seed, if used.
- Optuna study and trial results.
- Final pipeline or model artifact.
- Evaluation code and metrics.

The model development visible in the notebook was executed on 2026-08-24. The specific XGBoost package version and a serialized model artifact are not documented in the available notebook information.

## 13. Monitoring in production

Recommended monitoring metrics:

- RMSE and MAE on newly arriving labeled data.
- RÂ² over rolling time windows.
- Errors broken down by hour, weekday, season, and weather condition.
- Distribution of input features compared to the training dataset.
- Share of missing values and unknown categories.
- Availability and plausibility of lag and rolling features.
- Share of exceptionally large absolute errors.

Re-validation or retraining should be triggered if error metrics deteriorate persistently, feature distributions drift significantly, or data generation or demand behavior changes structurally.

## 14. Open items before production use

- Add XGBoost and Python versions.
- Supplement the exact feature importance table instead of only referencing the figure.
- Document SHAP analysis for global and local explanations.
- Add prediction intervals or uncertainty estimates.
- Formally document leakage checks for `weekday_hour_avg`, rolling, and lag features.
- Add tests for missing values, new categories, and late-arriving data.
- Version the model artifact and reference it with a hash or model ID.
- Define monitoring thresholds and retraining processes.

## 15. Source basis

This model card was created from the provided notebook `03_modelling.ipynb` and the UCI Bike Sharing Dataset description. The data splits, features, model comparisons, cross-validation results, and Optuna trials documented there form the basis of the statements. citefile:1citeweb:2

## 16. Citation

When using or referencing this dataset, please cite:

Fanaee-T, H. (2013). Bike Sharing [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C5W894 citeweb:2