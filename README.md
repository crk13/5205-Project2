# 5205-Project2-Trading Strategy

### 1. Overview

This directory contains the main backtesting script: `strategy.ipynb`.

This script is designed to:

1. Load predictive signals (ensemble_predictions.csv), asset prices (`price5.csv`), and benchmark data (`benchmark.csv`).

2. **Filter** all data to a specific out-of-sample (OOS) date range (`START_FILTER` to `END_FILTER`).

3. **Calibrate**: Automatically find the optimal `K` parameter by testing a `K_GRID` on the *first 10%* of the filtered data.

4. **Backtest**: Run a high-performance, fully-vectorized backtest on the *entire* dataset using the single `best_K` found during calibration.

5. **Analyze**: Generate performance tables, OLS regression statistics, and plots.

### 2. Required Files

To ensure reproducibility, the script must be run in a directory with the following structure:

```

/Project2/
│
├── strategy.ipynb
├── ensemble\_predictions.csv
├── price5.csv
└── benchmark.csv

````

* `ensemble_predictions.csv`: Must contain `datetime`, `symbol`, `ensemble_predicted_log_return`, `ensemble_weight_relative`.

* `price5.csv`: Must contain `datetime`, `symbol`, `open`, `close`.

* `benchmark.csv`: Must contain `datetime`, `symbol`, `close`.

### 3. How to Run

The script requires several common scientific Python packages.

**1. Install Dependencies:**

```bash
pip install pandas numpy matplotlib statsmodels jupyter
````

**2. Run the Script:**
Execute the script directly from your terminal. The script will run all steps (Data \> Filter \> Calibrate \> Backtest \> Analyze) sequentially.

```bash
jupyter nbconvert --to notebook --execute strategy.ipynb
```

### 4\. Expected Output

A successful run will print the following sequence to the console. It will also open several `matplotlib` windows to display the result plots.

```console
Reading CSVs...
Data before filtering: 2025-04-28 09:30:00 to 2025-10-27 15:59:00
Data filtered: 2025-04-27 15:59:00 to 2025-10-27 15:59:00. Total 75421 minutes.
Calibrating K...
Calibrated K = 1.5, calibration Sharpe = 0.0824
Running strategy (Vectorized)...
Running strategy on 75420 minutes...
Strategy run complete (Vectorized).
Computing performance metrics...

=== Performance summary ===
AnnReturn_strategy        : -9.538817
AnnVol_strategy           : 0.105500
Sharpe_strategy           : -90.415449
AvgTurnover_per_min       : 0.204368
MaxDrawdown               : -0.824425
AnnExcess_vs_SMH          : -12.566552
AnnReturn_SMH             : 3.027735
AnnVol_SMH                : 0.438516
AnnReturn_SPY             : 1.203747
AnnVol_SPY                : 0.191858
Calibrated_K              : 1.5

Regression vs SMH (alpha, beta, t-stats):
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const      -9.763e-05   2.51e-06    -38.932      0.000      -0.000   -9.27e-05
ret_t_SMH      0.0185      0.002     10.329      0.000       0.015       0.022
==============================================================================

Regression vs SPY (alpha, beta, t-stats):
==============================================================================
                 coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
const      -9.773e-05    2.5e-06    -39.054      0.000      -0.000   -9.28e-05
ret_t_SPY      0.0549      0.004     13.417      0.000       0.047       0.063
==============================================================================
Saved strategy_results.csv and performance_summary.csv
Plotting...
Done.
```

### 5\. Generated Artifacts (Outputs)

This script will generate two new files in the same directory:

1.  **`performance_summary.csv`**:

      * A single-row CSV file containing the final, aggregate metrics reported in the "Performance summary" table (e.g., `Sharpe_strategy`, `AnnReturn_strategy`, `MaxDrawdown`).

2.  **`strategy_results.csv`**:

      * A large, minute-by-minute CSV file detailing the strategy's P\&L (`pnl`, `pnl_gross`, `costs`), `turnover`, `cum_net`, target positions (`pos_NVDA`, etc.), and the raw signals (`predret_...`, `real_ret_t_...`) at every step.

### 6\. Key Configuration Parameters

All parameters can be modified directly in the "User Configuration" block at the top of `strategy.ipynb`:

  * `ASSETS = ['NVDA', 'MU', 'AVGO', 'AMD', 'AMAT']`

      * The list of symbols to include in the strategy.

  * `START_FILTER = '...'` / `END_FILTER = '...'`

      * The exact OOS time range for the backtest.

  * `COST_PER_TURN = 0.0005`

      * The transaction cost basis points (bps) applied to all turnover.

  * `K_GRID = [0.0, 0.25, 0.5, 1.0, 1.5,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0,10.0]`

      * The list of `K` values to test during the Step 5 calibration.

  * `CALIB_RATIO = 0.1`

      * The fraction of data (10%) to use for calibration.

  * `ROLL_STD_WIN = 300`

      * The 300-minute lookback window for the rolling volatility estimator ($\sigma_t$). This estimator is applied to the cross-sectional standard deviation of predictions.

<!-- end list -->

```
