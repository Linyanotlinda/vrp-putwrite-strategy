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
