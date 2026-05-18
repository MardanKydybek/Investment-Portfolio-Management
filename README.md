# Systematic Long-Short Equity Strategy: Risk-Adjusted Momentum with Utility-Based Optimization

This repository contains the data, core optimization engine, production rebalancing logs, and performance analysis for a systematic market-neutral long-short equity strategy developed at HEC Liège (May 2026). 

The framework evaluates the S&P 500 universe using an aggressive risk-profile setup, testing the limits of non-linear utility functions under real-world market friction.

## Strategy Architecture

* **Signal Generation:** Risk-Adjusted Momentum factor. It evaluates the 12-month lookback window (excluding the most recent month to bypass short-term reversals) divided by the 60-day realized volatility:
    $$Score_{i} = \frac{R_{i}^{(12m-1m)}}{\hat{\sigma}_{i}^{(60d)}}$$
* **Portfolio Optimization:** A sub-linear utility function is maximized independently for the long and short legs (10 assets each) using the SLSQP algorithm:
    $$U = E[R] - 0.5 \cdot \lambda \cdot (\sigma^2)^q$$
    *Parameters:* $\lambda = 1.76$ (Low risk aversion), $q = 0.93$ (Sub-linear variance penalty).
* **Constraints:** 200% Gross Exposure (100% Long / 100% Short), individual asset bounds between 1% and 25%, and a strict 30% GICS sector cap to avoid structural sector concentration.

## Performance & Critical Risk Analysis

During a 3-month live tracking execution (Feb–May 2026) on the StockTrak platform, the strategy delivered a **+36.25% absolute return**, outperforming the S&P 500 by **+27.13%** with a realized Beta of **0.13**. 

### Empirical Reality Checks

While the live tracking period highly rewarded this configuration, a deeper look at the 10-year backtest and execution logs reveals critical structural vulnerabilities:

1.  **The Cost of Leverage:** Net of stock-borrowing costs (~8% p.a. margin fees), the 10-year backtest yielded a humble +1.92% annualized return. The financial drag of maintaining short positions heavily eroded the underlying stock-picking alpha.
2.  **Regime Blindness:** The sub-linear penalty factor ($q = 0.93$) generates extremely flat indifference curves. The optimizer is practically blind to tail risks, aggressively chasing high-variance names during trend expansions. Consequently, during the choppy/mean-reverting regime of 2016–2021, the strategy suffered an annualized loss of **-9.6%**. 
3.  **Operational Friction & Live Adjustments:** Live execution exposed infrastructure dependencies. For example, when Hologic Inc. (HOLX) was abruptly excluded from the S&P 500 index during the live trading phase, platform execution limits blocked liquidation requests. This forced the strategy to hold an unhedged 1,990-share position, temporarily disrupting the 10x10 neutral structure.

## Repository Structure

* `notebooks/`: 
    * `Group13_strategy_Implementation.ipynb`: Core notebook implementing cross-sectional ranking, covariance matrix estimation, and SLSQP non-linear optimization.
    * `Rebalancing_March26.ipynb`: Production log for the March execution cycle, adjusting positions to dynamic momentum shifts.
    * `Rebalancing_April26.ipynb`: Production log for the April execution cycle, dealing with weekend execution adjustments and index constituent shocks.
* `data/`: Raw execution data including full transaction history logs and daily portfolio equity curves.
* `reports/`: The final institutional academic report detailing the mathematical formulation and Brinson attribution analysis.

## Next Steps

The next iteration of this project focuses on transforming the static risk parameters into a dynamic regime-switching model. By conditioning the exponent $q$ and gross leverage on macro indicators (such as VIX thresholds and moving average trend filters), the model aims to automatically scale down exposure and switch to exponential risk penalties before regime-shifts decimate the capital.
