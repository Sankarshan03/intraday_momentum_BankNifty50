# Intraday Momentum Strategy using Adaptive EMA (AEMA) for BankNifty

## Overview

This Jupyter notebook implements an **intraday momentum-based trading strategy** for the BankNifty index (^NSEBANK) using an **Adaptive Exponential Moving Average (AEMA)**. The strategy dynamically adjusts its smoothing factor based on market volatility (Average True Range), making it more responsive during trending periods and less sensitive during choppy/flat markets.

The notebook downloads 1-hour OHLCV data from Yahoo Finance, computes the AEMA, generates trading signals, backtests the strategy with and without transaction costs, and performs a walk-forward analysis to assess robustness.

---

## Features

- **Data Download**: Fetches 1-hour BankNifty data from Yahoo Finance (last 1 year).
- **Adaptive EMA (AEMA)**:
  - ATR (Average True Range) computed as a percentage of price.
  - Dynamic smoothing factor (`alpha`) scaled from ATR using min-max normalization.
  - EMA with variable `alpha` for each time step.
- **Trading Logic**:
  - **Long**: Close > AEMA (bullish momentum).
  - **Short**: Close < AEMA (bearish momentum).
  - Positions are taken on the next hour (signal shifted by 1).
- **Backtesting**:
  - Gross and net returns (with 0.05% per side transaction cost).
  - Performance metrics: Annualized Return, Max Drawdown, Sharpe Ratio, Sortino Ratio, Calmar Ratio, Hit Ratio, Information Ratio.
- **Walk-Forward Analysis**:
  - Data split into 3 chronological windows.
  - In each window, optimal ATR threshold is found on 70% in-sample data and tested on 30% out-of-sample data.
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
   - `start_date` and `end_date` in cell 2.
   - `atr_period` (default 14) in `calculate_dynamic_smoothing_factor()`.
   - `COST_PER_TRADE` (default 0.0005 = 0.05% per side) in cost modeling cell.
   - Walk-forward threshold percentiles `[0.5, 0.6, 0.7, 0.8]` in cell 22.

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
   - Compute ATR (14‑period) as a percentage of price.
   - Normalize ATR to [0,1] to get dynamic smoothing factor.
   - Apply recursive EMA formula: `EMA_t = alpha * price_t + (1 - alpha) * EMA_{t-1}`.

3. **Signal Generation**:
   - `signal = 1` if Close > AEMA (long), else `-1` (short).
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
| Annualized Return | 22.52% | 4.66% |
| Max Drawdown | -8.80% | -14.22% |
| Sharpe Ratio | 1.47 | 0.30 |
| Sortino Ratio | 1.94 | 0.40 |
| Calmar Ratio | 2.56 | 0.33 |
| Hit Ratio | 34.20% | 31.60% |
| Information Ratio | 0.66 | -0.06 |

**Walk‑Forward OOS Returns** (net of costs):
- Window 1: -6.78%
- Window 2: -3.96%
- Window 3: +3.32%

> **Note**: While the gross strategy shows positive performance, transaction costs significantly erode net returns. The walk‑forward results indicate limited out‑of‑sample consistency, suggesting the strategy may be sensitive to market regime changes and transaction costs. The negative information ratio post-cost indicates the strategy underperforms the benchmark on a risk-adjusted basis after accounting for trading costs.

---

## Limitations & Considerations

- **Transaction Costs**: Assumes fixed 0.05% per side. Real‑world costs (slippage, brokerage, taxes) may be higher, as evidenced by the significant drop in performance post-cost.
- **Data Frequency**: 1‑hour data may have gaps due to market holidays or intraday breaks.
- **Look‑Ahead**: The walk‑forward analysis uses in‑sample optimization; but the optimal threshold is still chosen based on past data. Future performance may vary.
- **Overfitting**: The strategy uses a simple crossover; more complex filters could improve robustness.
- **Leverage**: The backtest assumes trading 1 unit per signal; leverage can amplify returns and drawdowns.
- **Short Selling**: The strategy assumes ability to short the index, which may not be practical for all investors.
- **Benchmark Underperformance**: The negative information ratio post-cost suggests the strategy does not add value beyond the benchmark once costs are considered.

---

## Future Improvements

- Add a trend‑filter (e.g., 200‑period SMA) to avoid trading during sideways markets.
- Implement dynamic position sizing (e.g., volatility‑adjusted).
- Test on other indices or asset classes.
- Include slippage and market impact models.
- Optimize ATR period and threshold percentiles systematically.
- Consider a long-only version to reduce transaction costs and allow for easier execution.
- Incorporate volume or volatility filters to reduce false signals.
- Explore machine learning approaches for regime detection.
