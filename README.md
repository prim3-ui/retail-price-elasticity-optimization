# Price Elasticity & Revenue Optimization — UK E-Commerce Retailer

Estimating real price elasticity of demand from transaction-level data and translating it into bounded, business-realistic pricing recommendations with Monte Carlo uncertainty quantification.

## Business Problem

Most SKUs in a retail catalog are priced by habit, not by measured demand response. This project asks: **for products with enough real price variation in their sales history, what does the data say about their actual price elasticity — and what would a disciplined, risk-aware pricing recommendation look like?**

## Data

Real UK e-commerce transaction data (Dec 2010 – Dec 2011), ~300K line items across ~3,900 SKUs. This is a cleaned sample of the widely-used UCI/Kaggle "Online Retail" dataset. No synthetic data — every price and quantity figure is a real recorded transaction.

## Method

1. **Weekly aggregation per SKU** — daily transaction noise averaged into a clean weekly panel (price, quantity).
2. **Filtering** — only SKUs with ≥20 weeks of data and genuine price variation (CV > 4%) are eligible; elasticity cannot be estimated where price never moved.
3. **Log-log OLS regression per SKU**, controlling for month (seasonality): `ln(Quantity) ~ ln(Price) + month dummies`. The coefficient on `ln(Price)` is the estimated elasticity.
4. **Confidence flagging** — estimates are tagged high/medium/low confidence based on magnitude and stability; extreme, small-sample-driven estimates (|e| > 6) are excluded from headline numbers and reported separately rather than trusted at face value.
5. **Price optimization (Lerner rule)** — using a disclosed, benchmark-grounded gross margin assumption (45–55%, typical UK general/gift retail), computes the profit-maximizing price for each elastic SKU.
6. **Constraints, not blind extrapolation** — recommended prices are bounded to (a) the price range actually observed in the data and (b) a ±20% move cap, mirroring how real pricing teams test incrementally rather than shipping untested 40%+ price swings.
7. **Monte Carlo simulation (10,000 draws)** — propagates uncertainty in both the estimated elasticity (regression standard error) and the assumed margin, producing a *distribution* of recommended prices and revenue/profit impact rather than a false-precision point estimate.

## Key Results

- 34 SKUs showed statistically significant, economically sensible (negative) elasticity estimates
- Across the 31 high/medium-confidence SKUs, revenue-weighted median revenue uplift potential: **~61%**, profit uplift: **~32%** — via bounded, incremental price moves, not extreme repricing
- 3 SKUs flagged as low-confidence (elasticity estimates too extreme/unstable to act on with one year of data) and explicitly excluded from headline numbers

## Repository Structure

```
01_data_prep_elasticity.py       # data cleaning, weekly panel, per-SKU elasticity regression
02_price_optimization_montecarlo.py   # Lerner-rule optimization + Monte Carlo uncertainty
03_visualizations.py             # chart generation
METHODOLOGY.md                   # full transparency on assumptions and limitations
charts/                          # output visualizations
```

## Honest Limitations

- One year of data limits confidence in the most extreme elasticity estimates — flagged, not hidden
- No true cost data exists in this public dataset; margin is an explicit, disclosed assumption, not a hidden fabrication
- Constant-elasticity demand curves are a local approximation, which is why price recommendations are bounded rather than extrapolated freely
- Single retailer, UK-only — findings are not claimed to generalize beyond this business

See `METHODOLOGY.md` for full detail.
