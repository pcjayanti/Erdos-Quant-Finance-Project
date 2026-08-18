# Merton Jump-Diffusion: Theory, Hedging, and Market Calibration

Final project for The Erdős Institute Fall 2025 Quant Finance Boot Camp.

Merton's Jump-Diffusion (MJD) model extends Black-Scholes by adding a compound
Poisson jump process to the price dynamics, allowing it to generate the
volatility smile that Black-Scholes cannot. This repository studies the model
theoretically, tests it as a hedging framework, and calibrates it to live
SPY option data.

## Contents

**`European calls with MJD.ipynb`** — Derivation and numerical study of the MJD
model for European call pricing, plus a Delta-hedging experiment across
rebalancing frequencies. Hedging sharply reduces P&L variance, but the gains
saturate as rebalancing frequency increases.

**`MJD and real market data.ipynb`** — Calibration to market data and
out-of-sample testing. Parameters are fitted to the SPY call chain, then used
to price puts on the same expiry via put-call parity. The puts are never seen
during calibration, making this a genuine out-of-sample test. Results are scored
against a flat-volatility Black-Scholes benchmark.

**`data/`** — Saved SPY option chain snapshot (2026-09-18 expiry) and market
state. The notebook runs against this snapshot offline, so the results below are
reproducible rather than dependent on live data.

**`Project presentation`** - The presentation is a brief summary of the results from the original project (submitted in Nov 2025), and there is also an accompanying video summary at https://youtu.be/z8wizw0nYQI . The presentation and video do not reflect any modifications made after Nov 2025.


## Results

Calibrated on 234 liquid calls; tested on 245 liquid puts. Errors in
implied-volatility points:

| | IV RMSE | IV bias |
|---|---|---|
| MJD | 6.23 | −2.66 |
| Black-Scholes (flat vol) | 23.23 | −16.20 |

By moneyness (K/S):

| Bucket | n | MJD RMSE | BSM RMSE |
|---|---|---|---|
| deep OTM (<0.85) | 92 | 10.01 | 36.37 |
| OTM (0.85–0.95) | 77 | 1.43 | 11.49 |
| near ATM (0.95–1.05) | 73 | 1.11 | 2.22 |

MJD fits the liquid region to roughly 1.3 vol points, against 8.4 for the
benchmark. The advantage widens with distance from spot — the wings are
precisely where a flat-volatility model has no mechanism to respond.

**Limitation.** MJD systematically underprices far-tail volatility (−7.69 vol
point bias below 0.85 moneyness). This is a model limitation rather than a data
artifact: median bid-ask width there corresponds to 0.15 vol points against a
median model error of 4.81, and MJD's implied volatility falls outside the
quoted range on every deep-OTM strike. A single lognormal jump size
distribution cannot produce skew as steep as the market prices.

**On metrics.** Both models exceed R² = 0.98 on put prices, which is
uninformative here: a model using discounted intrinsic value alone, with no
volatility input, scores R² = 0.82 on the same data. Option prices are
dominated by predictable variation with moneyness, so R² largely measures
whether a model knows the gross shape of the price curve. Implied volatility
isolates the uncertainty premium — the part a model actually predicts.

## Reproducing

Requires `numpy`, `pandas`, `scipy`, `matplotlib`, `scikit-learn`, `yfinance`.
Run the load cell rather than the calibration cell to use the saved snapshot;
live calibration requires market hours.
