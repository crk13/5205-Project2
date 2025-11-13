# LightGBM
All LightGBM experiments — including hyperparameter tuning, feature selection, 
and rolling-window evaluation — were conducted within two complementary notebooks:

- [`LightGBM (Lasso Features, Rolling Evaluation)`](https://github.com/crk13/5205-Project2/blob/27630ef72819a31b5a55a407ab4a51a08f204f7f/script/01_lgbm_lasso_features_rolling.ipynb)
- [`LightGBM (5-Feature Overfitting Adjustment)`](https://github.com/crk13/5205-Project2/blob/27630ef72819a31b5a55a407ab4a51a08f204f7f/script/02_lgbm_5features_rolling.ipynb)

#### Key Experiment Components

* **Hyperparameter Tuning (All Features)** — *10-Fold TimeSeriesSplit*  
   *Focus:* Full feature set used to identify feature importance and interpret model behaviour.

* **Hyperparameter Tuning (Lasso-Selected 75 Features)**  
   *Focus:* Sparse feature subset derived from Lasso selection for efficient modeling and fair comparison with linear baselines.

* **Rolling Window Backtesting (Test Set, 30-Minute Horizon)**  
   *Focus:* Out-of-sample evaluation under a dynamic retraining setup to emulate live trading conditions.

* **Overfitting Diagnostics & Model Stability Check**  
   *Focus:* Evaluate generalisation through rolling IC, Sharpe consistency, and validation–test divergence.

* **Reduced 5-Feature Version (Overfitting Adjustment)**  
   *Notebook:* [`02_lgbm_5features_rolling.ipynb`](https://github.com/crk13/5205-Project2/blob/27630ef72819a31b5a55a407ab4a51a08f204f7f/script/02_lgbm_5features_rolling.ipynb)  
   *Focus:* A simplified variant retrained using the top five features to verify that model performance 
   was not dominated by a few predictors. Results showed minimal degradation in Sharpe ratio, 
   confirming the robustness of the LightGBM signal extraction process.
