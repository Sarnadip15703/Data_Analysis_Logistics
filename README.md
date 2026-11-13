# Logistics Delivery Time Prediction and Optimization

A beginner-friendly data science project that predicts how long a delivery will take, and uses the model to set better delivery promises and plan resources. Built in Python as part of a Logistics Data Analyst internship (Week 4).

## Problem

A logistics company promises delivery times to customers, but the promised time is almost unrelated to the real delivery time (correlation 0.04). For example, express deliveries are promised in 4.5 hours but take 6.2 hours on average, so 73.8% of them are late.

This project:
1. Predicts the delivery time in hours.
2. Uses the prediction to create better promises (optimization).

## Dataset

- **Source:** [Delivery Logistics Dataset (India - Multi-Partner)](https://www.kaggle.com/datasets/kundanbedmutha/delivery-logistics-dataset-india-multi-partner) by kundanbedmutha on Kaggle
- **Size:** 25,000 deliveries, 14 columns after cleaning
- **Cleaning (done in Week 2):** fixed the time columns (text to numbers), removed the unusable `delivery_id` column, checked for missing values, duplicates, and outliers
- **Cleaned file used here:** `Delivery_Logistics_Cleaned.csv`

| Role | Columns |
|---|---|
| Target (what is predicted) | `delivery_time_hours` |
| Features (what is used) | distance, package weight, delivery mode, weather, vehicle type, region, delivery partner, package type |
| Left out on purpose | `expected_time_hours` (the promise itself), `delayed`, `delivery_status`, `delivery_rating` (only known after delivery), `delivery_cost` (almost fully decided by distance) |

## Project Files

Rename these to match your own files.

```
.
├── README.md
├── Delivery_Logistics_Cleaned.csv       # cleaned dataset from Week 2
├── week4_delivery_time_prediction.ipynb # notebook with all the steps
└── Week4_Report.docx                    # written report
```

## How to Run

1. Install Python 3.9 or newer and Jupyter Notebook.
2. Install the libraries:

```bash
pip install pandas scikit-learn matplotlib jupyter
```

3. Put `Delivery_Logistics_Cleaned.csv` in the same folder as the notebook.
4. Open the notebook and run the cells from top to bottom:

```bash
jupyter notebook
```

## What the Notebook Does

| Step | What happens | Main tool |
|---|---|---|
| 1. Prepare data | Choose target and features, turn text columns into 0/1 columns, keep 20% of the data hidden for testing | `pandas`, `train_test_split` |
| 2. Train and compare models | Baseline (always the average), linear regression, decision tree, random forest | `scikit-learn` |
| 3. Cross-validation | Test each model on 5 different splits to check the results are stable | `cross_validate` |
| 4. Hyperparameter tuning | Try 12 settings of the decision tree and keep the best | `GridSearchCV` |
| 5. Feature importance | Find which inputs matter most | `RandomForestRegressor` |
| 6. Optimization | New promise = predicted time + safety buffer | `LinearRegression` |
| 7. Factor effects | Read the hours added by distance and weather from the model | model coefficients |

## Results

**Model comparison (5,000 test deliveries)**

| Model | MAE (hours) | RMSE (hours) | R2 |
|---|---|---|---|
| Baseline (always the average) | 2.51 | 3.11 | -0.00 |
| Linear regression | 1.28 | 1.62 | 0.73 |
| Decision tree | 1.30 | 1.65 | 0.72 |
| Random forest | 1.29 | 1.63 | 0.72 |

- **MAE:** the average mistake in hours.
- **RMSE:** like MAE, but big mistakes count more.
- **R2:** the share of the variation in delivery time the model explains (1.0 is perfect).

Cross-validation gave almost the same results (R2 0.726 to 0.735, spread about 0.005), so the results are stable. Tuning improved the decision tree by only about 0.01 hours of MAE. Linear regression is the best choice: as accurate as the others, faster, and easier to explain.

**What drives delivery time**

| Input | Importance |
|---|---|
| distance_km | 0.611 |
| weather_condition_stormy | 0.170 |
| weather_condition_rainy | 0.113 |
| weather_condition_foggy | 0.045 |
| package_weight_kg | 0.021 |

| Factor | Extra delivery time |
|---|---|
| Each additional 100 km | +2.50 hours |
| Stormy weather (vs clear) | +4.08 hours |
| Rainy weather (vs clear) | +3.05 hours |
| Foggy weather (vs clear) | +1.72 hours |

## Optimization

New promise = predicted delivery time + a safety buffer of 2.07 hours (the 90th percentile of the model's mistakes, so about 90% of deliveries should meet the promise).

| Delivery mode | Current promise | New promise | On time now | On time with new promise |
|---|---|---|---|---|
| Express | 4.5 h | 8.3 h | 35.8% | 88.3% |
| Same day | 8.0 h | 8.2 h | 78.1% | 91.2% |
| Standard | 24.0 h | 8.3 h | 100.0% | 90.3% |
| Two day | 16.0 h | 8.3 h | 99.9% | 89.5% |
| **Overall** | | | **78.2%** | **89.8%** |

On-time here means the actual time is less than or equal to the promised time. This is calculated directly and is slightly different from the dataset's own `delayed` column (73.3% on time).

## Recommendations

1. Replace fixed promises with promises based on distance and weather.
2. Add extra time on stormy (about 4 hours), rainy (about 3 hours), and foggy (about 1.7 hours) days, and plan extra driver capacity.
3. Focus route planning on shorter distances: each 100 km adds 2.5 hours.
4. Either make express truly faster or stop promising 4.5 hours.
5. Collect traffic, loading time, and driver data before building more complex models.

## Limitations

- About 27% of the variation in delivery time is not explained, because the dataset has no traffic, loading time, or driver information.
- The safety buffer is the same for every delivery, although mistakes may be larger in bad weather (not tested).
- The new promises were tested on historical data, not in real operations. A small pilot is needed before a full rollout.

## Next Steps

- Add traffic and loading time data.
- Try a different buffer for each delivery mode and weather type.
- Predict which deliveries will be late before they are sent.

