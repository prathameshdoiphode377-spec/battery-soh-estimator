# AI Battery Health Estimator

Estimating lithium-ion battery **State of Health (SOH)** from discharge data (voltage, current, temperature, time) using the NASA battery dataset, with a simple regression model.

SOH = present capacity / first-cycle capacity x 100

This is a study of battery data and simple models, not a complete battery-management system.

## Result

A one-feature linear model reaches an average **RMSE of 1.28 SOH points** (R2 0.981) on batteries it never saw during training.

```
SOH = 48.82 * t_norm + 47.24
```

`t_norm` is the time a cell takes to fall to 3.5 V during discharge, divided by the same value from that cell's first cycle.

| Model | Features | RMSE | R2 |
|---|---|---|---|
| Baseline (linear) | cycle number | 5.90 | 0.603 |
| Ridge | all 8 | 3.97 | 0.817 |
| Random forest | all 8 | 3.39 | 0.868 |
| XGBoost | all 8 | 3.35 | 0.864 |
| Ridge | 3 | 2.76 | 0.912 |
| Linear | t_to_3.5V | 1.83 | 0.958 |
| **Linear (final)** | **t_norm** | **1.28** | **0.981** |

RMSE is in SOH percentage points. Scores are the mean over leave-one-battery-out validation.

Main finding: one physically meaningful feature generalised better across cells than more features and more flexible models.

![Final model](final_model.png)

## Data

NASA Prognostics Center of Excellence Li-ion battery dataset, cells **B0005, B0006, B0007, B0018** (636 discharge cycles in total).

The data files are not stored in this repository. To get them:

1. Download: https://phm-datasets.s3.amazonaws.com/NASA/5.+Battery+Data+Set.zip (about 200 MB)
2. Unzip it and copy `B0005.mat`, `B0006.mat`, `B0007.mat` and `B0018.mat` (found in `1. BatteryAgingARC-FY08Q4`) into the `data/` folder.

Citation: B. Saha and K. Goebel (2007). Battery Data Set. NASA Prognostics Data Repository, NASA Ames Research Center, Moffett Field, CA.

## Method

- **Label:** SOH from measured capacity of each discharge cycle (capacity is never used as a feature).
- **Features (8 per cycle):** discharge duration, time to 3.5 V, mean voltage, minimum voltage, max and mean temperature, mean current, energy delivered.
- **Validation:** leave-one-battery-out (train on three cells, test on the fourth). Random splits would be misleading because neighbouring cycles are nearly identical.
- **Models:** cycle-count baseline, ridge, random forest, XGBoost, then smaller feature sets and a normalised one-feature linear model.
- **Metrics:** RMSE, MAE, R2.

## Project structure

- `data/` - NASA .mat files (not in the repo, see Data)
- `notebooks/soh.ipynb` - full analysis
- `results_final_model.csv`, `final_model.png` - final results
- `requirements.txt` - Python packages

## How to run

```bash
pip install -r requirements.txt
```

Put the four `.mat` files in `data/`, open `notebooks/soh.ipynb` (for example in VS Code with the Jupyter extension), choose your Python kernel and click **Run All**.

## Limitations

- Only four cells from one lab at room temperature.
- Needs a full constant-current discharge, and a first-cycle reference measurement for each cell.
- The feature was chosen after looking at correlations on all four cells, so the reported error is slightly optimistic.
- A new cell is predicted at about 96%, not 100%, because the line is not forced through 100%.
- Hyperparameters were not tuned. The Oxford Battery Degradation Dataset has not been tested.

## Future work

Oxford dataset test, partial-discharge and charge-phase features, incremental capacity (dQ/dV) analysis, feature selection inside each fold, remaining useful life prediction.

## Author

Prathamesh Doiphode, Bharati Vidyapeeth College of Engineering, Pune