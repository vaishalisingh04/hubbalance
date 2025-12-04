# HubBalance

Forecast station-level Bluebikes demand in Boston and recommend light rebalancing moves with interpretable rationales. The repo centers on an offline Jupyter notebook that generates synthetic data, builds features, trains models, and surfaces actionable recommendations without any external downloads.

## Repo map
- `hubbalance_offline_demo.ipynb` — end-to-end, fully offline demo pipeline.
- `HubBalance_Final_Report.pdf` — written report with motivation, method, and results.
- `hubBalance.pptx` — presentation deck.
- `ProjectProposal_Vaishali.pdf` — original one-page proposal.

## Quickstart (offline)
- Requirements: Python 3 with `numpy`, `pandas`, `scikit-learn`, `lightgbm`, `matplotlib`, `seaborn`, `shap`.
- Launch the notebook: open `hubbalance_offline_demo.ipynb` and run all cells top-to-bottom. No network or external data are needed; all tables are synthesized in-memory.
- What it builds: 10,080 synthetic rows (12 stations × 35 days, hourly), 56 engineered features, and LightGBM models for pickups, drop-offs, and stockout risk.

## Pipeline inside the notebook
- **Synthetic city + stations**: `make_city_context` creates hourly weather, MBTA headways, 311-like activity, and event pulses; `simulate_station_timeseries` generates per-station pickups/drop-offs/availability with commute and weather effects.
- **Feature engineering**: time encodings (hour/dow cyclic, weekend), weather, transit, 311 intensity, event score, lags (1h/3h/6h) for pickups, drop-offs, and availability, plus station/neighborhood identifiers.
- **Train/test split**: first 80% of the timeline for training, last 20% for validation to avoid leakage.
- **Baselines**: persistence (last hour) and seasonal median (same day-of-week and hour).
- **Models**: LightGBM regressors for pickups/drop-offs and a classifier for stockout probability; SHAP explains per-station risk drivers.
- **Rebalancing logic**: forecasts one hour ahead, flags shortages/overflows, and prints suggested actions (deliver/remove/monitor) with top SHAP rationales.

##Results
- Pickups MAE 1.98 vs 3.31 (persistence) and 2.59 (seasonal median).
- Drop-offs MAE 1.43 vs 2.09 (persistence) and 1.61 (seasonal median).
- Stockout classifier: ROC-AUC 0.70, AUPRC 0.92; high precision when flagging risky stations.
- Example rationale: “MBTA headway lowers risk (-0.88); dropoffs_lag6 raises risk (+0.51); pickups_lag6 raises risk (+0.41)”.

## Taking it further
- Tune stockout thresholds, truck capacity constraints, and planning horizon for operations.
- Package the forecasting and recommendation logic behind a FastAPI/Streamlit service and deliver alerts to dispatcher tablets.
