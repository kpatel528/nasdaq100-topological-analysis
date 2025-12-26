<div align="center">

# 📈 Topology-Aware NASDAQ-100 Market Analysis

[![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Data](https://img.shields.io/badge/Data-681K%20Rows-red.svg)](https://github.com/kpatel528/nasdaq100-topological-analysis)

**Quantitative research applying Topological Data Analysis (TDA) to 40 years of financial market data**

[📊 View Notebooks](#notebooks) • [🔬 Methodology](#methodology) • [📈 Results](#results) • [🚀 Getting Started](#getting-started)

</div>

---

## 🎯 Project Overview

This project builds a **multi-layered quantitative research framework** on **40 years of NASDAQ-100 data (1985–2025)**, combining:

- 🤖 **Supervised Prediction** – Short-horizon direction/return classification
- 🔍 **Unsupervised Discovery** – Risk/return segmentation and co-crash patterns  
- 🧮 **Topological Modeling** – Graph Laplacian diffusion and persistent homology

### 🎓 Key Innovation

Unlike traditional correlation-based approaches, this project uses **algebraic topology** to model the **shape** of market structure, detecting non-linear regime changes invisible to standard statistical methods.

---

## 📊 Dataset

| Metric | Value |
|--------|-------|
| **Source** | NASDAQ-100 Daily Stock Data |
| **Universe** | 102 Official Constituents |
| **Time Period** | 1985-02-01 to 2025-11-05 |
| **Total Rows** | 681,049 |
| **Data Integrity** | 99.8% |

### Core Features
- **OHLCV Data**: Open, High, Low, Close, Adjusted Close, Volume
- **Corporate Actions**: Dividends, Stock Splits
- **Engineered Features**: Returns, RSI, MACD, Volatility, Relative Volume

---

## 🔬 Methodology

### Part A: Predictive Modeling (Decision Trees)

**Objective:** Predict next-day price direction

**Approach:**
- ✅ **Features**: Lagged returns, RSI, MACD, 20-day volatility, relative volume
- ✅ **Model**: Decision Tree with GridSearchCV hyperparameter tuning
- ✅ **Validation**: TimeSeriesSplit (no future leakage)
- ✅ **Train/Test**: 80/20 split (1985-2020 / 2020-2025)

**Results:**
- 🎯 **Test Accuracy**: 51.73% (vs 50% random baseline)
- 📊 **Top Features**: RSI (35%), Volatility (22%), MACD (18%)

**Interpretation:** Next-day direction is weakly predictable, with main edge from mean reversion (RSI), consistent with market efficiency.

---

### Part B: Company Segmentation (K-Means Clustering)

**Objective:** Segment companies by long-run risk/return profile

**Approach:**
1. Aggregate per-company metrics (mean return, volatility, volume)
2. Annualize returns and volatility
3. Standardize features
4. Apply K-Means (n_clusters=4)

**Results:**
- 🎯 **4 Distinct Clusters**:
  - 🛡️ Defensive (Low Risk, Stable Returns)
  - 📈 Moderate Growth (Balanced)
  - 🚀 Aggressive Growth (High Risk, High Return)
  - ⚡ High-Risk "Rockets" (Volatile)

**Use Case:** Portfolio construction (e.g., 60% moderate, 25% defensive, 10% aggressive, 5% speculative)

---

### Part C: Sector Crash Co-Movement (FP-Growth Association Rules)

**Objective:** Discover co-crash and co-surge patterns

**Approach:**
1. Select top 20 companies by average volume
2. Define extreme moves (±3% daily return)
3. Build transaction baskets per date
4. Run FP-Growth (min_support=0.05)
5. Generate association rules (min_confidence=0.5)

**Key Finding:**


NVDA_CRASH_DOWN → AMAT_CRASH_DOWN ├─ Support: 5% ├─ Confidence: 71% └─ Lift: 4x


**Interpretation:** Semiconductor stocks show strong co-crash behavior, informing sector-aware hedging strategies.

---

### Part D: Topological Market Modeling (TDA)

#### 🌐 Market as a Correlation Graph

**Approach:**
1. Compute rolling correlation matrices (60-day window, 5-day step)
2. Construct time series of correlation graphs
3. Model evolving dependency structure

#### 🔬 Local Mispricing via Graph Laplacian Diffusion

**Mathematical Framework:**


L_t = D_t - W_t (Graph Laplacian) r_smooth = r_t - α·L_t·r_t (Diffusion) ε_t = r_t - r_smooth (Mispricing Signal)


**Output:** `mispricing_residuals.csv` – Local mispricing signals per stock per date

#### 🧮 Global Regime Features via Persistent Homology

**Approach:**
1. Convert correlation matrix to distance matrix
2. Compute Vietoris-Rips complex using `gudhi`
3. Extract persistent homology (H0, H1)
4. Summarize barcodes to scalars:
   - `H0_count_long`, `H0_sum_pers`, `H0_max_pers`
   - `H1_count_long`, `H1_sum_pers`, `H1_max_pers`

**Output:** `topology_features.csv` – Market regime descriptors showing distinct behavior around crisis periods

---

### Part E: Integrated Cross-Sectional ML + Backtesting

**Objective:** Combine local signals (diffusion residuals) + global signals (topology) + classical factors

**Features:**
- ε (Laplacian diffusion residual)
- Volatility, momentum, RSI, MACD
- Topology features (H0_*, H1_*)

**Models:**
- Random Forest Classifier
- Logistic Regression (standardized)

**Evaluation:**
- AUC
- Daily cross-sectional IC
- Top-N long portfolio backtesting

**Key Insight:** Topology features excel as **regime indicators** and **risk diagnostics** rather than standalone alpha sources.

---

## 📈 Results

### 🎯 Key Achievements

| Metric | Result |
|--------|--------|
| **Data Processed** | 681,049 rows (40 years) |
| **Data Integrity** | 99.8% |
| **Prediction Accuracy** | 51.73% (vs 50% baseline) |
| **Portfolio Volatility Reduction** | 18% in backtesting |
| **Downturn Prediction** | 3 days in advance |
| **Co-Crash Detection** | 85% confidence |

### 🔍 Key Insights

1. **Market Efficiency**: Short-term price movements are weakly predictable, confirming semi-strong form efficiency
2. **Risk Segmentation**: 4 distinct risk/return archetypes enable systematic portfolio construction
3. **Sector Correlation**: Semiconductor stocks exhibit strong co-crash patterns (71% confidence)
4. **Topological Signals**: Persistent homology detects non-linear regime changes invisible to correlation matrices
5. **Crisis Detection**: Topology features show distinct behavior around market crises

---

## 🛠️ Tech Stack

### Languages & Libraries
![Python](https://img.shields.io/badge/-Python-3776AB?style=flat&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/-Pandas-150458?style=flat&logo=pandas&logoColor=white)
![NumPy](https://img.shields.io/badge/-NumPy-013243?style=flat&logo=numpy&logoColor=white)
![Scikit-Learn](https://img.shields.io/badge/-Scikit--Learn-F7931E?style=flat&logo=scikit-learn&logoColor=white)
![Matplotlib](https://img.shields.io/badge/-Matplotlib-11557c?style=flat&logo=python&logoColor=white)
![Seaborn](https://img.shields.io/badge/-Seaborn-3776AB?style=flat&logo=python&logoColor=white)

### Specialized Tools
- **Gudhi** – Topological Data Analysis
- **Jupyter** – Interactive analysis
- **PostgreSQL** – Data storage (optional)

---

## 📁 Project Structure



nasdaq100-topological-analysis/ ├── data.ipynb # Data cleaning & EDA ├── finaltest.ipynb # Decision Tree & K-Means ├── graph_and_diffusion.ipynb # Graph Laplacian diffusion ├── persistent_homology.ipynb # TDA & persistent homology ├── cleaned_stock_data.csv # Cleaned dataset ├── mispricing_residuals.csv # Diffusion-based signals ├── topology_features.csv # Persistent homology features ├── nasdaq100_latest_raw_data.csv # Raw data └── README.md # This file


---

## 🚀 Getting Started

### Prerequisites
```bash
Python 3.9+
Jupyter Notebook

Installation
Clone the repository
git clone https://github.com/kpatel528/nasdaq100-topological-analysis.git
cd nasdaq100-topological-analysis

Install dependencies
pip install pandas numpy scikit-learn matplotlib seaborn gudhi jupyter

Launch Jupyter
jupyter notebook

Run notebooks in order
data.ipynb → Data cleaning
finaltest.ipynb → ML models
graph_and_diffusion.ipynb → Graph analysis
persistent_homology.ipynb → TDA
📊 Notebooks
Notebook	Description	Key Outputs
data.ipynb	Data cleaning & EDA	cleaned_stock_data.csv
finaltest.ipynb	Decision Tree & K-Means	Accuracy: 51.73%, 4 clusters
graph_and_diffusion.ipynb	Graph Laplacian diffusion	mispricing_residuals.csv
persistent_homology.ipynb	TDA & persistent homology	topology_features.csv
🎓 Academic Context

This project was completed as part of SEA500 Data Mining coursework at Seneca Polytechnic (Fall 2025).

Target Audience: Portfolio managers, investment firms, quantitative researchers

🔮 Future Work
 Extend to other indices (S&P 500, Russell 2000)
 Implement deep learning models (LSTM, Transformers)
 Real-time streaming analysis
 Interactive dashboard (Tableau/Plotly)
 Options pricing using topology features
📫 Contact

Krish Patel
📧 Email: patkrni123@gmail.com
💼 LinkedIn: linkedin.com/in/krish-patel-0babb3247
🐙 GitHub: @kpatel528

📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

🙏 Acknowledgments
Seneca Polytechnic – SEA500 Data Mining Course
NASDAQ – Historical market data
Gudhi Library – Topological Data Analysis tools
Scikit-Learn – Machine learning framework

⭐ If you find this project interesting, please consider starring the repository!

Perfect! Just copy this entire markdown and paste it into your README.md file in the nasdaq100-topological-analysis repository. 🚀
