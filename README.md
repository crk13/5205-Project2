# 5205-Project2

This repository documents the complete end-to-end workflow for developing, training, and backtesting a high-frequency quantitative trading strategy.

The project is divided into the following modular stages, presented in chronological order. Each stage is contained within its own Jupyter Notebook, which includes detailed code, model-specific analysis, and findings.

## Project Workflow & Pipeline

### 1. Data Collection, Processing & Feature Engineering

* [**`data_selected_code`**](https://github.com/crk13/5205-Project2/tree/data/data_selected_code)
* [**Data README (Click Here)**](https://github.com/crk13/5205-Project2/blob/data/README.md)

### 2. Predictive Modeling  and Ensemble Learning

#### 2.1 Lasso Regression

### 2.2 LightGBM

1. **Hyperparameter Tuning (All Features)** — *10-Fold TimeSeriesSplit*  
   *Focus:* Full feature set with interpretable model diagnostics.

2. **Hyperparameter Tuning (Lasso-Selected 75 Features)**  
   *Focus:* Sparse feature subset from Lasso for efficient modeling.

3. **Rolling Window Backtesting (Test Set, 30-Minute Horizon)**  
   *Focus:* Out-of-sample evaluation with dynamic re-training.

4. **Overfitting Diagnostics & Model Stability Check**  
   *Focus:* Evaluate generalization, rolling IC, and Sharpe consistency.



#### 2.3 Transformer

#### 2.4 Ridge Regression, Ensemble Model & Model Comparison

* [**`Ridge Regression notebook`**](https://github.com/crk13/5205-Project2/blob/ridge_regression/ridge_hyper_param_search.ipynb)
* [**`Ensemble Model & Model Comparison notebook`**](https://github.com/crk13/5205-Project2/blob/ridge_regression/weighted_model.ipynb)
* [**Ridge Regression, Ensemble Model & Model Comparison README (Click Here)**](https://github.com/crk13/5205-Project2/tree/ridge_regression)

### 3. Strategy Backtest & Analysis

* [**`strategy.ipynb`**](https://github.com/crk13/5205-Project2/blob/Strategy/strategy.ipynb)
* [**Strategy README (Click Here)**](https://github.com/crk13/5205-Project2/blob/Strategy/README.md)

---
**Note on final dataset upload**  
The `data/` folder in this repository only demonstrates the full **data-processing workflow and source code**.  
The final merged dataset used for model execution (`final_df.csv`) exceeds Github’s 100 MB upload limit and therefore cannot be hosted here.  
We have **uploaded `final_df.csv` separately to Canvas** — please **download it from Canvas and place it under**  
`5205-Project2/data/` **before running the notebooks or scripts**.

**Note on Reproducibility:**
For a detailed breakdown of the final strategy's implementation, precise reproduction commands, and expected outputs (as required by the "Reproducibility check"), please see the detailed in each readme.
