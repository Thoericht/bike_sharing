# Bike Sharing Demand Forecasting

## Project Overview

This project focuses on predicting the hourly number of bike rentals (`cnt`) using the UCI Bike Sharing Dataset. The workflow includes exploratory data analysis (EDA), feature engineering, model development, and evaluation of different machine learning approaches.

The goal is to build a reliable forecasting model suitable for a production-oriented demand prediction service.

## Dataset

The dataset contains **17,379 hourly observations** from 2011–2012.

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

## Final Model Performance

The best performing model was **XGBoost**.

Performance on the test set:

| Metric | Score |
|---|---:|
| R² | 0.957 |
| MAE | 29.12 rentals/hour |
| RMSE | 45.48 rentals/hour |

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