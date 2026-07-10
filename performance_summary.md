# Performance Summary: Regime-Shift TAA Strategy

This document provides a performance evaluation of the **Regime-Shift Tactical Asset Allocation (TAA) Strategy** against two static benchmarks: a **Static 60/40 Portfolio (60% SPY, 40% TLT)** and an **Equal Weight Portfolio (33.3% SPY, 33.3% TLT, 33.3% GLD)**.

The backtest period covers **2012-05-18 to 2023-12-29** (out-of-sample test period following the initial 1000-day training window starting in 2008). All metrics include a transaction cost of **10 bps per rebalance**.

---

## 📊 Backtest Performance Metrics

| Metric | Strategy (HMM + CVXPY) | Static 60/40 SPY/TLT | Equal Weight SPY/TLT/GLD |
| :--- | :---: | :---: | :---: |
| **Total Return** | **89.78%** | 167.65% | 93.88% |
| **Annualized Return (CAGR)** | **5.72%** | 8.93% | 5.92% |
| **Annualized Volatility** | **10.45%** | 10.36% | 9.07% |
| **Sharpe Ratio (Rf=0%)** | **0.5478** | 0.8616 | 0.6530 |
| **Sortino Ratio** | **0.7896** | 1.0813 | 0.8849 |
| **Maximum Drawdown** | **-23.73%** | -27.24% | -22.72% |
| **Calmar Ratio** | **0.2412** | 0.3278 | 0.2606 |
| **Annual Turnover** | **1.8075** | 0.0000 | 0.0000 |

---

## 🔍 Detailed Analysis & Observations

### 1. Risk Mitigation (Drawdown Control)
*   The primary goal of a macro-aware tactical asset allocation strategy is to protect capital during market crises. The Strategy achieved a **maximum drawdown of -23.73%**, which is **3.51% lower** than the static 60/40 portfolio's drawdown of **-27.24%**.
*   This was achieved by shifting the portfolio out of SPY and into safer assets (primarily TLT and GLD) when the HMM classifier flagged a **Crisis** state.

### 2. Return Drag (Underperformance vs. Benchmarks)
*   During the backtest period (2012-2023), the S&P 500 (SPY) experienced one of its most persistent and aggressive bull runs in history, fueled by low interest rates.
*   Because the HMM-based strategy is designed to be risk-averse and defensive in periods of moderate stress (Bear) or high volatility (Crisis), it reduced equity exposure during minor corrections or volatile sideways periods.
*   This defensive shifting created a drag on cumulative returns compared to a static, equity-heavy 60/40 portfolio that remained fully exposed to the rising stock market.

### 3. Volatility and Sharpe Ratio
*   The Strategy's annualized volatility was **10.45%**, comparable to the static 60/40 portfolio (**10.36%**).
*   Due to the lower CAGR and comparable volatility, the Sharpe Ratio of the Strategy (**0.5478**) was lower than that of the static 60/40 portfolio (**0.8616**) and the Equal Weight portfolio (**0.6530**).

### 4. Turnover and Transaction Costs
*   The annual turnover of the Strategy was **1.8075**, meaning the portfolio rebalanced roughly 1.8 times the total portfolio value per year.
*   At 10 bps per trade, this transaction cost was explicitly modeled and deducted from the daily portfolio returns, subtracting approximately **18 bps** of performance annually. This confirms the strategy does not rely on "free trading" and would remain viable in a real account.

---

## 🎨 Walk-Forward HMM Transition Matrix
The transition probability matrix of the final HMM is shown below. This represents the probability of switching between states from day $t$ to day $t+1$:

$$
P = \begin{bmatrix}
0.985 & 0.015 & 0.000 \\
0.038 & 0.957 & 0.005 \\
0.000 & 0.072 & 0.928
\end{bmatrix}
$$

*   **State 0 (Bull):** Extremely sticky, with a 98.5% daily probability of staying in the same state.
*   **State 1 (Bear):** Highly sticky (95.7%), with a small probability (3.8%) of reverting to Bull.
*   **State 2 (Crisis):** Sticky (92.8%), with a 7.2% daily probability of transitioning to Bear once volatility subsides.

This transition matrix confirms that the detected regimes are highly stable and persistent, preventing the strategy from "whipsawing" or changing weights too frequently.
