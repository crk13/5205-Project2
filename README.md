
# 5205-Project2 — Data Collection & Feature Engineering

## 1. Overview

This directory contains the **data ingestion**, **preprocessing**, and **alignment** pipelines that build three feature families used by downstream modeling/backtests:

- **Market features** (minute-level OHLCV + technical indicators)
- **Sentiment features** (minute-level news intensity/tone with robust intraday normalization)
- **Fundamental features** (dividend/split/EPS events aligned to the trading calendar)

Raw pulls are saved under `data/market`, `data/sentiment`, `data/fundamental` (and `data/benchmark` for ETFs if needed). Processed features are written to `data/features/*_features` as per-symbol CSVs, and the alignment step produces `ALL_*.csv` concatenations for modeling.

> **Runtime & Reproducibility**
> Some steps vary from minutes to hours depending on the date range and hardware. We provide **pre-generated artifacts in `data/features`** and expose **run switches in notebooks/scripts**. By default, heavy steps are **OFF**; toggle them to re-run the full pipeline when needed.

---

## 2. Repository Layout (Required Files/Folders)

```
/Project2/
│
├─ data_selected_code/
│   ├─ data_ingestion/
│   │   ├─ get_market_data.py
│   │   ├─ get_sentiment_data.py
│   │   └─ get_fundamental_data.py   # optional: ETF benchmarks (SPY, SMH)
│   ├─ preprocessing/
│   │   ├─ preprocess_market_data.py
│   │   ├─ preprocess_sentiment_data.py
│   │   └─ preprocess_fundamental_data.py
│   └─ data_alignment/
│       └─ build_all_csv.py
│
├─ data/
│   ├─ market/          # raw minute OHLCV by symbol
│   ├─ sentiment/       # raw minute sentiment by symbol
│   ├─ fundamental/     # raw daily fundamentals by symbol
│   ├─ benchmark/       # optional: ETF minute bars (SPY, SMH)
│   └─ features/
│       ├─ market_features/
│       ├─ sentiment_features/
│       └─ fundamental_features/
│
└─ .env                 # TWELVE_DATA_API_KEY
```

- `.env` must define `TWELVE_DATA_API_KEY`.
- BigQuery auth (for sentiment ingestion) uses Application Default Credentials or `GOOGLE_APPLICATION_CREDENTIALS`.

---

## 3. How to Run

### 3.1 Install Dependencies

```bash
# Core
pip install pandas numpy python-dateutil pytz python-dotenv requests

# Sentiment (GDELT via BigQuery)
pip install google-cloud-bigquery google-cloud-bigquery-storage
```

Authenticate BigQuery via:

```bash
gcloud auth application-default login
# or set GOOGLE_APPLICATION_CREDENTIALS to a service-account JSON
```

Place your Twelve Data key in `.env`:

```
TWELVE_DATA_API_KEY=your_key_here
```

### 3.2 Step A — Ingest Market (minute OHLCV)

```bash
python data_selected_code/data_ingestion/get_market_data.py   --start-date 2024-01-01 --end-date 2025-10-28 --interval 1min   --timezone America/New_York --chunk-bdays 10 --order ASC
```

Outputs one CSV per symbol in `data/market/`.

### 3.3 Step B — Ingest Sentiment (GDELT → minute)

```bash
python data_selected_code/data_ingestion/get_sentiment_data.py
```

Writes per-symbol minute-level files in `data/sentiment/` with counts and tone fields. Requires BigQuery credentials.

### 3.4 (Optional) Step C — Ingest Benchmarks (SPY, SMH)

```bash
python data_selected_code/data_ingestion/get_fundamental_data.py
```

Fetches ETF minute bars (SPY, SMH) into `data/benchmark/` if needed by analysis.

### 3.5 Step D — Preprocess Market → `*_mar_f.csv`

```bash
python data_selected_code/preprocessing/preprocess_market_data.py
```

Builds dense per-minute market features (returns, VWAP/EMA deviations, RSI, MACD, intraday seasonality), drops warm-up, applies robust z-scoring, and writes `data/features/market_features/<SYMBOL>_mar_f.csv`.

### 3.6 Step E — Preprocess Sentiment → `*_sen_core_f.csv` / `*_sen_sat_f.csv`

```bash
python data_selected_code/preprocessing/preprocess_sentiment_data.py
```

Creates a trading-minute panel, maps news to tradable minutes, engineers rolling/EWM/carry features, **normalizes by minute-from-open over trailing days**, deletes the first 200 minutes of each day, and exports **core** and **satellite** feature sets.

### 3.7 Step F — Preprocess Fundamental → `*_fun_f.csv`

```bash
python data_selected_code/preprocessing/preprocess_fundamental_data.py
```

Cleans dividend/split/EPS, builds 252-day rolling shares z-scores and event-robust z (8-event window), aligns non-trading-day events to the next trading day, and exports `data/features/fundamental_features/<SYMBOL>_fun_f.csv`.

### 3.8 Step G — Align & Concatenate → `ALL_*.csv`

```bash
python data_selected_code/data_alignment/build_all_csv.py
```

Checks schema consistency and concatenates per-symbol files into:
`ALL_mark_f.csv`, `ALL_fun_f.csv`, `ALL_sen_core_f.csv`, and `ALL_sen_sat_f.csv`.

---

## 4. Expected Console Flow

Ingestion prints per-symbol/progress messages. Preprocessing echoes shapes, window settings, and dropped warm-up diagnostics. Alignment prints column checks (e.g., `[OK] ALL_mark_f.csv share the same columns.`) and final save paths.

---

## 5. Generated Artifacts (Outputs)

- **Raw ingestion**

  - `data/market/<SYMBOL>_market_data_origin.csv` (minute OHLCV)
  - `data/sentiment/<SYMBOL>_sentiment_data_otigin.csv` (minute counts/tone)
  - `data/benchmark/*.csv` (SPY/SMH minute bars, optional)
- **Processed per symbol**

  - `data/features/market_features/<SYMBOL>_mar_f.csv`
  - `data/features/sentiment_features/<SYMBOL>_sen_core_f.csv`
  - `data/features/sentiment_features/<SYMBOL>_sen_sat_f.csv`
  - `data/features/fundamental_features/<SYMBOL>_fun_f.csv`
- **Concatenated (alignment)**

  - `data/features/market_features/ALL_mark_f.csv`
  - `data/features/fundamental_features/ALL_fun_f.csv`
  - `data/features/sentiment_features/ALL_sen_core_f.csv`
  - `data/features/sentiment_features/ALL_sen_sat_f.csv`

---

## 6. Key Configuration Parameters

**Common / Paths & tickers**

- Symbols: `["NVDA","AMD","AVGO","MU","AMAT"]` (used consistently across modules).

**Dates & warm-up**

- Suggested ingestion range: `FETCH_START_DATE="2024-01-01"` to `END_DATE="2025-10-28"`.
- Effective modeling range starts around `2024-04-30` to avoid cold starts.
- Drop the first `200` minutes for each day in sentiment features to remove unstable open-phase noise.

**Normalization / windows**

- Sentiment robust normalization by `minute_from_open` over the last `~60` trading days (with a minimum number of observations).
- Market robust z-scoring with a rolling window (e.g., `120` minutes; require sufficient history).
- Fundamental: `252` trading days for shares; event-robust z using an 8-event window (min 4), and IQR-rule outlier control (`1.5×IQR`).

**APIs & credentials**

- Twelve Data: `.env` with `TWELVE_DATA_API_KEY`.
- BigQuery: Application Default Credentials or service-account JSON via `GOOGLE_APPLICATION_CREDENTIALS`.

---

## 7. Reproducibility

- All paths are relative to this project.
- Random seeds are fixed where applicable.
- Heavy steps are switchable and their outputs are versioned by date/ticker.

---

## 8. Troubleshooting

- **Empty merges / NaNs**: ensure `datetime × symbol` grids align across market/sentiment/fundamental sources.
- **Missing credentials**: `.env` for Twelve Data; ADC or service account for BigQuery.
- **Very slow runs**: reduce the date window or disable optional plots; verify chunk size for API pulls.
- **Schema mismatch on alignment**: check that all per-symbol CSVs were produced by the same script version and window settings.

---

## 9. Acknowledgments

This dataset is built for DSA5205 Project 2 (research only). Public sources are accessed under their respective terms of use; credentials and rate limits must be respected.
