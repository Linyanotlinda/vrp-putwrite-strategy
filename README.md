# Volatility Risk Premium Timing of an SPX PutWrite Strategy

This project investigates whether the volatility risk premium can be used to improve the timing of a systematic SPX put-writing strategy.

The signal is constructed as the difference between VIX-implied volatility and 21-day realized S&P 500 volatility. A simple timing rule holds the Cboe S&P 500 PutWrite Index only when the standardized volatility risk premium exceeds a selected threshold.

## Overview

The workflow includes:

- Loading S&P 500, VIX, and Cboe PutWrite Index data
- Calculating daily S&P 500 log returns
- Estimating 21-day annualized realized volatility
- Constructing the volatility risk premium:

$$
VRP_t = VIX_t - RV_t
$$

- Standardizing VRP using a rolling 252-day z-score
- Comparing subsequent PutWrite returns across VRP regimes
- Building a timing rule using `VRP z-score > 1`
- Lagging the signal by one trading day to avoid look-ahead bias
- Evaluating return, volatility, Sharpe ratio, drawdown, and exposure
- Incorporating a 3-month Treasury return on idle capital
- Comparing against randomly timed strategies with similar exposure
- Testing threshold sensitivity
- Evaluating the frozen strategy on a final 30% temporal holdout sample
- Testing performance across subperiods

## Signal Construction

Daily S&P 500 log returns are calculated as:

$$
r_t = \ln\left(\frac{S_t}{S_{t-1}}\right)
$$

A 21-day rolling standard deviation is then annualized to estimate realized volatility:

$$
RV_t =
\sqrt{252}
\cdot
\sigma(r_{t-20}, \ldots, r_t)
$$

Realized volatility is expressed in percentage points so it is directly comparable with VIX.

The volatility risk premium is defined as:

$$
VRP_t = VIX_t - RV_t
$$

A positive VRP indicates that implied volatility exceeds recently realized volatility.

To account for changes in the level and dispersion of the volatility risk premium over time, the signal is standardized using a rolling 252-day z-score:

$$
z_t =
\frac{VRP_t - \mu_{t,252}}
{\sigma_{t,252}}
$$

## Regime Analysis

The standardized VRP is divided into three regimes:

- **Low:** `z < -1`
- **Normal:** `-1 <= z <= 1`
- **High:** `z > 1`

Subsequent PutWrite returns are compared across these regimes.

High-VRP observations were associated with stronger subsequent PutWrite returns over both one-day and 21-day horizons, motivating the use of VRP as a timing signal.

## Strategy

The strategy holds the PutWrite benchmark when:

```python
vrp_zscore > 1
```

and otherwise remains out of the PutWrite position.

The signal is lagged by one trading day so that today's end-of-day information determines tomorrow's exposure, avoiding look-ahead bias.

When the strategy is not allocated to the PutWrite index, a second implementation assumes that idle capital earns the 3-month U.S. Treasury constant-maturity rate rather than a zero return.

This cash-adjusted version is evaluated alongside the original zero-cash strategy so that the effect of the idle-capital assumption can be assessed separately from the VRP timing signal.

## Performance

| Strategy | Annualized Return | Annualized Volatility | Sharpe Ratio | Maximum Drawdown |
|---|---:|---:|---:|---:|
| Continuous PutWrite | 8.09% | 13.42% | 0.60 | -28.9% |
| VRP-Timed PutWrite | 5.31% | 6.99% | 0.76 | -7.4% |
| VRP-Timed PutWrite + Cash | 7.76% | 6.98% | 1.11 | -5.8% |

The timing rule allocates to the PutWrite index on approximately **11.5% of trading days**.

Without a return on idle capital, the timed strategy produces a lower absolute return than continuous PutWrite exposure, but materially reduces volatility and maximum drawdown.

Including a 3-month Treasury return on idle capital raises annualized return from approximately **5.31%** to **7.76%**, while volatility remains close to **7.0%**. Under the notebook's return-to-volatility Sharpe calculation, the ratio increases from approximately **0.76** to **1.11**.

This indicates that the zero-return assumption for idle capital was conservative.

## Robustness Tests

### Random Same-Exposure Benchmark

To distinguish the effect of the VRP signal from the effect of simply being invested less often, the strategy is compared with 1,000 randomly timed strategies with approximately the same average exposure.

The VRP-timed strategy's Sharpe ratio exceeded approximately **94.1%** of these randomly timed strategies.

This provides evidence that the timing signal contributed information beyond simply reducing market participation.

### Threshold Sensitivity

The VRP entry threshold was varied across:

- `0.50`
- `0.75`
- `1.00`
- `1.25`
- `1.50`

Results remained broadly supportive of the strategy for stricter thresholds.

| VRP Z-score Threshold | Annualized Return | Sharpe Ratio | Max Drawdown | Exposure |
|---:|---:|---:|---:|---:|
| 0.50 | 5.73% | 0.65 | -8.9% | 32.3% |
| 0.75 | 4.74% | 0.59 | -16.6% | 20.4% |
| 1.00 | 5.31% | 0.76 | -7.4% | 11.5% |
| 1.25 | 5.70% | 0.90 | -6.0% | 6.5% |
| 1.50 | 4.96% | 0.85 | -7.6% | 3.6% |

The main strategy retains the simple `z > 1` rule rather than selecting the highest-Sharpe threshold after observing the results.

### Out-of-Sample Holdout Test

The final **30%** of the historical backtest is treated as a temporal holdout sample.

Before evaluating this period, the strategy specification is kept unchanged:

- 21-day realized volatility
- 252-day rolling VRP z-score
- Entry threshold of `z > 1`
- One-day lag between signal formation and exposure

No parameters are adjusted based on holdout-period performance.

In the development sample, the VRP-timed strategy improved the Sharpe ratio from approximately **0.41** for continuous PutWrite exposure to **0.77**, while substantially reducing maximum drawdown.

In the holdout sample, the timing strategy continued to reduce volatility and drawdown, with maximum drawdown falling from approximately **-15.1%** to approximately **-2.7%**. However, its Sharpe ratio of approximately **0.74** was below the continuous PutWrite benchmark's approximately **1.30**.

The holdout test therefore suggests that the more persistent benefit of the timing rule may be exposure and drawdown control rather than consistently superior risk-adjusted returns.

### Subperiod Analysis

The sample was divided into two approximately equal subperiods.

In the first half:

- PUT Sharpe ratio: **0.45**
- VRP-timed Sharpe ratio: **0.62**
- PUT max drawdown: **-28.9%**
- VRP-timed max drawdown: **-7.4%**

In the second half:

- PUT Sharpe ratio: **0.85**
- VRP-timed Sharpe ratio: **0.97**
- PUT max drawdown: **-16.0%**
- VRP-timed max drawdown: **-3.5%**

The strategy therefore improved risk-adjusted performance and reduced drawdowns in both subperiods rather than relying on a single market episode.

## Interpretation

The results suggest that unusually high volatility risk premium regimes may offer more attractive conditions for systematic option-selling exposure.

Across the full sample, the main benefit of VRP timing is a substantial reduction in volatility and drawdown. The random same-exposure benchmark suggests that this result is not explained entirely by being invested less often.

Adding a Treasury return on idle capital materially improves the economics of the strategy by allowing unallocated capital to earn a positive return rather than assuming zero return.

However, the temporal holdout test shows that the strategy's Sharpe-ratio advantage is not stable across all periods. Although drawdown and volatility reduction persisted in the holdout sample, continuous PutWrite exposure achieved the higher Sharpe ratio.

The evidence therefore supports VRP more strongly as a regime-selection and risk-control signal than as a consistently superior source of risk-adjusted returns.

## Limitations

The analysis is based on a finite historical sample, and the results remain sample-dependent.

The PutWrite index is used as a benchmark return series rather than reconstructing individual historical SPX option trades.

The cash-adjusted implementation uses the 3-month U.S. Treasury constant-maturity rate as a proxy for the return available on idle capital. It does not model transaction costs, financing frictions, or implementation costs.

The holdout test provides one historical out-of-sample evaluation, but it does not establish that the strategy will perform similarly in future market regimes.

The results should therefore be interpreted as empirical evidence about the historical relationship between the volatility risk premium and PutWrite performance rather than evidence of guaranteed future profitability.

## Tools

- Python
- pandas
- NumPy
- Matplotlib

## Data

The analysis uses:

- S&P 500 daily prices
- VIX daily values
- Cboe S&P 500 PutWrite Index historical levels
- 3-month U.S. Treasury constant-maturity rates (`DGS3MO`) for the idle-cash return

The source data files used in the notebook are included in this repository for reproducibility.

## Notebook

See [`vrp_putwrite_strategy.ipynb`](vrp_putwrite_strategy.ipynb) for the full analysis.
