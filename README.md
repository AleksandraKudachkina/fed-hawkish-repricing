# Project 01 — When the Fed Turns Hawkish
### Cross-Asset Repricing After Monetary Policy Shocks

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)]()
[![Pandas](https://img.shields.io/badge/pandas-2.0%2B-orange)]()
[![Status](https://img.shields.io/badge/Status-Complete-brightgreen)]()

## Overview

This project investigates how major asset classes respond when markets 
unexpectedly reprice the expected Federal Reserve policy path higher 
("hawkish shocks"). Using 30+ years of FOMC meeting data (1994–2024), 
we measure policy surprises via daily changes in the 2-Year Treasury 
yield and estimate their impact on equities, the US Dollar, gold, 
Bitcoin, oil, and high-yield credit across multiple time horizons 
(1, 5, 20, and 60 trading days).

**Key question**: Do major asset classes react consistently to hawkish 
Fed surprises, and does this reaction persist over time?

---

## Data Sources

| Source | Data | Frequency |
|---|---|---|
| FRED API | 2Y/10Y Treasury yields, 10Y TIPS real yield, Fed Funds Rate, VIX, CPI | Daily / Monthly |
| Yahoo Finance | S&P 500, Nasdaq, DXY, Gold futures, Bitcoin, Oil futures, HYG (high-yield bond ETF) | Daily |
| Manual compilation | FOMC meeting dates (1994–2024) | Event-based |

---

## Methodology

1. **Shock measure**: For every FOMC meeting date, we compute the change 
   in the 2Y Treasury yield on that day (in basis points). This serves 
   as a continuous proxy for how much the market's expectations about 
   the Fed's rate path were revised — positive values indicate a hawkish 
   surprise, negative values a dovish one.

2. **Asset response**: For each shock, we measure forward cumulative 
   log-returns for each asset over 1, 5, 20, and 60 trading days.

3. **Regression**: We estimate 
   `Asset Return = alpha + beta * Yield_Shock + error` 
   using OLS with HAC (Newey-West) standard errors, which correct for 
   autocorrelation arising from overlapping return windows between 
   closely-spaced FOMC meetings.

4. **Sample**: 170 FOMC meetings (1994–2024), with asset-specific sample 
   sizes varying based on each instrument's trading history (e.g., 
   Bitcoin only from ~2014).

---

## Key Findings

| Asset | Direction | Strongest horizon | Statistical significance |
|---|---|---|---|
| **DXY (Dollar)** | Positive (strengthens) | All horizons | ✅ Strong (p<0.01 at every horizon) |
| **Gold** | Negative (weakens) | 1-day, 5-day | ✅ Strong short-term (p<0.01), fades by 20d |
| **Nasdaq** | Negative (weakens) | 1-day | ✅ Significant only at 1d (p=0.03) |
| **SP500** | Negative (weakens) | 1-day | ⚠️ Marginal (p=0.07) |
| **HYG (credit proxy)** | Negative (price falls) | 1-day | ⚠️ Marginal (p=0.04) |
| **Oil** | Mixed, near zero | — | ❌ Not significant at any horizon |
| **BTC** | Mixed, near zero | — | ❌ Not significant at any horizon |

### 1. The Dollar (DXY) shows the cleanest, most reliable reaction

Across every horizon tested (1d through 60d), DXY strengthens 
significantly following hawkish surprises (beta ranges from +0.0004 to 
+0.0006, p-values consistently below 0.01). This is the textbook 
result: a hawkish Fed repricing raises expected US rate differentials 
relative to other currencies, attracting capital flows into the dollar. 
This is the strongest and most economically intuitive finding in the 
entire study.

### 2. Gold reacts fast, but the effect fades within a month

Gold shows a statistically strong negative reaction on the day after 
a hawkish shock (beta=-0.0013, p<0.001), consistent with the theory 
that rising real yields increase the opportunity cost of holding a 
non-yielding asset. However, by the 20-day horizon (p=0.185), this 
relationship loses statistical significance — suggesting the initial 
repricing is absorbed relatively quickly, likely offset by other 
drivers (inflation-hedging demand, geopolitical risk) over longer 
periods.

### 3. Equities react, but the signal is fragile and short-lived

Nasdaq shows a statistically significant negative reaction one day 
after a hawkish shock (beta=-0.0005, p=0.03), consistent with higher 
discount rates compressing growth-stock valuations. The S&P 500 shows 
the same direction but falls just short of conventional significance 
(p=0.07). By the 20–60 day horizon, neither index shows a reliable 
effect — likely because daily equity returns are dominated by many 
other concurrent news flows (earnings, geopolitics, other macro 
releases) that dilute the pure monetary policy signal over longer 
windows.

### 4. Credit markets show early, modest stress

HYG (used here as a price-based proxy for high-yield credit risk) 
falls significantly the day after a hawkish shock (beta=-0.0006, 
p=0.04), consistent with rising perceived default risk when policy 
tightens unexpectedly. Note: this measures the *price* of the credit 
ETF, which moves inversely to the underlying credit spread — a price 
decline in HYG corresponds to a widening spread (i.e., a rising credit 
risk premium). A cleaner future iteration of this analysis would 
directly use the ICE BofA High Yield Option-Adjusted Spread series 
(available via FRED) rather than the ETF price, pending resolution of 
a data-merging issue encountered during this project (see Limitations).

### 5. Bitcoin shows no reliable relationship with Fed policy surprises

Across all horizons, BTC's beta coefficients are statistically 
indistinguishable from zero (p-values all above 0.3). With only 82 
usable observations (reflecting Bitcoin's shorter trading history), 
this project cannot confirm or reject either the "digital gold" or 
"risk asset" hypothesis for BTC's behavior around Fed policy shocks — 
the data is simply too noisy and the sample too limited at a daily 
frequency to detect a clear signal, if one exists.

### 6. Oil shows no consistent relationship

Oil's beta coefficients fluctuate in sign across horizons and remain 
statistically insignificant throughout (p-values all above 0.13). 
This is consistent with oil prices being driven predominantly by 
supply/demand fundamentals and geopolitical factors rather than US 
monetary policy surprises specifically.

---

## Limitations

1. **Daily-data granularity**: Modern Fed communication (forward 
   guidance) means markets often anticipate policy decisions before 
   the meeting itself, leaving limited "surprise" visible in daily 
   close-to-close yield changes. Intraday data around the exact 
   announcement window would likely reveal a cleaner signal.

2. **Regime heterogeneity**: Pooling 1994–2024 combines fundamentally 
   different macro environments — disinflationary hiking cycles 
   (2004–2006) vs. the aggressive 2022 inflation-fighting cycle — 
   which may partially offset each other when averaged together.

3. **Overlapping windows**: FOMC meetings occur roughly every 6–8 
   weeks, so 60-day forward return windows can overlap between 
   consecutive events. HAC standard errors partially correct for 
   this, but do not fully eliminate potential distortion at the 
   longest horizon.

4. **Sample size for newer assets**: BTC (N=82) and HYG (N=139) have 
   meaningfully fewer usable observations than legacy assets like 
   SP500 (N=170), simply because they didn't exist for the full 
   study period. This limits statistical power specifically for 
   these two assets.

5. **Credit spread data**: An attempt to use the direct ICE BofA High 
   Yield Option-Adjusted Spread (rather than the HYG ETF price) 
   encountered a data-merging issue that reduced the usable sample to 
   only 11 observations — too small to draw reliable conclusions. This 
   is flagged as a known issue for future refinement rather than 
   included in the final results.

---

## Conclusion

This project demonstrates a complete, honest empirical research 
pipeline: data engineering across multiple sources (FRED + market 
data), event/shock identification anchored to FOMC dates, regression-based 
hypothesis testing with robust standard errors, and transparent 
reporting of both strong findings (DXY, Gold short-term) and weak/null 
results (BTC, Oil, longer equity horizons). The directional consistency 
of nearly all assets with established monetary policy transmission 
theory — even where statistical significance is marginal — supports 
the underlying economic mechanism, while highlighting the practical 
limitations of daily-frequency event studies in isolating clean policy 
surprises from broader market noise.

---

## Repository Structure
fed-hawkish-repricing/
├── data/
│ └── (raw data pulled live via API — not stored in repo)
├── notebooks/
│ └── 01_full_analysis.ipynb
├── outputs/
│ └── figures/
│ ├── 2y_yield_with_shocks.png
│ └── beta_heatmap.png
└── README.md

---

## How to Reproduce

1. Get a free FRED API key at https://fred.stlouisfed.org/docs/api/api_key.html
2. Install dependencies:
   ```bash
   pip install pandas numpy matplotlib seaborn fredapi yfinance statsmodels
   Set your API key in the notebook's Section 0
Run all cells sequentially


Tools Used
Python, pandas, statsmodels (OLS regression with HAC standard errors),
FRED API, yfinance, matplotlib, seaborn

Relevance
This project demonstrates skills directly relevant to Macro Research,
Global Markets, Market Risk, and Portfolio Analytics roles: monetary
policy transmission analysis, event study methodology, cross-asset
correlation analysis, and robust statistical inference under real-world
data limitations.
