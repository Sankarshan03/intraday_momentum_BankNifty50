# Intraday Momentum Strategy using Adaptive EMA (AEMA) with Market Regime Filter

## Overview

This Jupyter notebook implements an **intraday momentum-based trading strategy** for the BankNifty index (^NSEBANK) using an **Adaptive Exponential Moving Average (AEMA)** with a **market regime filter**. The strategy dynamically adjusts its smoothing factor based on market volatility (Average True Range) and only takes trades during trending market conditions, making it more robust and selective.

The notebook downloads 1-hour OHLCV data from Yahoo Finance, computes the AEMA, classifies market conditions, generates trading signals based on the AEMA crossover, backtests the strategy with and without transaction costs, and performs a walk-forward analysis to assess robustness.

---

## Features

- **Data Download**: Fetches 1-hour BankNifty data from Yahoo Finance (last 1 year).
- **Adaptive EMA (AEMA)**:
  - ATR (Average True Range) computed as a percentage of price.
  - Dynamic smoothing factor (`alpha`) scaled from ATR using min-max normalization.
  - EMA with variable `alpha` for each time step.
  - ATR period set to **30** for more stable volatility measurement.
- **Market Regime Filter**:
  - Classifies market as **"Trending"** or **"Flat"** based on the 70th percentile of the dynamic smoothing factor.
  - Only takes signals during **Trending** periods to avoid whipsaws in choppy markets.
- **Trading Logic**:
  - **Long**: Close > AEMA (bullish momentum)
  - **Short**: Close < AEMA (bearish momentum)
  - Positions are taken on the next hour (signal shifted by 1, to avoid look-ahead bias).
- **Backtesting**:
  - Gross and net returns (with 0.01% per side transaction cost).
  - Performance metrics: Annualized Return, Max Drawdown, Sharpe Ratio, Sortino Ratio, Calmar Ratio, Hit Ratio, Information Ratio.
- **Walk-Forward Analysis**:
  - Data split into 3 chronological windows.
  - In each window, optimal ATR threshold is found on 70% in-sample data and tested on 30% out-of-sample data.
  - Percentiles tested: [0.5, 0.6, 0.7, 0.8].
- **Visualization**:
  - Plot of Close price and AEMA.
  - Cumulative return curves (gross and net).

---

## Requirements

Install the required Python libraries:

```bash
pip install pandas numpy matplotlib yfinance plotly
```

**Note**: The notebook uses `plotly` for interactive charts. If you prefer static plots, you can replace with `matplotlib`.

---

## Usage

1. **Run the notebook sequentially** from top to bottom.
2. **Modify parameters**:
   - `start_date` and `end_date`.
   - `atr_period` (default **30**) in `calculate_dynamic_smoothing_factor()`.
   - `COST_PER_TRADE` (default 0.0001 = 0.01% per side) in cost modeling cell.
   - Walk-forward threshold percentiles `[0.5, 0.6, 0.7, 0.8]`

3. **Interpret results**:
   - Check performance metrics table (gross vs net).
   - Evaluate walk-forward out-of-sample returns to gauge strategy robustness.

---

## Key Functions

| Function | Description |
|----------|-------------|
| `average_true_range()` | Computes ATR as a percentage of Close price. |
| `calculate_dynamic_smoothing_factor()` | Maps ATR to a smoothing factor `alpha` using min-max scaling. |
| `calculate_adaptive_ema()` | Calculates EMA with variable `alpha` for each time step. |
| `run_backtest()` | Helper function for walk‑forward analysis; applies threshold, signals, costs, and returns net cumulative return. |

---

## Methodology

1. **Data Preparation**:
   - Download 1‑hour OHLCV data.
   - Flatten MultiIndex columns, add `Date` and `Time` columns.

2. **Adaptive EMA Calculation**:
   - Compute ATR (**30‑period**) as a percentage of price.
   - Normalize ATR to [0,1] to get dynamic smoothing factor.
   - Apply recursive EMA formula: `EMA_t = alpha * price_t + (1 - alpha) * EMA_{t-1}`.

3. **Signal Generation**:
   - `signal = 1` if Close > AEMA **AND** market is Trending (long).
   - `signal = -1` if Close < AEMA **AND** market is Trending (short).
   - `signal = 0` if market is Flat (no position).
   - `position = signal.shift(1)` to avoid look‑ahead bias.

4. **Backtesting**:
   - Strategy return = hourly_return × position.
   - Transaction costs applied when position changes (`trade_execution` flag).

5. **Walk‑Forward Analysis**:
   - Split data into 3 chronological chunks.
   - For each chunk, split into 70% in‑sample (IS) and 30% out‑of‑sample (OOS).
   - Find threshold that maximizes IS net return (by testing percentiles 0.5, 0.6, 0.7, 0.8).
   - Evaluate OOS performance with that threshold.

---

## Results Summary (from notebook)

| Metric | Gross (No Cost) | Net (Post‑Cost) |
|--------|-----------------|-----------------|
| Annualized Return | 32.46% | 23.87% |
| Max Drawdown | -10.12% | -11.00% |
| Sharpe Ratio | 2.12 | 1.56 |
| Sortino Ratio | 3.05 | 2.24 |
| Calmar Ratio | 3.21 | 2.17 |
| Hit Ratio | 35.63% | 35.02% |
| Information Ratio | 1.01 | 0.71 |

**Walk‑Forward OOS Returns** (net of costs):
- Window 1: 0.05%
- Window 2: 1.2%
- Window 3: -0.3%

> **Note**: The market regime filter significantly improves performance compared to the version without this filter. The strategy shows positive gross returns with a reasonable Sharpe ratio. However, transaction costs still erode a significant portion of the returns. The walk‑forward results show mixed performance, with windows 1 and 2 performing well but window 3 showing a loss.

---

## Limitations & Considerations

- **Transaction Costs**: Assumes fixed 0.01% per side. Real‑world costs (slippage, brokerage, taxes) may be higher, as evidenced by the significant drop in performance post-cost.
- **Data Frequency**: 1‑hour data may have gaps due to market holidays or intraday breaks.
- **Look‑Ahead**: The walk‑forward analysis uses in‑sample optimization; but the optimal threshold is still chosen based on past data. Future performance may vary.
- **Overfitting**: The strategy uses a simple crossover; more complex filters could improve robustness.
- **Leverage**: The backtest assumes trading 1 unit per signal; leverage can amplify returns and drawdowns.
- **Short Selling**: The strategy assumes ability to short the index, which may not be practical for all investors.
- **Benchmark Performance**: The positive information ratio (0.71 net) suggests the strategy adds modest value beyond the benchmark.

---

## Future Improvements

- Optimize ATR period and threshold percentiles systematically.
- Implement dynamic position sizing (e.g., volatility‑adjusted).
- Test on other indices or asset classes.
- Include slippage and market impact models.
- Consider a long-only version to reduce transaction costs.
- Incorporate additional market regime indicators (e.g., volume, volatility).
- Explore machine learning approaches for regime detection.
- Add a stop-loss mechanism to limit drawdowns.
