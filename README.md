# Regime-Shift: Macro-Aware Tactical Asset Allocation Engine

## Overview
This repository contains a dynamic, macro-aware portfolio allocation system that detects hidden market regimes and automatically adjusts asset weights between Equities, Bonds, and Gold. By utilizing a Hidden Markov Model (HMM) and strict walk-forward validation, the engine intelligently navigates Bull, Bear, and Crisis markets without falling victim to lookahead bias.

---

## 1. Key Design Decisions

### Why 3 Regimes?
We modeled the market using **3 hidden states** (Bull, Bear, Crisis). Market behavior typically falls into these distinct categories:
*   **Bull:** Low volatility, steady upward momentum.
*   **Bear:** Moderate-to-high volatility, negative momentum.
*   **Crisis:** Extreme volatility, violent price swings, and panic.

Restricting the model to three states prevents overfitting and ensures that the optimizer receives clear, actionable signals rather than noisy, ambiguous micro-regimes.

### Why These Specific Features?
The HMM relies on features that capture both the *direction* and the *uncertainty* of the market:
*   **126-day Momentum:** Represents the 6-month macro trend using rolling windows. It helps the model understand the broader direction of the market rather than getting faked out by weekly noise.
*   **21-day Realized Volatility:** Captures the immediate, 1-month realized uncertainty of the market using a rolling standard deviation of returns.
*   **VIX (India VIX):** A forward-looking measure of market fear and implied volatility.
*   *Note on Smoothing:* We applied a 21-day Simple Moving Average (SMA) to these features. This acts as a low-pass filter, preventing the HMM from rapidly oscillating between states due to daily data spikes.

### Portfolio Optimization Logic
Instead of a static 60/40 allocation, we use **Convex Optimization (`cvxpy`)** with strict, regime-specific constraints to mathematically pick the best portfolio weights[cite: 2]:
*   **Bull Regime:** Maximizes risk-adjusted return with a low risk penalty ($\gamma = 0.5$) and a hard constraint to hold at least 60% equities.
*   **Bear Regime:** A defensive posture with a higher risk penalty ($\gamma = 2.0$) that caps equities at 40% and requires at least 20% in bonds.
*   **Crisis Regime:** Solves for the Global Minimum Variance portfolio to minimize volatility[cite: 2]. Equities are capped at a strict 10%, forcing a flight to safety (Bonds $\ge$ 40%, Gold $\ge$ 20%).

### Avoiding Lookahead Bias
The backtest uses a **Strict Walk-Forward Harness**[cite: 2]. The HMM is trained on an initial 3-year window and steps forward 21 days at a time, being re-fit only on past data at every step[cite: 2]. Feature scaling (Z-scoring) and transition matrices are *only* calculated using past data. The regime assigned to "today" never has access to "tomorrow's" data.

---

## 2. Installation & Setup

### Prerequisites
Ensure you have **Python 3.9+** installed[cite: 2]. The project relies on the following libraries[cite: 2]:

```bash
pip install numpy pandas matplotlib yfinance hmmlearn cvxpy scikit-learn scipy
