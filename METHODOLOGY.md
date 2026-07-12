# Methodology

## 1. Data Source and Honesty Note

The dataset used is real transaction data from a UK-based online retailer (Dec 2010 – Dec 2011), a cleaned sample derived from the widely-used UCI/Kaggle "Online Retail" dataset. Every price, quantity, and revenue figure in this project is a real recorded transaction — nothing in the underlying data is simulated.

Simulation is used only in one place: the Monte Carlo layer that propagates *uncertainty* around model estimates (elasticity standard error, assumed margin range). This is standard practice for quantifying confidence in a model output — it is not a substitute for real data.

## 2. Why Weekly Aggregation

Daily transaction counts per SKU are noisy and sparse. Aggregating to weekly price (revenue-weighted average) and quantity gives a cleaner signal for regression while retaining enough time periods (up to 53 weeks) to estimate a trend.

## 3. Why Log-Log Regression

`ln(Q) = α + β·ln(P) + month fixed effects + ε`

β is directly interpretable as the price elasticity of demand — the percentage change in quantity for a 1% change in price. This is the standard, textbook-defensible approach for elasticity estimation and is easy to explain and defend in an interview, unlike more opaque ML approaches.

Month fixed effects control for seasonality (this dataset spans a UK Christmas selling season, which otherwise would confound price and demand trends).

## 4. Filtering Criteria and Why

A SKU is only included if:
- ≥20 weeks of sales data (enough periods for a stable regression)
- Price coefficient of variation > 4% (elasticity is mathematically unidentifiable if price never moved)
- Average price > £0.50 (excludes near-zero-price data artifacts)

This is a **necessary honesty step**: most SKUs in the full dataset don't have enough price variation to say anything reliable about their elasticity. Rather than force an estimate everywhere, the analysis is restricted to the ~1,800 SKUs where estimation is actually valid, and further to the top 40 by revenue for the optimization layer.

## 5. Confidence Flagging

Elasticity magnitude thresholds:
- |e| ≤ 4: high confidence
- 4 < |e| ≤ 6: medium confidence
- |e| > 6: low confidence — flagged and excluded from headline portfolio metrics

This threshold is a judgment call, not a statistical law — the reasoning is that very large elasticity magnitudes on a single year of weekly data are more likely to reflect a handful of influential weeks (e.g. a stockout or promotion) than a stable true demand response. This is disclosed explicitly rather than presented as certainty.

## 6. Price Optimization — Lerner Rule

Given a demand curve with constant elasticity e and marginal cost C, the profit-maximizing price is:

```
P* = C × (e / (e + 1))     [valid for e < -1]
```

This is a standard result in pricing economics (the "Lerner markup rule"), not a custom heuristic.

**Cost assumption**: this public dataset contains no cost data. Rather than invent a number, the analysis uses a disclosed, published-benchmark gross margin range for UK general/gift retail (45–55%) to back into an implied unit cost: `C = P_current × (1 − margin)`. This assumption is stated openly in every output and is treated as a live input to the Monte Carlo simulation (i.e., its uncertainty is propagated, not hidden behind a single number).

## 7. Why Recommendations Are Bounded, Not Extrapolated

An earlier version of this analysis let the Lerner rule recommend prices freely, which produced revenue uplift estimates in the hundreds or thousands of percent for the most elastic SKUs. This is a textbook extrapolation failure: a constant-elasticity curve fit on a limited price range is not a reliable guide to demand at prices the business never actually charged.

Two constraints were added:
1. Recommended prices are clipped to the price range **actually observed** in the data for that SKU.
2. Recommended prices are additionally capped at a **±20% move** from the current price, reflecting how pricing teams in practice test changes incrementally rather than shipping an untested 40%+ swing off a one-year model.

This produces smaller, more defensible uplift numbers — which is the point. A number a hiring team can trust is more valuable than a bigger number they'd immediately distrust.

## 8. Monte Carlo Simulation

For each SKU, 10,000 draws are generated:
- Elasticity ~ Normal(estimated β, its regression standard error)
- Margin ~ Uniform(45%, 55%)

Each draw produces an optimal price, resulting quantity (from the constant-elasticity demand curve), and revenue/profit outcome. Reporting the 10th/50th/90th percentiles (rather than a single number) makes the genuine uncertainty visible instead of implying false precision.

## 9. Known Limitations (Stated Openly)

- One year of data; elasticity could shift with more history or across economic conditions
- No true cost data; margin is an assumption, clearly disclosed
- Single UK retailer; not claimed to generalize to other markets or categories
- Constant-elasticity functional form is an approximation, not a guaranteed true demand shape
- Cross-price effects between SKUs (substitution/complementarity) are not modeled
