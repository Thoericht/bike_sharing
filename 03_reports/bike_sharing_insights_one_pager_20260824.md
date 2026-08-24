# Objective

The objective of this project is to predict the hourly number of bike rentals (`cnt`) using the UCI Bike Sharing Dataset. The analysis includes exploratory data analysis (EDA), feature engineering and model comparison to identify a suitable model for a production forecasting service.

# Exploratory Data Analysis

The dataset contains **17,379 hourly observations** from 2011–2012.

The EDA identified three main patterns:

- Bike rental demand is strongly driven by **temporal patterns**, especially the hour of the day, with clear morning and evening demand peaks on working days.

- Rental demand shows strong **seasonal and calendar effects**, including differences between weekdays, weekends, and working days.

- The target variable is **right-skewed**, with a limited number of hours showing exceptionally high rental demand.

Based on these findings, additional temporal and historical features were engineered, including:
- `time_of_day` to capture broader daily demand phases,
- lag features (`lag_1`, `lag_24`, `lag_168`) to capture short-term, daily and weekly dependencies,
- rolling statistics (`rolling_24h`, `rolling_168h`) to represent recent demand trends,
- `weekday_hour_avg` to capture recurring hourly patterns by weekday.

A feature derived from registered users was excluded because it introduced **target leakage**.

# Model Comparison

Three models were evaluated using a time-based train-test split and TimeSeriesSplit cross-validation.

| Model | R² | MAE | RMSE |
|---|---:|---:|---:|
| Random Forest | 0.9547 | 28.99 | 46.91 |
| Linear Regression | 0.9033 | 47.99 | 68.56 |
| **XGBoost** | **0.9575** | **29.12** | **45.48** |

# Selected Model

**XGBoost** was selected as the final model because it achieved the best overall predictive performance, particularly in terms of RMSE and explained variance.

The model effectively captures non-linear relationships and interactions between temporal, calendar and historical demand features.

Feature importance analysis shows that the dominant predictors are:

- `lag_1`, representing the previous hour's demand,
- `time_of_day` and `hr`, representing daily demand patterns,
- `weekday_hour_avg`, capturing recurring demand behaviour by weekday and hour,
- `rolling_168h`, representing weekly demand trends.

This confirms the EDA findings that bike rental demand is primarily driven by temporal patterns, while weather variables provide additional but comparatively smaller predictive value.

**Final performance:**

- **R²:** 0.957
- **MAE:** 29.12 rentals/hour
- **RMSE:** 45.48 rentals/hour

# Production Considerations

The implementation uses reusable scikit-learn pipelines, consistent preprocessing, time-based validation and modular evaluation functions to support maintainability and reproducibility.

For a production forecasting service, further improvements could include:
- automated hyperparameter optimization,
- continuous model monitoring,
- drift detection for changing demand patterns,
- retraining pipelines with newly collected rental data.