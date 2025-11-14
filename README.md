# 5205-Project2 (Li Yuxin)

Li Yuxin's branch for DSA5205 Project 2 collects the workflows used to benchmark a linear LASSO signal against a Transformer forecaster trained with a Huber loss. The project centers on two notebooks in `notebooks/` and a set of 30-minute backtests stored under `output/` for downstream model comparison.

## Environment & Dependencies
- Tested with Python 3.12 (`conda` env `dsa5205-p1`).
- Core packages: `numpy`, `pandas`, `matplotlib`, `seaborn`, `scikit-learn`, `torch`, `tqdm`, `joblib`, plus standard library modules (`pathlib`, `math`, `re`, `warnings`, `json`).
- Install with your preferred tool, e.g. `pip install numpy pandas matplotlib seaborn scikit-learn torch tqdm joblib`.

## Data
- The notebooks expect `final_df.csv` to live in a `data/` folder that sits alongside the notebooks directory (`notebooks/data/final_df.csv`).
- The raw CSV is too large to track in Git. Copy it manually before running anything and, if necessary, edit the `DATA_PATH`/`file_path` variables near the top of each notebook (they currently point to `../autodl-tmp/final_df.csv` in the Transformer notebook and `../data/final_df.csv` in the LASSO notebook).

## Reproducibility
Set a single experiment seed before running either notebook to keep data splits and PyTorch weight initialization stable:

```python
SEED = 42
import os, random, numpy as np, torch
os.environ["PYTHONHASHSEED"] = str(SEED)
random.seed(SEED)
np.random.seed(SEED)
torch.manual_seed(SEED)
torch.cuda.manual_seed_all(SEED)
```

Any ad-hoc sampling in the notebooks (e.g., scatter plots in `Transformer_Huber.ipynb`) also uses `random_state=42`.

## Key Notebooks
1. `notebooks/LASSO.ipynb`
   - Cleans tabular factors, removes unstable columns, applies median imputation → quantile clipping → scaling pipelines, then fits a `sklearn` LASSO across rolling windows.
   - Saves model diagnostics plus the ranked coefficients for the top 75 features (see `output/lasso_top_75_features.csv`).
   - Produces the 30-minute PnL stream in `output/lasso_rolling_backtest_results.csv`, which is later consumed for model comparison metrics.
2. `notebooks/Transformer_Huber.ipynb`
   - Builds sequence datasets (30-step lookback, 1-step horizon) with optional categorical embeddings, symbol IDs, and standardized targets.
   - Trains the `ReturnTransformer` using SmoothL1/Huber loss, logs training curves, and evaluates the full 30-minute backtest whose results land in `output/transformer_backtest_results_Huber_final.csv`.
   - Includes analysis cells (decile plots, residual diagnostics) driven by the saved backtest CSV.

## Outputs
- `output/lasso_rolling_backtest_results.csv`: LASSO 30-minute rolling backtest with realized returns for benchmarking.
- `output/transformer_backtest_results_Huber_final.csv`: Transformer (Huber) 30-minute backtest used for the final comparison table.
- `output/lasso_top_75_features.csv`: Feature names, signed coefficients, and absolute magnitudes for the top 75 LASSO signals.
- Additional Transformer experiments (`transformer_backtest_results_BCELoss.csv`, `transformer_backtest_results_MSE.csv`) are retained for reference but not part of the main report.

## Running the Project
1. Create/activate the environment and install the dependencies listed above.
2. Place `final_df.csv` under `notebooks/../data/` (or update the `DATA_PATH` variables to wherever you store the file).
3. Launch JupyterLab/Notebook, open `notebooks/LASSO.ipynb`, run all cells to regenerate the feature selection table and the LASSO backtest.
4. Open `notebooks/Transformer_Huber.ipynb`, ensure the data path points to the same CSV, set the seed cell, and run all cells to retrain the Transformer and refresh its backtest CSV.
5. Use the CSVs in `output/` for your downstream comparison script/report.

## Maintainer
Li Yuxin
