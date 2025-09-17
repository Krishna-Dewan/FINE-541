# FINE-541
Portfolio Optimizer Tool

Portfolio Optimization Toolkit

A robust R-based tool for constructing and analyzing optimal investment portfolios. This toolkit moves beyond traditional Mean-Variance Optimization (MVO) by maximizing the Sharpe Ratio (finding the tangency portfolio) and incorporating real-world constraints like transaction costs, position limits, and cash targets.
Table of Contents

   ## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Theoretical Foundation](#theoretical-foundation)
- [Installation & Setup](#installation--setup)
- [Usage Guide](#usage-guide)
  - [1. Prepare Your Data](#1-prepare-your-data)
  - [2. Configure Input Parameters](#2-configure-input-parameters)
  - [3. Run the Optimization](#3-run-the-optimization)
  - [4. Evaluate a New Stock](#4-evaluate-a-new-stock)
- [Output Interpretation](#output-interpretation)
- [Customization](#customization)
- [Dependencies](#dependencies)
- [Disclaimer](#disclaimer)

## Overview

This project implements a constrained portfolio optimization model. It solves for the portfolio weights that maximize the risk-adjusted return (Sharpe Ratio) based on our forecasts, while respecting constraints such as:

-Minimum and maximum asset allocation limits.
-A target cash allocation (with a soft penalty for deviation).
-Transaction costs.
-It also includes a module to pre-evaluate a new stock against your existing benchmark.

## Key Features

- **Sharpe Ratio Maximization:** Finds the tangency portfolio
- **Real-World Constraints:** Enforces practical trading limits
- **Transaction Cost Awareness:** Optimizes net of trading costs
- **Multi-Currency Support:** Handles USD and CAD-denominated assets

    Comprehensive Performance Reporting: Provides expected return, volatility, Sharpe Ratio, Jensen's Alpha, and M^2 measure for the optimized portfolio.
 
## Theoretical Foundation

Traditional Mean-Variance Optimization (MVO) minimizes risk for a given target return, which is highly sensitive to error-prone expected return inputs. This project instead maximizes the Sharpe Ratio:

$$SR = \frac{\mu_p - r_f}{\sigma_p}$$

This approach finds the portfolio with the highest reward per unit of risk (the tangency portfolio). An investor can then adjust their overall risk by allocating between this optimal risky portfolio and the risk-free asset. This method is generally more robust and conceptually sound than standard MVO.

The optimization problem is solved using CVXR, an R package for convex optimization, which handles the constraints and objective function efficiently.


## Installation & Setup


Install Required Packages: Run the following commands in your R console:

    `r

`install.packages(c("quantmod", "CVXR", "dplyr", "readr", "purrr", "zoo"))`


## Usage Guide

### 1. Prepare Your Data

The script expects CSV files for asset prices and FX rates. The data reading functions are designed for a specific format (e.g., from Yahoo Finance or Investing.com).

  CAD Stocks: Place CSV files for Canadian-listed stocks (e.g., "XCV Historical Data.csv").

  USD Stocks: Place CSV files for US-listed stocks.

  FX Rate: Provide a CSV file for the USD/CAD exchange rate (e.g., "USD_CAD Historical Data.csv"). The script will use this to convert USD asset prices to CAD.

### 2. Configure Input Parameters

The core of the setup is in the Inputs section. You must modify these variables to match your scenario:

`r

```
# ---- Inputs (replace with your data) ----
price_data <- price_data  # This is loaded from your CSV files

# Your price forecasts for each asset
expected_prices <- c(
  XCV = 50,   # expected future price for XCV
 LAS = 240,   # expected future price for LAS
  AA  = 48.16,   # expected future price for AA
  GLD = 482   # expected future price for GLD
)

# Your chosen benchmark (e.g., 100% XCV)
benchmark_w <- c(XCV = 1.0, LAS = 0, AA = 0, GLD = 0)

# YOUR CURRENT PORTFOLIO WEIGHTS (MUST SUM TO 1.0)
current_w <- c(XCV = 0.376, LAS = 0.1873, AA = 0.0987, GLD = 0.0636, Cash = 0.2744)

# Your total portfolio value in CAD
portfolio_value <- 288392.68

# Your estimated transaction cost ($)
transaction_cost_dollars <- 50

# The risk-free rate (e.g., annualized T-bill rate)
Rf <- 0.04

# Target cash weight and penalty strength for deviation
w_cash_target <- 0.08
lambda_cash <- 50

# Your personal risk aversion coefficient
risk_aversion <- 0.2

# Minimum and maximum allowed weights for each asset
min_weights <- c(XCV = 0.05, LAS = 0.10, AA = 0.0, GLD = 0.10, Cash = 0.08)
max_weights <- c(XCV = 0.80, LAS = 0.20, AA = 0.20, GLD = 0.15, Cash = 0.10)

```

### 3. Run the Optimization

Execute the entire script. The solver will calculate the optimal weights that maximize your utility function, which balances expected return against risk, transaction costs, and cash deviation.

### 4. Evaluate a New Stock

The Evaluate a New Stock section allows you to analyze a potential new addition without re-running the entire optimization.

Load the new stock's historical data CSV.

Crucial: Set your price forecast for the new stock: `expected_price_new <- 26`

Run the section. It will calculate and print the new stock's expected return, its beta and alpha relative to your benchmark (XCV), and its Information Ratio.

## Output Interpretation

The script provides three key outputs:

  `Optimal Portfolio Weights`: The recommended asset allocation as percentages.

  `Optimal Dollar Allocation`: The recommended allocation in dollar terms.

  `Recommended Trade Orders`: The dollar amount to buy or sell for each asset to move from your current_w to the optimal_w.

  Performance Metrics:

  `Expected Return (Rp)`: The forecasted return of the optimized portfolio.

  `Volatility (σp)`: The forecasted risk (standard deviation) of the portfolio.

  `Sharpe Ratio`: `Rp` per unit of risk (higher is better).

  `Alpha (Jensen)`: The excess return expected from the portfolio compared to the benchmark, adjusted for risk (beta).

  `M^2`: A measure of risk-adjusted performance compared to the benchmark.

## Customization

  - Constraints: Modify the `min_weights`, `max_weights`, and constraints list to add group constraints (e.g., sector limits) or change existing ones.

  - Objective Function: Adjust the `risk_aversion` parameter. A higher value will result in a more conservative portfolio.

  - Cash Penalty: Change `lambda_cash` to make the optimizer adhere more strictly (`lambda_cash` >> 50) or more loosely (`lambda_cash` ~ 1) to the cash target.

## Dependencies

 CVXR: For solving the convex optimization problem.

 `dplyr`, `purrr`, `readr`: For data manipulation and reading CSVs.

  `zoo`: For handling missing data (na.locf).

  `quantmod`: (Currently loaded but not essential for the core script; can be removed if unused).

