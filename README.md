# GB Imbalance Price Nowcast + Battery Dispatch Backtest

This project pulls GB imbalance system prices from the Elexon BMRS Insights API, builds a short-horizon price nowcast, and converts that signal into charge/discharge decisions for a SOC-limited battery. It then backtests the dispatch and reports £ PnL.

## Notebooks
- `notebooks/01_pull_system_prices.ipynb`  
  Pulls system prices for a single settlement date from the API, creates timestamps, and saves the raw day file to `data/raw/`. This is a sanity-check notebook.

- `notebooks/02_build_price_history.ipynb`  
  Pulls a date range (eg last 365 days), caches each day to `data/raw/`, and combines everything into `data/processed/prices.parquet`.

- `notebooks/03_baseline_forecast.ipynb`  
  Creates lag + calendar features and computes a baseline forecast (lag-48: yesterday same settlement period). Reports baseline MAE.

- `notebooks/04_train_model.ipynb`  
  Trains a gradient-boosted tree model using time-series cross-validation and compares MAE vs baseline. Saves predictions to `data/processed/preds.parquet`.

- `notebooks/05_backtest.ipynb`  
  Converts predictions into a charge/discharge/hold signal using rolling quantiles, simulates a SOC-limited battery, and reports £ PnL + plots.

## Data
- Source: Elexon BMRS Insights API (system prices), half-hour settlement periods.
- Note: SBP and SSP are equal here under the single imbalance price scheme, so the project models one system price series.

## Model
- Baseline MAE: 40.56 £/MWh (lag-48)
- Model MAE (5-fold time split): 24.92 £/MWh (`HistGradientBoostingRegressor`)

## Backtest (battery dispatch)
Assumptions:
- 1 MW / 2 MWh battery
- 90% round-trip efficiency
- Charge/discharge/hold signal from rolling quantiles of recent prices

Results:
- Total PnL: £12,526.36
- Charge periods: 354
- Discharge periods: 279

## Assumptions / limitations
This is a simplified backtest intended to demonstrate workflow and reasoning. It ignores transaction costs, bid/offer spreads, stacking across multiple markets, and asset/network constraints.