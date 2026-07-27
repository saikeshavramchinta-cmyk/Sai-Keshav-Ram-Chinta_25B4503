# Summer of Quant Advanced Project
# 📈 Regime-Shift: A "Smart" Asset Allocation Engine


In one simple sentence: **I built a program that looks at market data, figures out if the market is calm, falling, or in a full-blown crisis, and automatically reshuffles a portfolio (between stocks, bonds, and gold) to match the current mood**.

## 🤔 The Problem: Why are we doing this?
Most simple investment portfolios just pick fixed weights—say, 60% stocks and 40% bonds—and never change them. That works perfectly fine when markets behave normally. But what happens during a sudden crash? A fixed split falls apart because it has no way of knowing the world has changed. 

My goal was to build something smarter: a mathematical system that detects when the "regime" of the market shifts, and dynamically adjusts your investments to protect your capital.

---

## 🧠 The "Why": Key Design Decisions

I had to make several crucial choices to ensure this model was both mathematically sound and practical. Here is a peek under the hood at why I built it this way:

### 1. Keeping it Simple with 3 Regimes
I chose to train the Hidden Markov Model (HMM) to detect exactly **3 hidden states**[cite: 2]. Why? Because market behavior naturally clusters into these distinct categories:
*   **Bull:** The market is calmly rising[cite: 2].
*   **Bear:** The market is steadily falling[cite: 2].
*   **Crisis:** The market is experiencing violent, high-volatility moves[cite: 2].
*By sticking to just three regimes, we avoid statistical "overfitting." If we gave the model 10 regimes, it would get confused by daily noise. Three regimes give us clear, actionable signals.*

### 2. Choosing the Right Market "Thermometers" (Features)
A model is only as good as the data you feed it. Instead of just looking at raw prices, I engineered features that capture both the *direction* and the *uncertainty* of the market[cite: 2]:
*   **Momentum:** Calculated using rolling windows to see if the macro trend is pointing up or down[cite: 2]. 
*   **Volatility:** Calculated using the rolling standard deviation of returns to measure how wildly prices are swinging[cite: 2].
*   **Indian VIX:** A natural, forward-looking proxy for market fear[cite: 2].

### 3. Letting Stats Drive the Weights (Convex Optimization)
Instead of guessing how much of our money should be in stocks versus bonds, I used **convex optimization** to mathematically pick the best portfolio weights for whatever regime we are currently in[cite: 2]. 
*   In a **Bull** market, the math optimizes for maximum return[cite: 2]. 
*   In a **Crisis**, the objective function flips entirely to minimize volatility, automatically forcing a flight to safe assets[cite: 2]. 

### 4. Keeping the Math Honest (No Time-Traveling!)
The easiest trap to fall into when building financial models is **lookahead bias**—accidentally letting the model peek at future data[cite: 2]. To prevent the model from "cheating," I built a strict **walk-forward validation harness**[cite: 2]. The HMM is re-fit only using past data at every single step, moving forward through time[cite: 2]. This ensures our testing is grounded in reality.

### 5. Staying Grounded in Reality (Transaction Costs)
Trading isn't free. If a model rapidly flips between assets every two days, it might look great on paper, but the transaction costs will quietly destroy your real-world returns[cite: 2]. To reflect reality, I explicitly modeled a **5-10 basis point transaction cost** for every rebalance[cite: 2]. 

---

## 🛠️ The Tech Stack
*   **Python 3.9+**[cite: 2]
*   **yFinance:** To pull the daily returns for our assets and the VIX[cite: 2].
*   **hmmlearn:** The statistical brains behind detecting the hidden market regimes[cite: 2].
*   **CVXPY:** The optimization engine that solves for our exact portfolio weights[cite: 2].
*   **Pandas, NumPy, & Matplotlib:** For wrangling data and generating our visualizations[cite: 2].

---

## 🚀 How to Run It & Reproduce My Results

You don't need to be a math whiz to test this out! The entire pipeline—from downloading data to generating the final backtest results—runs top to bottom in one go[cite: 2].


📊 What You'll See in the Output
When you run the code, it will automatically output a few key things:

The Regime Chart: A plot of the historical price data with colored bands layered on top, showing exactly when the HMM thought we were in a Bull, Bear, or Crisis market[cite: 2].

The Transition Matrix: A matrix showing the probability of the market switching from one state to another (proving that market regimes tend to be "sticky")[cite: 2].

The Scorecard: A performance summary comparing our smart dynamic strategy against a static 60/40 portfolio and an equal-weight portfolio[cite: 2]. We compare them fairly using the Sharpe ratio, Sortino ratio, max drawdown, Calmar ratio, and turnover[cite: 2].
