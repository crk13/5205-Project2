# 5205 Project 2: Ridge Regression, Ensemble Model & Model Comparison

This repository contains the implementation of the **Ridge Regression** and **Sharpe-weighted Ensemble** models for predictive signal generation and model comparison in a high-frequency trading context.  
The project focuses on evaluating model performance based on **Out-of-Sample (OOS) R²** and **Sharpe Ratio**, highlighting the trade-off between statistical accuracy and trading profitability.

---

## Overview

The project aims to forecast **next-minute log returns** using a **30-minute rolling window** of features.  
We benchmark four base models — *Lasso, Ridge, LightGBM,* and *Transformer* — and integrate them through a **dynamic Sharpe-weighted Ensemble**.  
The analysis explores both predictive quality and trading relevance through rolling backtesting and signal-level evaluation.

---

## Repository Structure

```

Project2
│
├── data/
│   └── ridge_result_df.csv          # Ridge rolling backtest results (others uploaded via Canvas)
│
├── results/                         # All model backtest results (Lasso, Ridge, LightGBM, Transformer, Ensemble)
│
├── Plots/
│   ├── oos_r2_plot.png              # OOS R² trend over test period
│   └── sharpe_comparison.png        # Signal Sharpe ratio comparison
│
├── eda.ipynb                        # Initial feature integration and duplicate removal
├── ridge_data_checking.ipynb        # Detects nulls, extreme values, and removes bad columns
├── ridge_hyper_param_search.ipynb   # Ridge training, alpha tuning, and rolling window backtest
├── weighted_model.ipynb             # Ensemble modeling and model comparison
│
└── README.md

````

---

## Dependencies

```python
import pandas as pd
import numpy as np
import joblib
import re
from tqdm import tqdm
import matplotlib.pyplot as plt
import seaborn as sns

from sklearn.base import BaseEstimator, TransformerMixin
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler, MaxAbsScaler
from sklearn.linear_model import Ridge
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer, TransformedTargetRegressor
from sklearn.model_selection import TimeSeriesSplit, GridSearchCV
````

These packages are available in standard Python distributions such as **Anaconda** or can be installed via:

```bash
pip install pandas numpy seaborn scikit-learn tqdm matplotlib joblib
```

---

## How to Run

All notebooks can be executed sequentially in **Jupyter Notebook** or **VS Code**.

1. **Feature Preparation**

   * `eda.ipynb`: Combines raw feature files and removes duplicates.
   * `ridge_data_checking.ipynb`: Identifies null or extreme-value columns and filters out invalid features.

2. **Model Training and Validation**

   * `ridge_hyper_param_search.ipynb`:
     Performs Ridge Regression training and hyperparameter search (`alpha ∈ [1, 10,000]`).
     After fine-tuning, the best alpha is **75**.
     The notebook implements a **30-minute rolling window** to predict the **next-minute log return**, followed by rolling backtesting.

3. **Model Ensemble and Evaluation**

   * `weighted_model.ipynb`:
     Builds the **dynamic Sharpe-weighted Ensemble** combining all four base models.
     Compares signal-level metrics (Sharpe, OOS R²) and visualises performance using plots under `/Plots/`.

---

## Outputs

| Folder / File                  | Description                                                    |
| ------------------------------ | -------------------------------------------------------------- |
| `/data/ridge_result_df.csv`    | Ridge model rolling backtest results                           |
| `/results/`                    | Backtest outputs of all models (used for ensemble aggregation) |
| `/Plots/oos_r2_plot.png`       | Cumulative Out-of-Sample R² across all models                  |
| `/Plots/sharpe_comparison.png` | Sharpe ratio comparison (signal-level)                         |

---

## Key Methodology

* **Rolling Window Prediction:**
  Each model is trained on a 30-minute window and predicts the next 1-minute log return.

* **Feature Selection:**
  75 key features selected via Lasso regression.

* **Ridge Regression:**
  Tuned using grid search and TimeSeriesSplit.
  Serves as a baseline for stability and interpretability.

* **Dynamic Sharpe-weighted Ensemble:**
  Model weights are updated every minute based on each model’s recent **raw Sharpe ratio** (clipped at 0).
  The ensemble prediction is the weighted average of all models’ log return forecasts.

* **Performance Metrics:**

  * **Out-of-Sample (OOS) R²:** Measures predictive fit relative to a zero-return baseline.
  * **Sharpe Ratio:** Measures risk-adjusted consistency of trading signals.

---

## Results Summary

| Model                      | OOS R² | Signal Sharpe | Annualised Sharpe |
| -------------------------- | ------ | ------------- | ----------------- |
| Lasso                      | -0.166 | 0.92          | 287.61            |
| Ridge                      | -0.030 | 0.79          | 248.01            |
| LightGBM                   | -0.039 | 1.01          | 316.07            |
| Transformer                | 0.000  | **2.04**      | **639.72**        |
| Ensemble (Sharpe-weighted) | -0.092 | **0.40**      | 87.15             |

* The **Transformer** achieves the best Sharpe Ratio (~2.0), capturing strong temporal dependencies.
* **LightGBM** performs moderately well but shows higher variance.
* **Ridge** and **Lasso** provide stable yet conservative signals.
* The **Ensemble Model** produces smoother but weaker results — frequent weight updates led to signal cancellation.

---

## Interpretation

* **Negative OOS R²** values are expected in high-frequency finance due to noise-dominated returns.
  Even strong trading signals often fail to outperform the naïve zero predictor statistically.
  Hence, **Sharpe Ratio** serves as the more meaningful metric for evaluating model value.
* **Sharpe > 1.0** is considered good, and **>2.0** is strong.
  Under this benchmark, only the Transformer achieved high-quality, risk-adjusted returns.
  The ensemble stabilised volatility but sacrificed profitability.

---

## Future Improvements

To enhance ensemble robustness and reduce short-term noise sensitivity:

* Apply **smoothed or exponentially decayed Sharpe estimation** for more stable weighting.
* Introduce a **Bayesian / Kalman filtering layer** to infer latent model performance.
* Add **cross-model correlation penalisation** to prevent redundant exposures.
* Explore **multi-horizon weighting**, integrating short- and medium-term Sharpe trends.

---

## Notes

* Due to GitHub’s **100 MB file limit**, only Ridge results are included in `/data/`.
  Complete datasets and full model outputs are available on **Canvas**.
* All code was tested in Python 3.11 under macOS and Linux environments.

