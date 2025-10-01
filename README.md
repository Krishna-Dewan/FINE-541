# Portfolio Optimization & Performance Analysis

This R script performs **portfolio construction, optimization, and performance evaluation** for a multi-asset portfolio with a cash component.  
It leverages **historical price data, CAPM modeling, custom forecasts, and risk/return optimization** to recommend optimal portfolio weights.

The workflow includes:
- Data loading and preparation (12-month historical prices)
- CAPM-based expected return estimation
- Risk decomposition (systematic vs. idiosyncratic)
- Portfolio optimization with custom constraints
- Trade recommendations
- Performance analysis against an **XCV ETF benchmark**
- Visualization of efficient frontier, cumulative returns, and risk decomposition

---

## Dependencies
The script requires the following R packages:

```r
library(dplyr)
library(readr)
library(purrr)
library(zoo)
library(CVXR)      # for convex optimization
library(ggplot2)   # for visualization
library(tidyr)
```
## Parameters

1. Data & File Inputs

```r
stock_files   # CSV files for each asset's historical prices
stock_tickers # Asset tickers (XCV, LAS, AA, GLD, LNF, HLF, NPI)

```
- Data is pulled for the last 252 trading days (~12 months).

- XCV is treated as the benchmark.

2. Portfolio Parameters

```r
portfolio_value <- 290898.22  # Total portfolio value (USD)
current_w <- c(XCV = 0.376, LAS = 0.1873, AA = 0.0987, GLD = 0.0636, LNF = 0, HLF = 0, NPI = 0, Cash = 0.2744)
```

    - Defines the current portfolio weights.
    
    - Includes a Cash component.

3. Expected Price Targets

```r
expected_prices <- c(
  LAS = 254,
  AA  = 33.91,
  GLD = 360,
  LNF = 35.57,
  HLF = 21.39,
  NPI = 27.62
)
```
    User-defined expected prices for individual assets.

    XCV’s return is derived via CAPM and adjusted for fees.

4. Benchmark & Fees

```r
Rf <- 0.04                   # Annual risk-free rate
xcv_management_fee <- 0.0050 # 0.50%
xcv_mer <- 0.0055            # 0.55%
xcv_total_fees <- 0.0105     # Total: 1.05%
```
    
 XCV ETF expected return is reduced by its management fees.

5. Optimization Constraints

```r
min_weights <- c(XCV = 0.15, LAS = 0.10, AA = 0.00, GLD = 0.08, LNF = 0.05, HLF = 0.10, NPI = 0.05, Cash = 0.06)
max_weights <- c(XCV = 0.80, LAS = 0.30, AA = 0.30, GLD = 0.12, LNF = 0.30, HLF = 0.30, NPI = 0.30, Cash = 0.10)

w_cash_target <- 0.08
```
- Sets allocation floors and caps for each asset and cash.

- Enforces a target cash allocation.

6. Cost & Risk Parameters
   
```
transaction_cost_dollars <- 50   # Per-asset trade cost
lambda_cash <- 50                # Penalty for deviation from target cash
risk_aversion <- 0.05            # Penalizes portfolio variance
```
7. Regularization
   
```
covariance_epsilon <- 1e-6
```
-Ensures the covariance matrix remains positive-definite.

What the Script Does
Step-by-Step Process

- Load & Clean Data

        Reads price data for each asset.

        Keeps only the last 12 months (≈252 trading days).

        Fills missing prices with the last known value (na.locf).

- Compute Returns & Expected Returns

        Calculates daily log returns.

        Derives annualized expected returns using:

            CAPM (for XCV)

            User-provided price targets (for other stocks)

- CAPM Analysis

        Computes betas, alphas, and volatility decomposition for each asset.

- Covariance Matrix Construction

        Builds annualized covariance matrix for portfolio risk modeling.

        Adds regularization if needed.

- Portfolio Optimization

        Uses mean-variance optimization via CVXR.

        Maximizes:

        Expected Return 
        – (risk_aversion × Variance) 
        – Transaction Costs 
        – Cash Penalty

        Respects min/max weight constraints and total weight = 1.

- Trade Recommendations

        Compares current vs. optimal weights.

        Calculates trade amounts and total transaction cost.

- Performance Evaluation

        Reports portfolio return, volatility, Sharpe, Sortino, alpha, beta, tracking error, and information ratio.

        Compares optimized portfolio vs. XCV benchmark.

- Visualization

        Efficient frontier with optimal portfolio & benchmark

        Cumulative return comparison

        Risk decomposition (systematic vs. idiosyncratic)

## Key Outputs

   - Optimal portfolio weights

   - Trade orders & transaction cost

   - Performance metrics: return, volatility, Sharpe, Sortino, alpha, beta

   - Plots:

        Efficient frontier

        Portfolio vs. benchmark cumulative returns

        Risk decomposition bar chart

## Notes

    Adjust price targets, constraints, or risk_aversion to explore different strategies.

    Ensure min_weights sum ≤ 1 to avoid infeasible optimization.

    Cash penalty (lambda_cash) encourages allocation discipline around w_cash_target.

    Regularization (covariance_epsilon) improves numerical stability for covariance matrices.

## Quick Start

    Place all input CSVs in the specified folder (New dataset/).

    Modify parameters (price targets, constraints, risk-free rate, etc.) as needed.

    Run the R script.

    Review the console output for weights, trades, and performance.

    Inspect the generated plots for risk/return insights.
