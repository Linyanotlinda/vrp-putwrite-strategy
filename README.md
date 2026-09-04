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
- Evaluating return, volatility, Sharpe ratio, drawdown, and exposure
- Comparing against randomly timed strategies with similar exposure
- Testing threshold sensitivity
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

## Performance

### Continuous PutWrite Exposure

- Annualized return: **8.09%**
- Annualized volatility: **13.42%**
- Sharpe ratio: **0.60**
- Maximum drawdown: **-28.9%**

### VRP-Timed PutWrite Strategy

- Annualized return: **5.31%**
- Annualized volatility: **6.99%**
- Sharpe ratio: **0.76**
- Maximum drawdown: **-7.4%**
- Market exposure: **11.5%**

The timed strategy produced lower absolute returns than continuous PutWrite exposure, but materially reduced volatility and maximum drawdown while improving risk-adjusted performance.

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

The main benefit of the timing rule was not higher absolute return, but improved risk-adjusted performance and substantially lower drawdowns.

The strategy was invested on only a small fraction of trading days, so lower exposure explains part of the reduction in risk. However, the random same-exposure comparison, threshold sensitivity analysis, and subperiod analysis provide additional evidence that the VRP signal itself contains useful timing information.

## Limitations

The analysis is based on a finite historical sample, and the results remain sample-dependent.

The PutWrite index is used as a benchmark return series rather than reconstructing individual historical SPX option trades.

When the strategy is not invested in the PutWrite index, the backtest assumes a zero return on idle capital. Incorporating a Treasury-bill or other cash return would be a natural extension.

The results should therefore be interpreted as empirical evidence about the historical relationship between the volatility risk premium and PutWrite performance, rather than evidence of guaranteed out-of-sample profitability.

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

Raw source files are not included in the repository.

## Notebook

See [`vrp_putwrite_strategy.ipynb`](vrp_putwrite_strategy.ipynb) for the full analysis.
