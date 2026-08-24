# Objective

The objective of this project is to predict the hourly number of bike rentals (`cnt`) using the UCI Bike Sharing Dataset. The analysis includes exploratory data analysis (EDA), feature engineering, hyperparameter optimization, and model comparison to identify a suitable model for a production forecasting service.

# Exploratory Data Analysis

The dataset contains **17,379 hourly observations** collected between 2011 and 2012.

The EDA identified three main patterns:

- Bike rental demand is strongly driven by **temporal patterns**, especially the hour of the day, with clear morning and evening demand peaks on working days.
- Rental demand shows strong **seasonal and calendar effects**, including differences between weekdays, weekends, and working days.
- The target variable is **right-skewed**, with a limited number of hours showing exceptionally high rental demand.

Based on these findings, additional temporal and historical features were engineered:

- `time_of_day` to capture broader daily demand phases,
- lag features (`lag_1`, `lag_24`, `lag_168`) to capture short-term, daily, and weekly dependencies,
- rolling statistics (`rolling_24h`, `rolling_168h`) to represent recent demand trends,
- `weekday_hour_avg` to capture recurring hourly demand patterns by weekday,
- `trend_diff` to capture short-term changes in demand.

A feature derived from registered users was excluded because it introduced **target leakage**.

# Model Comparison

Three models were evaluated using **TimeSeriesSplit cross-validation** with a time-based validation strategy.

The cross-validation results were used for model selection. XGBoost hyperparameters were optimized using **Optuna** to identify the best-performing parameter configuration.

## Cross-Validation Results

| Model | Feature Set | R² | MAE | RMSE |
|-------|-------------|---:|---:|---:|
| Linear Regression | V01 | 0.876 | 39.7 | 56.5 |
| Random Forest | V01 | 0.861 | 36.2 | 57.2 |
| **XGBoost (Optuna CV)** | **V01** | **0.891** | **33.0** | **50.6** |

The cross-validation results show that **XGBoost achieved the best overall performance**. It provided the highest explained variance and the lowest prediction error, demonstrating that the model can effectively capture nonlinear relationships and complex temporal dependencies.

Based on these results, XGBoost was selected as the final forecasting model.

## Final Test Set Evaluation

The selected XGBoost model was evaluated on an unseen time-based test set to assess its generalization capability.

| Metric | Value |
|--------|------:|
| R² | 0.9575 |
| MAE | 29.12 rentals/hour |
| RMSE | 45.48 rentals/hour |

The final evaluation confirms that the optimized XGBoost model generalizes well to unseen future observations.

# Selected Model

**XGBoost** was selected as the final model based on its superior cross-validation performance after Optuna-based hyperparameter optimization.

The model effectively captures nonlinear relationships and interactions between temporal, calendar, historical demand, and weather-related features.

Feature importance analysis shows that prediction performance is mainly driven by historical demand and temporal information:

| Feature | Importance |
|---------|-----------:|
| `lag_1` | 30.75% |
| `time_of_day` | 14.12% |
| `weekday_hour_avg` | 9.92% |
| `hr` | 8.63% |
| `lag_168` | 8.21% |
| `lag_24` | 5.69% |
| `weathersit` | 5.07% |
| `rolling_24h` | 4.32% |
| `rolling_168h` | 3.47% |

Additional calendar features such as `weekday`, `workingday`, `holiday`, `season`, `month`, and `year` provide further predictive information.

Weather variables including temperature, humidity, wind speed, and temperature changes contribute less compared to historical and temporal features.

These results confirm the EDA findings that hourly bike rental demand is primarily driven by temporal patterns and previous rental behaviour, while weather conditions provide additional but comparatively smaller predictive value.

# Final Performance

- **R²:** 0.9575
- **MAE:** 29.12 rentals/hour
- **RMSE:** 45.48 rentals/hour

# Production Considerations

The implementation uses reusable scikit-learn pipelines, consistent preprocessing, optimized hyperparameters, time-based validation, and modular evaluation functions to ensure maintainability and reproducibility.

Potential improvements for a production forecasting service include:

- continuous model monitoring,
- concept and data drift detection,
- scheduled retraining using newly collected rental data,
- periodic hyperparameter re-optimization as demand patterns evolve,
- monitoring prediction quality over time through production metrics and alerting.