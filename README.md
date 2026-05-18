Readme egg price shocks · MD
Copy

# U.S. Egg Price Shock Detection & Attribution
 
A step-by-step econometric framework for detecting and explaining sudden price spikes in U.S. retail egg markets, applied to 45 years of monthly data (1980–2026).
 
## Overview
 
U.S. egg prices are prone to sudden, sharp spikes during supply disruptions — but isolating how much of each spike is caused by feed costs, avian flu outbreaks, or energy prices has proven difficult. This project introduces a transparent, reproducible framework to detect and attribute these shocks using an autoregressive baseline model, walk-forward out-of-sample residuals, and rolling z-score thresholding.
 
The framework is intentionally general: the same pipeline can be applied to any monthly commodity price series where understanding the causes of price spikes is of interest.
 
**Course:** DS 440 — Data Science Capstone, Penn State University (Spring 2026)  
**Authors:** Varsha Giridharan, Sree Penumuchu, Sahana Ramachandran, Advika Sadineni
 
---
 
## Key Findings
 
- **Seasonality is the single biggest fix.** Adding monthly seasonal dummies reduced detected shocks from 9 to 5 — a large share of apparent volatility was just predictable holiday and production-cycle patterns, not genuine disruptions.
- **Feed costs pass through fast.** A 10% unexpected increase in feed costs (BLS Feed PPI) is associated with a ~4.4% increase in egg prices within one month (elasticity: +0.436), consistent with the short passthrough window documented for minimally processed foods.
- **All detected shocks are upward.** Every anomaly across all model versions is a positive deviation — egg price disruptions are driven by supply contractions, not demand collapses.
- **Natural gas deteriorates model fit.** Despite a sound theoretical motivation, the gas covariate worsened performance because the 2008–2013 training window predates the transmission mechanism that became active after the 2022 Russia-Ukraine war — a cautionary result about fixed training windows.
- **HPAI flock losses add structure but don't fully explain peak episodes.** Feed costs and layer mortality together improve explanatory power but cannot account for the full magnitude of the 2015 and 2022–2025 shocks, suggesting multi-factor co-occurrence matters.
---
 
## Methodology
 
### Baseline Model (v0)
An AR(1) process fitted on 2008–2013 data. The coefficient is estimated once and held fixed — never re-estimated on the data it predicts — so that predictions are not contaminated by the shocks we are trying to detect.
 
```
log(price_t) = c + φ · log(price_{t-1}) + ε_t
```
 
### Model Versions
| Version | Covariates Added | Shocks Detected |
|---------|-----------------|-----------------|
| v0 | AR(1) only | 9 |
| v1 | + Monthly seasonal dummies | 5 |
| v2 | + Log Feed PPI (lag 1) | 5 |
| v3 | + Log layer losses (lag 1) | 6 |
| v4 | + Log natural gas price (lag 1) | 9 |
 
### Shock Detection
Residuals are standardized using a rolling 12-month z-score:
 
```
z_t = (ε_t − μ_t) / σ_t
```
 
A month is flagged as a shock when `|z| ≥ 2`. The rolling mean subtraction corrects for local drift without absorbing slow-building sustained increases into the baseline.
 
### Walk-Forward Evaluation
All residuals are computed out-of-sample via a walk-forward loop over the 2013–2026 evaluation window, ensuring the model never sees the data it is predicting.
 
---
 
## Data Sources
 
| Series | Source | Coverage |
|--------|--------|----------|
| Retail egg prices (Grade A Large) | FRED `APU0000708111` | 1980–2026 |
| Feed PPI (prepared animal feeds) | BLS `WPU02930102` | 1989–2026 |
| Henry Hub natural gas spot price | FRED `MHHNGSP` | 1997–2026 |
| Layer losses (HPAI flock mortality) | USDA NASS monthly surveys | 2013–2026 |
 
All series are log-transformed prior to modeling. Small gaps are filled by linear interpolation.
 
---
 
## Repository Structure
 
```
├── models.ipynb                          # All 5 model versions, residuals, shock detection
├── feed_cost_exploration_log_prices.ipynb  # Feed PPI & corn price EDA, lead-lag analysis
├── energy_egg_shocks.ipynb               # Natural gas covariate analysis
├── avian_flu_analysis_final.ipynb        # HPAI layer loss covariate analysis
├── AvianFluCovariate_datasets.ipynb      # USDA NASS data processing
└── Final_Report.pdf                      # Full paper with results and discussion
```
 
---
 
## Setup
 
```bash
pip install numpy pandas matplotlib statsmodels
```
 
All notebooks were developed in Google Colab. Data files are loaded from a shared Google Drive folder — update the file paths at the top of each notebook to match your local setup.
 
---
 
## Results Summary
 
The v1 model (AR(1) + seasonality) provides the cleanest shock detection baseline: 5 genuine anomalies corresponding to the 2015–2016 HPAI outbreak, the COVID-era supply disruption, and the sustained 2022–2025 price surge. The AR(1) persistence coefficient of 0.938 confirms strong month-to-month inertia — once a shock pushes prices above the baseline, they remain elevated for several months before gradually returning.
 
The framework is designed to be interpretable and extensible. Adding any new monthly covariate requires only a one-line change to the exogenous variable block and a re-run of the walk-forward loop.
 
---
 
## Limitations
 
- Fixed 2008–2013 training window may not capture structural shifts in price dynamics after 2020
- Rolling z-score normalization can dampen detection of slow-building sustained increases (e.g., 2022–2023)
- Attribution is correlational, not causal — co-occurring shocks cannot be cleanly separated
- Short 2013–2026 evaluation window limits confidence in conclusions about rare, large-magnitude events
