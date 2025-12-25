# Topology-Aware NASDAQ‑100 Market Analysis

## 1. Project Overview

This project builds a multi-layered quantitative research framework on **40 years of NASDAQ‑100 data (1985–2025)**. It combines:

- **Supervised prediction** – short-horizon direction/return classification
- **Unsupervised structure discovery** – risk/return segmentation and co-crash patterns
- **Topological market modeling** – graph Laplacian diffusion and persistent homology

The goal is not just to “beat the market”, but to:

- Quantify how predictable short-term stock moves actually are
- Discover stable long-run structures in risk and correlation
- Model global market regimes using tools from **algebraic topology**

All analysis is implemented in **Python** using **Jupyter notebooks** and **CSV** data files.

---

## 2. Data

**Source:** NASDAQ‑100 daily stock dataset  
**Universe:** 102 official NASDAQ‑100 constituents  
**Period:** 1985‑02‑01 to 2025‑11‑05  
**Size:** ~681,049 rows, 11+ columns

Core columns:

- `Date` – trading day
- `Company` – ticker (e.g., AAPL, MSFT)
- `Open`, `High`, `Low`, `Close`
- `Adj_Close` – adjusted close
- `Volume`
- `Dividends`, `Stock_Splits`
- `Return` – daily log/percent return (engineered)

### 2.1 Data Cleaning

Key steps:

- Removed duplicate rows on (`Company`, `Date`) – none found.
- Checked for logical inconsistencies:
  - `Low > High` → 0 rows
  - Negative `Open` or `Volume` → 0 rows
- Kept zero‑volume days (~0.13%) to avoid artificial gaps in time series.
- Sorted data by (`Company`, `Date`) for correct lag/rolling calculations.
- Saved cleaned dataset as a CSV (e.g. `nasdaq100_cleaned_with_returns.csv`).

---

## 3. Project Goals

1. **Predictive:**  
   Can we predict short-horizon direction or outperformance using technical indicators and topological features?

2. **Descriptive (Risk/Return Structure):**  
   Can we cluster companies into stable risk–return archetypes for portfolio construction?

3. **Prescriptive (Risk Management):**  
   When one stock or sector crashes, which others tend to move with it?

4. **Topological:**  
   Can we model the **shape** of the market (via correlations) using algebraic topology, and use it as a regime indicator and feature set?

---

## 4. Methods

### 4.1 Feature Engineering (Per Company, Per Date)

From raw OHLCV data, the following features were engineered:

- **Daily Return (lagged)**
  - `Feat_Return_Lag1`: return at \( t-1 \)
  - `Feat_Return_Lag2`: return at \( t-2 \)

- **RSI (Relative Strength Index, 14-day)**
  - Computed from rolling gains/losses
  - Shifted: `RSI(t-1)` used to predict day \( t \)

- **MACD (12–26 EMA spread)**
  - \( \text{MACD} = \text{EMA}_{12}(\text{Adj\_Close}) - \text{EMA}_{26}(\text{Adj\_Close}) \), shifted

- **20-Day Volatility**
  - Rolling std of returns over 20 days: `Volatility_20`

- **Relative Volume**
  - `Vol_Rel = Volume / rolling_mean_20(Volume)`

- **Additional factors for ML:**
  - Momentum features (e.g., 20‑day and 60‑day cumulative returns)
  - Rolling volatility and other risk proxies

**Leakage prevention:** All features used to predict day \( t \) are computed from data up to \( t-1 \) and then shifted accordingly.

---

### 4.2 Part A – Short-Horizon Directional Prediction (Decision Tree)

**Task:**  
Predict next-day direction:

- Target:  
  \[
  y_t = \mathbb{1}(\text{Return}_t > 0)
  \]

**Setup:**

- Features: lagged returns, RSI, MACD, `Volatility_20`, `Vol_Rel`, etc.
- Train–test split:
  - Train: ~1985–2020 (80% of history)
  - Test: ~2020–2025 (20% of history)
- Model: `DecisionTreeClassifier` with hyperparameter tuning:
  - `max_depth` \(\in\) {3, 5, 7, 10}
  - `min_samples_leaf` \(\in\) {100, 500, 1000}
  - `criterion` \(\in\) {gini, entropy}
- Validation: `TimeSeriesSplit` (no future leakage)

**Result (example):**

- Test accuracy ≈ **51.73%** vs 50% random baseline
- Tree feature importances:
  - RSI ≈ 35%
  - Volatility_20 ≈ 22%
  - MACD ≈ 18%
  - Lagged returns + relative volume: remainder

**Interpretation:**  
Next-day direction is only very weakly predictable. The main edge comes from **mean reversion (RSI)**, consistent with empirical market efficiency.

---

### 4.3 Part B – Company Segmentation (K-Means Clustering)

**Goal:** Segment companies by long-run risk/return profile.

**Steps:**

1. Aggregate per-company:
   - Mean daily `Return`
   - Mean `Volatility_20`
   - Mean `Volume`

2. Annualize:

   - \( \text{Annual\_Return} = \text{mean\_daily\_return} \times 252 \)
   - \( \text{Annual\_Vol} = \text{mean\_vol\_20} \times \sqrt{252} \)

3. Standardize features:

   - `StandardScaler` on `[Annual_Return, Annual_Vol]`

4. Apply **K-Means**:

   - `KMeans(n_clusters=4, random_state=42, n_init=10)`

**Outcome:**

- 4 clusters, e.g.:
  - Defensive, Moderate Growth, Aggressive Growth, High-Risk “Rockets”
- 2D scatter: `Annual_Return` vs `Annual_Vol` colored by cluster + centroids.

**Use-case:**  
Cluster info can be used for **portfolio construction** (e.g., 60% moderate growth, 25% defensive, 10% aggressive, 5% speculative).

---

### 4.4 Part C – Sector Crash Co-Movement (FP-Growth Association Rules)

**Goal:** Discover co-crash and co-surge patterns among highly liquid names.

**Steps:**

1. Select **top 20** companies by long-run average `Volume`.
2. Define **extreme moves**:
   - Daily return > +3% → `TICKER_ROCKET_UP`
   - Daily return < −3% → `TICKER_CRASH_DOWN`
3. For each date, build a transaction (basket) of all extreme events for that day.
4. Run **FP-Growth**:
   - `min_support = 0.05`
5. Generate association rules:
   - Metric: `confidence`
   - `min_threshold = 0.5`

**Example rule:**

- `NVDA_CRASH_DOWN → AMAT_CRASH_DOWN`
  - Support ≈ 5%
  - Confidence ≈ 71%
  - Lift ≈ 4x

**Interpretation:**  
Semiconductor-related stocks show strong **co-crash** behavior, informing sector-aware hedging strategies.

---

### 4.5 Part D – Topological Market Modeling (Graph Laplacian + Persistent Homology)

This is the **algebraic topology** extension of the project.

#### 4.5.1 Market as a Correlation Graph

Using the daily `Return` matrix (`Date` × `Company`):

1. Compute **rolling correlation matrices**:
   - Window: ~60 trading days
   - Step: ~5 days

2. For each window, define a graph:

   - Nodes: stocks
   - Edges: pairwise correlations
   - Adjacency matrix \( W_t \) from correlations (e.g., thresholded at 0.3, non-negative)

3. This yields a time series of graphs \( G_t = (V, E_t) \), describing evolving dependency structure.

#### 4.5.2 Local Mispricing via Graph Laplacian Diffusion

For each graph snapshot:

1. Compute **graph Laplacian**:

   \[
   L_t = D_t - W_t
   \]

2. Define a **node signal** \( r_t \):

   - Recent cumulative return (e.g., last 5 days) for each stock.

3. Apply a diffusion step:

   \[
   r_{\text{smooth}} = r_t - \alpha L_t r_t
   \]
   \[
   \epsilon_t = r_t - r_{\text{smooth}}
   \]

4. Interpretation:

   - \( \epsilon_{t,i} \) is the **local mispricing signal** for stock \( i \) at time \( t \), relative to its neighbors in the correlation network.
   - Collected into `mispricing_residuals.csv` (Date × Company).

#### 4.5.3 Global Regime Features via Persistent Homology

To capture the **global topology** of the correlation structure:

1. Convert correlation matrix \( C_t \) to a distance matrix:

   \[
   d_{ij} = \sqrt{2(1 - \rho_{ij})}
   \]

2. Use **Vietoris–Rips complex** (via `gudhi`) to compute persistent homology up to H1:

   - H0: connected components
   - H1: loops/cycles

3. Summarize barcodes to scalars:

   For each dimension (H0, H1):

   - `count_long`: number of bars with persistence above a threshold
   - `sum_pers`: total persistence
   - `max_pers`: maximum persistence

4. Store results in `topology_features.csv`:

   - Columns: `H0_count_long`, `H0_sum_pers`, `H0_max_pers`, `H1_count_long`, `H1_sum_pers`, `H1_max_pers`
   - Index: `Date`

These features act as **market regime descriptors** (e.g., degree of clustering/fragmentation, complexity of loop structure), and show distinct behavior around crisis periods.

---

### 4.6 Part E – Integrated Cross-Sectional ML + Backtesting

**Goal:** Combine:

- Local signal: **diffusion residual** \( \epsilon_{t,i} \)
- Global signal: **topological features** at time \( t \)
- Classical factors: volatility, momentum, etc.

to perform cross-sectional stock selection.

**Primary setup:**

- Horizon: \( H = 5 \) trading days
- Forward returns:
  \[
  r^{(H)}_{t,i} = \frac{P_{t+H,i}}{P_{t,i}} - 1
  \]
- Index proxy: equal-weighted NASDAQ‑100
- Label variants:
  - Binary “outperform index over next H days”
  - (Optionally) top/bottom quantile labels (easier classification)

**Features per (Date, Company):**

- `epsilon` (from Laplacian diffusion)
- `Volatility_20`, momentum, etc.
- Topology features: `H0_*`, `H1_*` for that date

**Models:**

- Random Forest classifier
- Logistic Regression with standardized features (optionally)
- Evaluation:
  - AUC
  - Daily cross-sectional IC (correlation between scores and realized returns)
  - **Top‑N long portfolio**:
    - For each test date, buy top N stocks by predicted outperformance probability and hold for H days.

Results show that:

- Short-term alpha is weak (AUC slightly above random, realistic in equities).
- The pipeline is **methodologically correct**: no lookahead, proper time splits, and multi-level feature integration.
- Topology features are more useful as **regime indicators / risk diagnostics** than standalone alpha sources.

---

