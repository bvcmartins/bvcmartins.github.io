---
layout: post
title: "Quantum vs. Classical Portfolio Optimization: A Benchmark Across 680 Securities"
date: 2026-06-30 10:00:00 -0000
categories: quantum finance portfolio-optimization benchmarking
tags: [Quantum, D-Wave, Portfolio Optimization, Finance, QUBO, HRP, Sharpe, Benchmark, SLSQP]
---

*Can quantum computing improve how we allocate capital? We ran the numbers.*

---

## Overview

Portfolio optimization is a computationally hard problem. As the number of assets grows, the search space for optimal allocations explodes. This makes it a natural target for quantum approaches—at least in theory. In this study, we benchmarked classical and quantum solvers across three distinct optimization formulations on a universe of **680 US stocks and ETFs** spanning 2011–2024.

The setup: 10 randomly selected 30-day walk-forward test periods. Each solver optimizes weights using the preceding 30 days of returns, then we evaluate performance on the following 30 days. No forecasting, no transaction costs—a clean apples-to-apples comparison of the optimization methods themselves.

The three formulations tested:

| Formulation | Objective | Quantum Solver |
|---|---|---|
| **QUBO** | Maximize in-sample Sharpe (QUBO binary encoding) | D-Wave CQM |
| **Sharpe** | Maximize risk-adjusted returns (continuous weights) | D-Wave Nonlinear |
| **HRP** | Hierarchical risk parity (tree-based allocation) | — |
| **Pareto** | Efficient frontier (risk-return tradeoff) | D-Wave CQM |

Baseline in all experiments: **Equal Weights (1/N)** across all 680 securities.

---

## Part I — QUBO Formulation

The earliest formulation in this project encodes portfolio weights as binary variables and submits the problem as a Constrained Quadratic Model (CQM) to D-Wave Advantage. The quantum solver competed against Scipy SLSQP and a Genetic Algorithm on the same QUBO objective.

### Results

| Solver | n | Avg Return | Std | vs. Benchmark | Sharpe | Vol | Max DD | Avg Runtime |
|---|---|---|---|---|---|---|---|---|
| Scipy SLSQP | 10 | **+6.55%** | ±4.05% | +5.80pp | **+3.99** | 14.76% | −3.71% | 3,400s |
| Genetic Algorithm | 10 | +2.70% | ±3.60% | +1.95pp | +1.88 | 13.48% | −4.32% | 4,340s |
| Equal Weights | 10 | +0.77% | ±3.53% | baseline | +0.58 | 13.68% | −5.04% | <1s |
| **D-Wave CQM** | **8** | **−1.87%** | ±4.40% | **−2.62pp** | **−0.99** | 11.72% | −5.19% | **8,095s** |

*(D-Wave run on 8 of 10 periods due to compute quota constraints.)*

**SLSQP dominates**, delivering 6.55% average 30-day return with the strongest Sharpe (+3.99) and lowest drawdown (−3.71%). The Genetic Algorithm earns roughly half that return at similar volatility but longer runtime.

**D-Wave CQM underperforms significantly.** It is the only solver with a negative average return (−1.87%), finishing below even the Equal Weights benchmark by 2.6 percentage points. Its Sharpe of −0.99 indicates the solver is actively destroying risk-adjusted value. Runtime is more than double SLSQP's, averaging over 2 hours per period.

The result suggests that, at this problem scale (680 assets), the QUBO binary encoding introduces too coarse a discretization for D-Wave to find competitive portfolios. The constraint overhead of the CQM formulation may also be overwhelming the annealer's ability to converge.

---

## Part II — Sharpe Optimization

The Sharpe formulation moves away from binary encoding. Weights are continuous variables and the objective is direct Sharpe ratio maximization. D-Wave's Nonlinear Solver (NL) was tested here instead of the CQM, applying quantum resources to a continuous optimization landscape.

### Results

| Solver | n | Avg Return | Std | vs. Benchmark | Sharpe | Vol | Max DD | Avg Runtime |
|---|---|---|---|---|---|---|---|---|
| Scipy SLSQP | 10 | **+5.60%** | ±3.40% | +4.85pp | **+4.27** | 12.28% | **−3.17%** | 2,411s |
| Genetic Algorithm | 10 | +2.56% | ±4.30% | +1.81pp | +1.94 | 13.26% | −4.34% | 4,358s |
| Equal Weights | 10 | +0.77% | ±3.53% | baseline | +0.58 | 13.68% | −5.04% | <1s |
| **D-Wave Nonlinear** | **3** | **−2.09%** | ±5.60% | **−2.84pp** | **−1.22** | 14.29% | **−6.43%** | **31,342s** |

*(D-Wave tested on only 3 periods.)*

The pattern from the QUBO formulation repeats. SLSQP again leads with +5.60% average return, the best Sharpe (+4.27), and the tightest drawdown (−3.17%). The continuous-weight formulation appears to be its natural habitat.

**D-Wave Nonlinear is the worst performer on every metric.** The negative return (−2.09%) and Sharpe (−1.22) are comparable to the CQM result, but the situation is worse in two ways:
1. **Volatility is highest** at 14.29%—it is taking on more risk than any other method while generating the worst returns.
2. **Runtime is catastrophic**: 31,342 seconds average (~8.7 hours per period), over 13× slower than SLSQP and over 7× slower than the Genetic Algorithm.

The 3-period sample limits statistical confidence, but the direction is unambiguous across both D-Wave formulations.

---

## Part III — Hierarchical Risk Parity (HRP)

HRP is a fundamentally different approach: rather than maximizing a return objective, it builds portfolios by clustering assets via hierarchical correlation structure and allocating risk inversely proportional to cluster variance. We tested the Pearson correlation + Ward linkage variant.

### Results

| Solver | n | Avg Return | Std | vs. Benchmark | Sharpe | Vol | Max DD | Avg Runtime |
|---|---|---|---|---|---|---|---|---|
| HRP (Pearson–Ward) | 10 | +2.67% | ±3.16% | −0.25pp | **+2.43** | **9.87%** | **−2.61%** | 18s |
| Equal Weights | 10 | +2.88% | ±3.44% | baseline | +1.85 | 14.04% | −3.64% | <1s |

*Note: the HRP benchmark is a different set of 10 periods from QUBO/Sharpe, so cross-set comparisons should be made cautiously.*

HRP tells a different story from the return-maximizing solvers. Its average return (+2.67%) is slightly below the Equal Weights baseline (+2.88%) in these periods, but the **risk profile is dramatically better**:

- **Volatility: 9.87%** — the lowest of any solver across all experiments, nearly 40% below Equal Weights in this set.
- **Max drawdown: −2.61%** — the best across all methods.
- **Sharpe: +2.43** — second only to SLSQP in the Sharpe formulation.

HRP achieves this in **18 seconds**—orders of magnitude faster than any return-optimizing solver. For investors who prioritize capital preservation and consistent risk-adjusted performance over raw return maximization, HRP's profile is compelling.

---

## Part IV — Pareto Efficient Frontier

The Pareto experiment explores the full risk-return tradeoff. Each solver is run at 11 risk-aversion levels (λ = 0.0 to 1.0), tracing an efficient frontier from pure return-seeking (λ=0) to pure risk-minimization (λ=1). Classical solvers were tested across all 10 periods; D-Wave CQM was limited to a single period.

### Efficient Frontier: Average Return vs. Risk per λ

#### Classical Solvers (averaged across 10 periods)

| λ | SLSQP Return | SLSQP Risk | Riskfolio Return | Riskfolio Risk | GA Return | GA Risk |
|---|---|---|---|---|---|---|
| 0.0 (max return) | 0.867% | 2.137% | 0.866% | 2.137% | 0.122% | 0.761% |
| 0.2 | 0.866% | 2.091% | 0.866% | 2.088% | 0.125% | 0.753% |
| 0.5 | 0.866% | 2.084% | 0.865% | 2.071% | 0.116% | 0.756% |
| 0.8 | 0.865% | 2.066% | 0.865% | 2.048% | 0.120% | 0.752% |
| 1.0 (min risk) | 0.864% | 2.042% | 0.862% | 2.003% | 0.120% | 0.752% |

*(Return and risk are 30-day figures, averaged across 10 periods.)*

#### D-Wave CQM (1 period, λ = 0 to 1)

| λ | D-Wave Return | D-Wave Risk | Sharpe | Runtime |
|---|---|---|---|---|
| 0.0 | **0.371%** | **1.172%** | 0.316 | 286s |
| 0.1 | 0.010% | 0.126% | 0.077 | 294s |
| 0.2 | 0.002% | 0.243% | 0.009 | 17,526s |
| 0.3 | 0.012% | 0.235% | 0.049 | 1,154s |
| 0.4 | 0.013% | 0.190% | 0.071 | 4,814s |
| 0.5 | 0.010% | 0.235% | 0.042 | **36,199s** |
| 0.6 | 0.010% | 0.226% | 0.046 | 565s |
| 0.7 | −0.007% | 0.236% | −0.029 | 1,194s |
| 0.8 | 0.005% | 0.236% | 0.021 | 310s |
| 0.9 | 0.018% | 0.194% | 0.093 | 4,105s |
| 1.0 | 0.004% | 0.139% | 0.027 | 3,942s |

### Analysis

**SLSQP and Riskfolio are essentially identical.** Both trace the same efficient frontier with return near 0.86% across all λ values and risk declining from 2.14% to 2.04% as risk aversion increases. Riskfolio achieves this in ~2 seconds; SLSQP takes ~800 seconds for the same result—Riskfolio's MPT-based solver is ~400× faster with no penalty in solution quality.

**Genetic Algorithm explores a different region of the frontier.** It consistently finds lower-return, lower-risk portfolios (~0.12% return, ~0.75% risk) regardless of λ. The GA's multi-objective landscape here appears to collapse onto a single cluster—it is not effectively navigating the λ parameter to produce a spread-out frontier.

**D-Wave CQM produces a severely degraded frontier.** With one exception (λ=0.0, where it achieves 0.37% return), all remaining points cluster near zero return (0.001%–0.018%). This is roughly 50–70× lower return than SLSQP/Riskfolio at comparable or higher risk levels. The solver is failing to find competitive solutions for the intermediate and high risk-aversion regimes.

**D-Wave runtime on the Pareto frontier is deeply unpredictable.** The λ=0.5 case consumed 36,199 seconds (~10 hours) for a single portfolio that returned 0.010% with 0.235% risk. The λ=0.2 case took 17,526 seconds. Other λ values completed in under 600 seconds. This variance of ~60× in solve time for structurally similar problems is a significant operational concern.

---

## Summary: Head-to-Head Comparison

### Single-Objective (QUBO / Sharpe formulations)

| Solver | Formulation | Avg Return | Sharpe | Vol | Max DD | Runtime |
|---|---|---|---|---|---|---|
| SLSQP | QUBO | **+6.55%** | **+3.99** | 14.76% | −3.71% | ~1hr |
| SLSQP | Sharpe | +5.60% | +4.27 | **12.28%** | **−3.17%** | ~40min |
| GA | QUBO | +2.70% | +1.88 | 13.48% | −4.32% | ~72min |
| GA | Sharpe | +2.56% | +1.94 | 13.26% | −4.34% | ~73min |
| HRP (Pearson–Ward) | HRP | +2.67% | +2.43 | **9.87%** | **−2.61%** | **18s** |
| Equal Weights | — | ~+0.8–2.9% | 0.58–1.85 | 13.7–14.0% | −3.6–5.0% | <1s |
| D-Wave CQM | QUBO | −1.87% | −0.99 | 11.72% | −5.19% | ~2.25hr |
| D-Wave Nonlinear | Sharpe | −2.09% | −1.22 | 14.29% | −6.43% | **~8.7hr** |

### Multi-Objective (Pareto / Efficient Frontier)

| Solver | Frontier Coverage | Best Return | Best Sharpe | Avg Runtime/λ |
|---|---|---|---|---|
| SLSQP | Full (λ 0→1) | 0.866% | 0.461 | ~800s |
| Riskfolio | Full (λ 0→1) | 0.866% | 0.465 | **~2s** |
| GA | Partial (collapsed) | 0.125% | 0.183 | ~132s |
| **D-Wave CQM** | Mostly collapsed | **0.371%** (λ=0 only) | 0.316 | **~6,300s avg** |

---

## D-Wave: A Closer Look

Across all four experimental setups, D-Wave hardware failed to match classical solvers on any performance metric. The pattern is consistent:

**On solution quality:** Both the CQM and Nonlinear solver produce negative or near-zero returns where SLSQP achieves 5–7%. In the Pareto experiment, D-Wave's frontier is almost entirely collapsed to the lower-left corner of the risk-return space, failing to leverage higher risk tolerance when λ is low.

**On runtime:** D-Wave's wall-clock time is not competitive. It ranges from ~2× to ~50× slower than SLSQP depending on the experiment, with extreme and unpredictable variance across runs (36,000s vs. 285s for structurally similar Pareto problems).

**On consistency:** The CQM Pareto run at λ=0.7 produced the only negative return (−0.007%), and Sharpe ratios across λ values range from −0.029 to +0.316 with no coherent trend—the solver is not reliably following the risk-aversion signal.

### Potential explanations

These results are not a condemnation of quantum computing broadly, but they do highlight specific current limitations:

1. **Problem scale.** With 680 continuous-valued assets, the problem is large. D-Wave's strength has historically been small, structured binary problems. The QUBO encoding discretizes weights but cannot represent the full continuous solution space well at this scale.

2. **Constraint overhead.** Portfolio constraints (weights sum to 1, long-only) add significant overhead to the CQM formulation. At 680 variables, the constraint penalty landscape may dominate the objective.

3. **Qubit connectivity.** D-Wave's quantum annealing topology requires chains of physical qubits to represent single logical qubits. Long chains degrade solution quality.

4. **Hybrid mode.** Both the CQM and Nonlinear solvers used here are D-Wave's hybrid quantum-classical solvers, not pure annealing. The long runtimes suggest the classical component is doing heavy lifting, which limits any quantum speedup argument.

---

## Conclusions

This benchmark draws a clear picture for the 680-asset, 30-day horizon scenario:

1. **Scipy SLSQP is the strongest single-objective solver.** It delivers the best returns (5.6–6.6%), best Sharpe (4.0–4.3), and lowest drawdown (−3.2%) in both QUBO and Sharpe formulations. It is not the fastest, but its quality-to-cost ratio is unmatched.

2. **Riskfolio is the efficiency champion for multi-objective work.** On the efficient frontier, it matches SLSQP solution quality at 400× less compute time (~2s vs. ~800s per λ value).

3. **HRP is the best risk-management option.** For investors prioritizing volatility control and drawdown limitation over return maximization, HRP Pearson–Ward achieves the lowest volatility (9.87%), best max drawdown (−2.61%), and strong Sharpe (2.43) in just 18 seconds.

4. **D-Wave does not yet compete at this problem scale.** In every experiment and on every metric—return, Sharpe, volatility, drawdown, consistency, and runtime—D-Wave hardware underperformed all classical solvers, including Equal Weights. The results suggest that today's quantum annealers and hybrid solvers are not yet ready for continuous, large-scale portfolio optimization.

The most important caveat: this is a backtesting study on 10 short periods. D-Wave was run on fewer periods due to cost, which limits statistical power for the quantum comparisons. Future work should include more D-Wave runs across the full 10-period set to confirm these findings.

---

*Data: 680 US stocks and ETFs, 2011–2024. Test: 10 × 30-day walk-forward windows. Optimization window: 30 days prior. No transaction costs, no forecasting.*

*Code: [github.com/bvcmartins/quantum_portfolio](https://github.com/bvcmartins/quantum_portfolio)*
