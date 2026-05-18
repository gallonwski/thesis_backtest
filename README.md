# thesis_backtest – ML Backtest for Thesis

This repository contains the code for my bachelor's thesis case study on **machine learning in financial markets** at **Università Ca’ Foscari Venezia**, with a focus on the distinction between **predictive performance** and **economic value** net of transaction costs.

The core idea is simple:  
the same predictive model (Random Forest stock selection) is implemented through different portfolio rules, and we show how **turnover, rebalancing frequency and costs** can completely change realized performance.

Repository: https://github.com/gallonwski/thesis_backtest

---

## Project overview

The project implements an end‑to‑end workflow:

- Download historical price data for a universe of large‑cap U.S. stocks
- Build a monthly panel with lagged technical features
- Train a **Random Forest** model in a rolling window to predict one‑month‑ahead returns
- Measure predictive power using the **Rank Information Coefficient (RIC)**
- Construct three long‑only portfolio strategies with different implementation rules:
  - Strategy A: monthly rebalancing
  - Strategy B: quarterly rebalancing
  - Strategy C: partial adjustment (smoothing)
- Apply transaction costs and compute **net** performance
- Compare strategies and benchmark (S&P 500) via:
  - Equity curves
  - Drawdowns
  - Turnover and cost drag
  - Cost sensitivity plots

The goal is **illustrative** rather than commercial: this is a didactic example of how ML signals translate into portfolio-level outcomes.

---

## Repository structure

The repository is organized as follows:

```text
.
├── data/                    # Raw / intermediate data (e.g. downloaded prices)
├── output/                  # Generated results (csv, figures)
│   ├── equity_curves.png
│   ├── drawdown.png
│   ├── rank_ic_timeseries.png
│   ├── cost_sensitivity.png
│   └── summary_tables.csv
├── thesis_backtest.ipynb    # Main notebook with full pipeline
├── requirements.txt         # Python dependencies
└── README.md                # This file
```

You can adapt folders as needed (e.g. add `src/` if you refactor helper functions out of the notebook).

---

## Methodology

### Universe and data

- Universe: 20 large‑cap U.S. stocks (subset of S&P 500)
- Frequency: monthly (from daily adjusted prices)
- Sample period: 2010‑01 to 2024‑12
- Data source: Yahoo Finance (`yfinance`)

### Features (predictors)

The model uses a compact set of **lagged technical features**, for example:

- 12‑month momentum
- 3‑month momentum
- 6‑month rolling volatility
- Price relative to 52‑week high
- Log‑price as size proxy
- 1‑month reversal

All features are lagged by one month to reduce look‑ahead bias.

### Model

- Algorithm: `RandomForestRegressor` (scikit‑learn)
- Rolling window: 60 months (5 years of history)
- Re‑estimated monthly
- Evaluation metric: **Rank Information Coefficient (RIC)**  
  (Spearman correlation between predicted and realized cross‑sectional returns)

### Portfolio strategies

All strategies use the same predicted ranking each month and hold the **top 20%** of stocks long‑only with equal weights:

- **Strategy A – Monthly rebalancing**  
  Rebalances to the new top‑20% portfolio every month.

- **Strategy B – Quarterly rebalancing**  
  Rebalances only every 3 months, holding weights constant in between.

- **Strategy C – Partial adjustment (smoothing)**  
  Each month moves only a fraction (for example 30%) from current weights toward the new target weights.

### Transaction costs

- Baseline: 0.2% per side (0.4% round‑trip) on traded notional
- Turnover: sum of absolute weight changes each month
- Net return:  
  `net_return = gross_return – turnover * cost_per_side`

A **cost sensitivity study** varies the cost between 0.1% and 0.3% per side.

---

## Key results (conceptual)

At a high level, the backtest illustrates that:

- The Random Forest model has **weak but positive** predictive power (RIC slightly above zero, hit rate just over 50%).
- All three strategies can generate **positive net Sharpe ratios** versus the S&P 500 benchmark.
- **Strategy B (quarterly rebalancing)** typically delivers:
  - higher **net** return,
  - higher **net Sharpe**,
  - lower **max drawdown**,
  - and lower sensitivity to transaction costs
  than the fully‑monthly Strategy A.
- The ranking between A and B **reverses** once costs are included:
  - gross performance: A ≳ B  
  - net performance: B > A  
  driven entirely by the difference in turnover.

This supports the main thesis:  
**predictive performance and economic value are not the same thing**, and implementation (turnover and costs) can dominate small differences in model accuracy.

---

## How to run

1. **Clone the repo**

```bash
git clone https://github.com/gallonwski/thesis_backtest.git
cd thesis_backtest
```

2. **Create and activate a virtual environment** (optional but recommended)

```bash
python -m venv .venv
source .venv/bin/activate   # macOS/Linux
# .venv\Scripts\activate    # Windows
```

3. **Install dependencies**

```bash
pip install -r requirements.txt
```

4. **Run the notebook**

- Open `thesis_backtest.ipynb` in VS Code / Jupyter
- Select the `.venv` kernel (if you created one)
- Run all cells in order:
  - data download and feature engineering
  - rolling Random Forest
  - portfolio construction
  - performance, charts, and tables

Generated figures and tables will be saved under `output/`.

---

## Requirements

Main Python packages:

- `pandas`
- `numpy`
- `yfinance`
- `scikit-learn`
- `matplotlib`
- `jupyter` / `ipykernel`

A full list is in `requirements.txt`.

---

## Disclaimer

This project is for **educational and research purposes only**.  
It is not investment advice and is not intended for live trading.  
All results are based on historical simulations with simplified assumptions and are subject to model risk, data issues, and implementation constraints.

---

## Thesis context

This repository accompanies my bachelor’s thesis at **Università Ca’ Foscari Venezia** on **machine learning in financial markets**, with a specific focus on the gap between **predictive performance** and **economic value**. The empirical work implemented here illustrates, in a simple and transparent setup, how turnover, transaction costs and rebalancing rules can materially alter the economic value of a given predictive signal.
