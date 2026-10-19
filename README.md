# Revenue Forecasting & Price Sensitivity

Modelling how EV charging revenue responds to demand, pricing, and network growth, forecasting 2025 monthly revenue, and simulating the effect of price changes of +5% and −10%.

## The Question

Revenue is price × energy delivered. This project asks three things: what drives energy delivered, whether customers respond to price, and what a price change would do to 2025 income.

## Repo Structure

```
├── notebooks/        → full analysis notebook
├── output/           → daily_revenue.csv, monthly_revenue_forecast.csv, price_scenarios.csv
├── dashboard/        → Power BI .pbix file + page screenshots
├── slides/           → presentation deck (PPTX)
└── documentation/    → project documentation
```

## Data

The source is the `sessions` table of the EV charging dataset (294,024 sessions, January 2022 – December 2024), using `energy_kwh`, `price_per_kwh`, `total_cost`, `station_id` and the session timestamp. The raw file is not included in this repo. It is hosted as a Kaggle Dataset: **https://www.kaggle.com/datasets/sanyazuberi/revenue-dataset**.

Output files:
- `daily_revenue.csv` — the revenue time series: daily sessions, energy, revenue, active stations, average price, and per-station measures (1,096 days)
- `monthly_revenue_forecast.csv` — 2025 monthly baseline revenue forecast
- `price_scenarios.csv` — annual revenue under +5% and −10% price changes at four elasticity assumptions

## Key Findings

- **Revenue grew with the network, not with customer behaviour.** Annual revenue rose from £1.03M (2022) to £2.59M (2023) to £4.55M (2024), while active stations grew from 8 to 17 to 33. Demand per station stayed flat at about 13.9 sessions per station-day.
- **No price sensitivity could be detected.** Price rose 25% in 2023 and fell 8% in 2024, yet sessions per station-day barely moved (13.92, 13.88, 13.85). The two year-to-year elasticity estimates (−0.21 and +0.21) have opposite signs, so they are not a consistent response. At session level, price variation left after removing year and peak-hour effects has a correlation of −0.0016 with energy per session.
- **Price is rule-based, not random.** It has a yearly base level and a constant ~10% premium between 16:00 and 20:59. It does not differ by charger type, user type, payment method, day of week, or region (regional gaps disappear once year is controlled).
- **Strong weekly seasonality, no reliable yearly seasonality.** Weekends run about 27% below weekdays. Yearly seasonality was tested and dropped (RMSE 30.76 with it, 30.41 without).
- **2025 baseline revenue is £4.50M**, about 1.2% below 2024 (£4.55M), assuming 33 stations and the 2024 average price of £0.588/kWh.

## Forecast Model

Revenue = stations × kWh per station-day × price. Only the unpredictable part, daily kWh per station, is modelled, with Prophet. Stations and price are explicit assumptions, so the model does not extrapolate network expansion or price steps as a trend.

Validated on a held-out final 90 days (1,006 training days, 90 test days, all at 33 stations): RMSE 30.4 kWh per station-day (about 5% of the typical level) and MAPE 3.8%. Validation uses the actual price, so revenue error equals demand error by construction. The chosen model runs slightly low (under 2% per month) over the October–December test window.

## Price Scenarios

Demand scales by (1 + price change)^ε and revenue = demand × price. The measured elasticity (about zero) is the base case. The other values are assumptions shown as sensitivity, not findings.

| Elasticity (ε) | +5% price | −10% price |
|---|---|---|
| 0 (measured) | +5.0% (£4.73M) | −10.0% (£4.05M) |
| −0.5 | +2.5% | −5.1% |
| −1.0 | 0.0% | 0.0% |
| −1.5 | −2.4% | +5.4% |

Break-even is at ε = −1: if customers are less price-sensitive than that, a price rise increases revenue, and if more sensitive it reduces it. This dataset cannot say which side real customers fall on. Network size is the larger lever: each station earns about £136k a year at baseline, so a 5% price rise at zero elasticity (about £225k) is worth roughly 1.65 stations.

## Dashboard

Built in Power BI, three pages:
- **Revenue, Price & Demand:** revenue, average price, sessions per station-day, and active stations by year
- **2025 Forecast:** monthly baseline revenue forecast
- **Price Scenarios:** revenue change by elasticity and price scenario, plus the annual scenario table

See `dashboard/` for the `.pbix` file and page screenshots.

## Reproducing This Project

1. Download the raw dataset from the Kaggle link above.
2. Run the notebook in `notebooks/` from top to bottom in a fresh session. It writes the three CSVs in `output/`.
3. Open the `.pbix` in `dashboard/` and refresh the data sources if the CSV paths have changed.

Prophet's point forecasts are deterministic for fixed data and settings, but results can differ slightly across machines.

## Tools Used

Python (pandas, Prophet, scikit-learn) · Kaggle Notebooks · Power BI

## Limitations

- Elasticity rests on only three price levels, and the price steps coincide with network growth, so demand is compared per station.
- The dataset appears simulated (flat month-to-month seasonality, session durations capped at 15–60 minutes), so a measured elasticity of zero may reflect how the data was generated, not how real customers behave.
- Per-station energy drifts down across years (689, 653, 641 kWh per station-day), mainly through lower energy per session. This drift was not diagnosed, and the 2025 forecast carries it forward.
- The baseline holds the network at 33 stations. The network doubled each January for two years, so any further growth would raise revenue far more than any price change.
- The validation window covers only October–December 2024 and cannot speak to other months.
