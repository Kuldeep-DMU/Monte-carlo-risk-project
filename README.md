# Monte Carlo Option Pricing and Portfolio Value-at-Risk

A Python project in two parts: pricing a European option by Monte Carlo simulation, and measuring and backtesting the market risk of a stock portfolio with Value-at-Risk (VaR) and Expected Shortfall (ES).

Built in Google Colab with NumPy, pandas, SciPy and Matplotlib.

## Results

### 1. Option pricing (S0=100, K=100, T=1, r=5%, sigma=20%)

| Method | Price | Standard error |
|---|---|---|
| Black-Scholes (closed form) | 10.4506 | n/a |
| Monte Carlo, 1,000,000 paths | 10.4524 | 0.0147 |
| Monte Carlo + antithetic variates, 1,000,000 paths | 10.4570 | 0.0104 |

Monte Carlo is within 0.02% of Black-Scholes. Antithetic variates cut the standard error by 29%. Both 95% confidence intervals contain the Black-Scholes price.

![Convergence](results/option_convergence.png)

### 2. Portfolio risk (equal-weight AAPL, MSFT, JPM, XOM, JNJ; 4,232 daily returns, Jan 2010 to Oct 2026)

| 1-day | Method | VaR | ES |
|---|---|---|---|
| 95% | Historical | 1.63% | 2.61% |
| 95% | Parametric (normal) | 1.76% | 2.22% |
| 95% | Monte Carlo | 1.76% | 2.22% |
| 99% | Historical | 3.19% | 4.39% |
| 99% | Parametric (normal) | 2.52% | 2.90% |
| 99% | Monte Carlo | 2.52% | 2.88% |

At 99%, the normal-based methods understate VaR by about 21% and ES by about 34% compared with the historical method.

### 3. Backtest (99% VaR, rolling 250-day window, 3,982 days)

| Model | Exceptions | Expected | Kupiec p-value | Result at 5% |
|---|---|---|---|---|
| Historical | 53 | 39.8 | 0.046 | Reject (borderline) |
| Parametric (normal) | 89 | 39.8 | < 0.001 | Reject |

The normal model fails by a wide margin because real returns have fatter tails than a normal distribution. Exceptions cluster in stress periods such as 2011, 2018, March 2020 and April 2025, which suggests volatility-adaptive models (EWMA, GARCH) as a next step.

![Backtest](results/var_backtest.png)

## Method

- **Option pricing:** simulate terminal prices under geometric Brownian motion, S_T = S0 exp((r - sigma^2/2)T + sigma sqrt(T) Z), discount the average payoff, and compare with the Black-Scholes formula. Standard error falls as 1/sqrt(N). Antithetic variates pair each draw Z with -Z to reduce variance.
- **Historical VaR/ES:** empirical loss quantile and the average loss beyond it.
- **Parametric VaR/ES:** assumes normally distributed portfolio returns.
- **Monte Carlo VaR/ES:** simulates 200,000 correlated asset-return vectors using the Cholesky factor of the covariance matrix.
- **Backtest:** the VaR for each day uses only the previous 250 returns. The Kupiec proportion-of-failures test checks whether the number of exceptions matches the 1% expected rate.

## Data

Daily closing prices from the Kaggle file `SnP_daily_update.csv` (S&P 500 daily update dataset by yash16jr), covering 2010 to 2026. The file is 162 MB, so it is not included here. Download it from the dataset page on Kaggle, where the licence terms are also listed. Prices appear to be split- and dividend-adjusted.

## How to run

Option A, Google Colab: open `monte_carlo_risk_colab.ipynb`, upload `monte_carlo_risk.py` and the Kaggle CSV when asked, and run the cells in order.

Option B, locally:
```
pip install -r requirements.txt
python monte_carlo_risk.py --source csv --csv SnP_daily_update.csv --tickers AAPL MSFT JPM XOM JNJ --start 2010-01-01 --end 2026-11-01
```
Outputs are written to `results/`.

## Limitations

- Only five large, liquid US stocks with equal weights.
- The dataset contains current S&P 500 members only, so delisted companies are missing (survivorship bias).
- Constant-volatility assumptions in the parametric and Monte Carlo VaR methods.
- The option study uses one set of illustrative parameters and a single random seed.

## Possible extensions

EWMA or GARCH volatility, Student-t or filtered historical simulation, stressed VaR, and a C++ version of the pricing loop.

## Files

- `monte_carlo_risk.py`: all code
- `monte_carlo_risk_colab.ipynb`: the notebook run, with outputs
- `results/`: tables (CSV) and charts (PNG) from the run
