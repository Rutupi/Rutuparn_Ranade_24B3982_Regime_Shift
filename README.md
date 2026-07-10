# Regime-Shift Tactical Asset Allocation Engine

This project implements a **Macro-Aware Tactical Asset Allocation Engine** that dynamically reshuffles a portfolio between Equities (SPY), Bonds (TLT), and Gold (GLD) by detecting hidden market regimes from price and volatility data.

---

## 💡 Core Strategy & Design Decisions

### 1. Why 3 Regimes?
We model the market using **3 hidden states** (Bull, Bear, and Crisis) representing distinct economic environments:
*   **Bull Regime (Green):** Calmly rising market characterized by positive momentum and low realized volatility.
*   **Bear Regime (Orange):** Flat or falling market characterized by negative momentum and moderate volatility.
*   **Crisis Regime (Red):** Severe market drawdown characterized by extreme volatility and sharp downward price action.

### 2. Feature Selection
The model uses a small, robust set of features describing the market's state:
*   **SPY 21-Day Volatility:** Standard close-to-close realized volatility, annualized. Volatility spikes are the most reliable discriminators of market stress.
*   **SPY 21-Day Momentum:** Cumulative log return. Distinguishes between calmly rising markets (Bull) and steadily dropping markets (Bear).
*   **VIX Index Level:** A well-known forward-looking proxy for market fear. Helps the model react quickly to implied volatility surges.

### 3. Z-Score Normalization
To prevent **lookahead bias**, all features are normalized dynamically using an **expanding window** z-score:
$$z_t = \frac{x_t - \mu_{[0:t]}}{\sigma_{[0:t]}}$$
This ensures that the feature scaling at time $t$ only uses information available up to time $t$.

### 4. Walk-Forward HMM Classification
We set up a time-series-correct **Expanding-Window Walk-Forward Validation**:
*   **Initial Training Window:** 1000 trading days (approx. 4 years).
*   **Rebalance Frequency:** 21 trading days (approx. 1 month).
*   At each step, the HMM is fit *only* on the training window. Test data is scaled using train parameters, and predicted using the Viterbi algorithm.
*   State labels are mapped dynamically based on mean volatility in the training set: lowest volatility is Bull, middle is Bear, and highest is Crisis.

### 5. Convex Portfolio Optimization (CVXPY)
Portfolio weights are optimized dynamically at each rebalance step:
*   **Bull Regime:** Maximize returns with low risk aversion: $\max w^T \mu - 1.5 w^T \Sigma w$.
*   **Bear Regime:** Maximize returns with moderate risk aversion and limit stock exposure: $\max w^T \mu - 5.0 w^T \Sigma w$ s.t. $w_{\text{SPY}} \le 30\%$.
*   **Crisis Regime:** Minimize portfolio variance and avoid stocks: $\min w^T \Sigma w$ s.t. $w_{\text{SPY}} \le 5\%$.
*   **Constraints:** Long-only ($w \ge 0$), fully invested ($\sum w = 1$).

---

## 📈 Performance Summary

Below is the out-of-sample backtest comparison (2012–2024) including a transaction cost of **10 bps per rebalance**:

| Metric | Strategy (HMM + CVXPY) | Static 60/40 SPY/TLT | Equal Weight SPY/TLT/GLD |
| :--- | :---: | :---: | :---: |
| **Total Return** | **89.78%** | 167.65% | 93.88% |
| **Annualized Return (CAGR)** | **5.72%** | 8.93% | 5.92% |
| **Annualized Volatility** | **10.45%** | 10.36% | 9.07% |
| **Sharpe Ratio** | **0.5478** | 0.8616 | 0.6530 |
| **Sortino Ratio** | **0.7896** | 1.0813 | 0.8849 |
| **Max Drawdown** | **-23.73%** | -27.24% | -22.72% |
| **Calmar Ratio** | **0.2412** | 0.3278 | 0.2606 |
| **Annual Turnover** | **1.8075** | 0.0000 | 0.0000 |

### 🔍 Key Insights
1.  **Drawdown Reduction:** The Regime-Shift Strategy successfully mitigated drawdown during major market crises (e.g., 2020 and 2022), achieving a **Max Drawdown of -23.73%** compared to **-27.24%** for the static 60/40 benchmark.
2.  **Underperformance vs. 60/40:** During the 2012-2024 period, the US stock market experienced a historic, persistent bull run. Because the regime-shift model actively reduced equity exposure during perceived Bear and Crisis periods, it dragged on cumulative return compared to a static, equity-heavy 60/40 portfolio.
3.  **Turnover & Costs:** The strategy has an annual turnover of 1.8075, resulting in approximately 18 bps of annual transaction costs.

---

## 🚀 How to Run the Code

### 1. Prerequisites
Ensure you have Python 3.9+ installed. The environment uses the following libraries:
```bash
pip install numpy pandas matplotlib yfinance scipy hmmlearn cvxpy nbconvert
```

### 2. Run the Notebook
The full pipeline (data download, feature engineering, walk-forward HMM, CVXPY optimization, backtesting, and plotting) is implemented in `Regime_Shift_Engine.ipynb`.
To run it from the command line:
```bash
jupyter nbconvert --to notebook --execute --inplace Regime_Shift_Engine.ipynb
```
Or open the notebook in Jupyter Lab/Notebook and run all cells.

### 3. Review Generated Artifacts
The execution will produce the following plots in the root directory:
*   `equity_curve.png`: Comparative growth of $1000.
*   `drawdowns.png`: Shaded drawdown comparison.
*   `regime_overlay.png`: SPY close price color-coded by the predicted HMM regime.
*   `asset_allocation.png`: Stacked area chart showing how the portfolio weights shifted over time.

