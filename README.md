# Starting Finance Fund - Risk Analysis

Risk analysis of the Starting Finance club fund. The notebook downloads market data from Yahoo Finance for the positions that have a ticker, converts everything to EUR and computes returns, volatility, correlations, maximum drawdown, Value at Risk (VaR) and Conditional VaR (CVaR).

## Scope

- **Covered:** equities, ETFs and ETPs with a Yahoo Finance ticker, about 57% of the fund capital (EUR 7.38M).
- **Not covered:** bonds (no ticker, so no price history), the physical silver ETC and the Ethereum ETP (no data on Yahoo Finance), and the cash balance. They are excluded from every return-based metric.

## Method

1. **Data.** Daily adjusted close prices from 2023-01-01, forward-filled over market holidays. Tickers without data are dropped.
2. **Conversion to EUR.** The listing currency of each ticker is read from Yahoo Finance (23 of the 40 listings are in USD). Prices in pence are divided by 100, and every non-EUR series is divided by the daily FX rate, so returns include the currency effect.
3. **Returns.** Daily log returns per asset, with mean, standard deviation, variance, correlation and covariance.
4. **Portfolio returns.** Current weights, assumed constant over the whole history, so the results are a backtest of today's allocation and not the fund's actual past performance. Weights are rescaled to 100% over the covered assets. Because some ETFs are recent, weights are rescaled day by day over the assets that have data, and days where less than 80% of the weight has data are dropped.
5. **Risk measures.**
   - Volatility from the portfolio returns and from the covariance matrix, `sqrt(w' Cov w)`, annualised with 252 trading days.
   - Maximum drawdown from the equity curve.
   - Historical VaR and CVaR at 95% confidence, 1-day horizon, in percent and in EUR on the covered value.

## Results

| Metric | Value |
|---|---|
| Period | 2023-01-04 to 2026-11-11 (996 trading days) |
| Covered value | EUR 7.38M (56.6% of fund capital) |
| Annualised return | 14.9% |
| Annualised volatility | 13.0% (portfolio returns), 13.3% (covariance matrix) |
| Maximum drawdown | -18.3% (peak 2025-02-10, trough 2025-04-07) |
| 1-day VaR 95% | 1.20% (about EUR 88k) |
| 1-day CVaR 95% | 1.81% (about EUR 132k) |

### Observations

- **Diversification.** The weighted average of the single-asset daily volatilities is about 1.6%, while the portfolio daily volatility is about 0.8%, roughly half. Diversification works, but it is uneven.
- **Correlation clusters.** The heatmap shows tight clusters: the US equity trackers (S&P 500 cap-weighted and equal-weight, small caps, Nasdaq 100) with correlations of roughly 0.7 to 0.9, the gold group (physical gold, gold miners, Franco-Nevada), the two uranium ETFs and the software names. Holding several trackers of the same index adds little diversification.
- **Low-correlation assets.** The floating-rate bond ETF, the Bitcoin ETP, Sealsq and Emerald Resources have low correlation with the rest of the portfolio.
- **Risk is not proportional to weight.** Ranking assets by weight times volatility (which ignores correlations), the Bitcoin ETP is about 6% of the covered weight but about 11% of this standalone risk. Bitcoin, DRAM, the two uranium ETFs, gold miners and Sealsq together are about 15% of the weight and about 30% of the standalone risk.
- **Drawdown.** The maximum drawdown of -18.3% happened between February and April 2025, and the portfolio went back above its previous peak by autumn 2025.
- **Tail risk.** CVaR is about 1.5 times VaR, so on the worst 5% of days the average loss is about 50% larger than the VaR threshold.

## Conclusion

On the covered 57% of the fund, today's allocation had an annualised volatility of about 13% and an annualised return of about 15% since January 2023, with a maximum drawdown of -18.3% in early 2025. On a typical day the loss exceeds 1.2% (about EUR 88k) on 5% of days, and the average loss on those days is 1.8% (about EUR 132k).

The portfolio is well diversified across asset types: its volatility is roughly half the weighted average of its components. Diversification is weaker inside the equity ETFs, which are strongly correlated, and risk is concentrated in a small group of volatile positions (crypto, thematic ETFs, gold miners) that weigh about 15% of the covered portfolio but about 30% of its standalone risk.

These figures describe only the equity, ETF and commodity part of the fund. Bonds, silver and cash, about 43% of the capital, are excluded: they would lower the overall risk but carry risks of their own (duration, credit, FX) that are not modelled here. With a short history and constant weights, the numbers are an estimate of the past risk of today's allocation, not a forecast.

Possible next steps: a bond risk module (yield to maturity, duration, DV01) and a per-position risk contribution analysis that uses the covariance matrix.

## Limitations

- **Coverage.** About 43% of the fund is excluded, so the metrics describe only part of the fund.
- **Short history.** Data starts in 2023 and covers a single market regime, which is why 95% is the main confidence level: a 99% VaR would rest on very few observations.
- **Static weights.** Current weights are applied to the past, so the equity curve and drawdown do not reflect the real allocation over time.
- **Historical VaR.** It assumes the past distribution of returns is representative of the future and says nothing about losses beyond the observed sample.
- **Data quality.** Yahoo Finance tickers and listing currencies can be wrong or change, so prices should be checked against the fund's own records.
- **Log-return aggregation.** Portfolio returns are a weighted sum of daily log returns, a good approximation at daily frequency.

## How to run

```bash
pip install numpy pandas matplotlib seaborn yfinance jupyter
jupyter notebook starting-fund-risk-analysis.ipynb
```

The notebook reads a local CSV with the fund positions, which is not included in this repository. Run the cells in order; the download and currency-conversion cells need an internet connection.
