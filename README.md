# Bike Sharing Demand Forecasting

## Project Overview

This project focuses on predicting the hourly number of bike rentals (`cnt`) using the UCI Bike Sharing Dataset. The workflow includes exploratory data analysis (EDA), feature engineering, model development, and evaluation of different machine learning approaches.

The goal is to develop a reliable forecasting model as a foundation for a production-oriented demand prediction service.

[View the executive summary presentation (PDF)](https://github.com/Thoericht/bike_sharing/blob/main/03_reports/bike_sharing_forecast_presentation_20260828.pdf)

## Dataset

This project uses the Bike Sharing Dataset published by Fanaee-T, H. (2013) through the UCI Machine Learning Repository. The dataset contains **17,379 hourly observations** from 2011–2012, including rental counts, weather conditions, and calendar-related variables.

Citation:  
Fanaee-T, H. (2013). *Bike Sharing [Dataset].* UCI Machine Learning Repository.  
https://doi.org/10.24432/C5W894

The target variable is:

- `cnt`: total number of bike rentals per hour

The data includes temporal, calendar, weather, and seasonal information.

## Feature Engineering

Additional features were created to improve forecasting performance:

- `time_of_day` – captures daily demand phases (e.g., morning/evening peaks)
- `lag_1`, `lag_24`, `lag_168` – historical demand dependencies
- `rolling_24h`, `rolling_168h` – recent demand trends
- `weekday_hour_avg` – recurring demand patterns by weekday and hour

Features causing target leakage were removed to ensure realistic evaluation.

## Models Evaluated

The following models were compared:

- Linear Regression
- Random Forest
- XGBoost

Evaluation was performed using a time-based train-test split and time-series validation.

## Model Comparison

The models were evaluated on the same time-based test set using R², MAE, and RMSE.

| Model | R² | MAE (rentals/hour) | RMSE (rentals/hour) |
|---|---:|---:|---:|
| XGBoost | 0.957 | 29.12 | 45.48 |
| Random Forest | 0.955 | 28.90 | 46.83 |
| Linear Regression | 0.903 | 47.99 | 68.56 |

XGBoost achieved the highest R² and the lowest RMSE, making it the strongest overall model for this forecasting task. Random Forest produced the lowest MAE, indicating slightly better average absolute accuracy, but its larger RMSE suggests that it made more substantial errors on some observations.

The final model selected for the forecasting service is **XGBoost**.

## Time-Series Validation

To assess model stability across different time periods, a five-fold time-series cross-validation was performed. The fold-level R² values ranged from 0.665 to 0.934, with lower performance in the earliest validation period and more stable results in later periods.

This indicates that the model performs well overall but that forecast accuracy varies across different temporal segments.

## Key Findings

The analysis shows that bike rental demand is mainly driven by:

- previous demand levels (`lag_1`)
- hour of the day
- time-of-day patterns
- weekday-specific demand behavior

Temporal features provide the strongest predictive power, while weather variables contribute additional information.

## Project Structure

├── data/ # Dataset files

├── notebooks/ # EDA and modelling notebooks

├── reports/ # Presentation and 1 pager

└── README.md

## Technologies

- Python
- Pandas
- NumPy
- Scikit-learn
- XGBoost
- Matplotlib / Seaborn

## Future Improvements

Possible extensions for a production system:

- Automated hyperparameter optimization
- Model monitoring and drift detection
- Automated retraining pipeline
- Deployment as a forecasting API
