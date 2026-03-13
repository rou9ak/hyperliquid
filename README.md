# Trader Performance vs Market Sentiment — Primetrade.ai Assignment

**Author:** Rounak Paul  
**Completed:** March 14, 2026

---

## Objective

Analyze how Bitcoin market sentiment (Fear/Greed Index) relates to trader behavior and performance on Hyperliquid. The goal is to surface actionable patterns that could inform smarter trading strategies.

---

## Project Structure

```
.
├── data/
│   └── raw/
│       ├── fear_greed_index.csv       # Bitcoin Fear/Greed Index (2018–2025)
│       └── historical_data.csv        # Hyperliquid trader data (211,224 trades)
├── analysis.ipynb                     # Main analysis notebook
├── app.py                             # Streamlit dashboard
├── merged_daily.csv                   # Processed: daily trader stats merged with sentiment
├── sentiment_stats.csv                # Aggregated stats by sentiment classification
├── account_segments.csv               # Per-account segments (risk, frequency, winner type)
├── account_segments_with_clusters.csv # K-Means cluster labels per account
└── README.md
```

**Output charts (saved during notebook run):**
- `pnl_winrate_by_sentiment.png`
- `behavior_by_sentiment.png`
- `segment_sentiment_pnl.png`
- `heatmap_weekday_sentiment.png`
- `timeseries_pnl_sentiment.png`
- `trade_size_distribution.png`
- `feature_importances.png`
- `elbow_method.png`

---

## Setup & How to Run

### Requirements

```bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn streamlit plotly
```

### Run the Notebook

```bash
jupyter notebook analysis.ipynb
```

Run all cells top to bottom. Outputs (CSVs, charts) will be saved in the working directory.

### Run the Streamlit Dashboard (Bonus)

```bash
# First run the notebook to generate merged_daily.csv, then:
streamlit run app.py
```

---

## Datasets

| Dataset | Source | Rows | Key Columns |
|---|---|---|---|
| Fear/Greed Index | [Google Drive](https://drive.google.com/file/d/1PgQC0tO8XN-wqkNyghWc_-mnrYv_nhSf/view?usp=sharing) | 2,644 | date, value, classification |
| Hyperliquid Trades | [Google Drive](https://drive.google.com/file/d/1IAfLZwu6rJzyWKgBToqwSmmVYU6VbjVs/view?usp=sharing) | 211,224 | Account, Coin, Execution Price, Size USD, Side, Closed PnL, Timestamp |

**Overlap window:** March 2023 – May 2025 (6 matching days in the filtered sample, 85.7% retention after inner join).

---

## Methodology

### Part A — Data Preparation

- Loaded both datasets and inspected shape, missing values (none found), and duplicates.
- Parsed Hyperliquid's `Timestamp` column (milliseconds → UTC datetime), then normalized to date for daily alignment.
- Parsed Fear/Greed `date` column and sorted chronologically.
- Built daily per-account metrics: `daily_pnl`, `trade_count`, `avg_trade_size`, `win_rate`, `long_short_ratio`.
- Merged on `date` (inner join) yielding 77 account-day records across 6 sentiment-matched dates.

### Part B — Analysis

**Performance by sentiment (Fear vs Greed):**

| Sentiment | Mean Daily PnL | Median Daily PnL |
|---|---|---|
| Fear | $209,373 | $81,390 |
| Greed | $99,676 | $35,988 |
| Neutral | $19,843 | ~$0 |
| Extreme Greed | $35,393 | $0 |

A Mann-Whitney U test confirms the Fear vs Greed PnL difference is statistically significant (p = 0.0356).

**Trader behavior by sentiment:**  
Trade frequency, average trade size, and long/short ratio all vary across sentiment regimes. Traders tend to be more active and take larger positions on Fear days — likely contrarian or momentum-driven.

**Trader segments (3 defined):**
- **Risk segment:** Large Sizes vs Small Sizes (split at median `avg_trade_size`)
- **Frequency segment:** Frequent vs Infrequent (split at median `days_active`)
- **Winner segment:** Consistent Winners (win rate ≥ 50% + positive total PnL) vs Others

**K-Means clustering (k=4)** was used to derive behavioral archetypes from `total_pnl`, `avg_trade_size`, `total_trades`, and `avg_win_rate`. Optimal k was selected via the elbow method.

### Part C — Insights

**Insight 1 — Fear days produce higher PnL (p < 0.05).**  
Traders on Fear days outperform Greed days on both mean and median PnL. This is consistent with contrarian strategies being more effective during panic-driven mispricings.

**Insight 2 — Trade size increases under Fear, not Greed.**  
Large-size traders post significantly better outcomes on Fear days, suggesting that experienced traders deploy more capital precisely when sentiment is negative.

**Insight 3 — Win rate is more stable than PnL across sentiments.**  
While absolute PnL swings are large, win rates are relatively consistent, meaning sentiment primarily affects trade magnitude (how much you make/lose per trade), not directional accuracy.

---

## Strategy Recommendations

**Strategy 1 — Fear is an entry signal for high-conviction traders.**  
During Fear days, consistent winners (win rate ≥ 50%, positive cumulative PnL) should consider increasing position sizes relative to their baseline. The data shows this segment extracts the most value from fear-regime volatility.

**Strategy 2 — Reduce activity during Extreme Greed.**  
Extreme Greed days show the lowest mean PnL and near-zero median PnL in this dataset. Infrequent traders in particular should consider sitting out or reducing trade frequency during Extreme Greed, as the edge appears thinnest in euphoric market conditions.

---

## Bonus

- **Predictive model:** A Gradient Boosting Classifier trained on `avg_trade_size`, `trade_count`, `win_rate`, `long_short_ratio`, and `sentiment_enc` achieves **81% accuracy** on held-out data (F1: 0.87 for profitable days).
- **Clustering:** K-Means (k=4) identifies four behavioral archetypes, surfacing distinct risk/reward profiles across the trader population.
- **Dashboard:** `app.py` provides an interactive Streamlit interface with PnL distribution histograms and a risk vs win-rate scatter plot, filterable by sentiment.
