---
layout: post
title: "Quantum Portfolio Optimization: A Reality Check on Classical vs Quantum Performance"
date: 2025-12-15 12:00:00 -0000
categories: quantum-computing finance optimization
tags: [Quantum Computing, Portfolio Optimization, D-Wave, Finance, Benchmarking, QUBO, Machine Learning]
---

The promise of quantum computing solving complex optimization problems faster and better than classical computers is tantalizing. Portfolio optimization, with its combinatorial complexity across hundreds of assets, seems like a perfect candidate for quantum advantage. After extensively testing D-Wave's quantum annealer against classical optimization methods on a portfolio of 680 securities, I have results that challenge this assumption.

## TL;DR

**Classical optimization methods decisively outperformed quantum approaches** across all tested scenarios. The best classical solver (Scipy SLSQP) achieved 8.66% returns compared to D-Wave's -1.86% returns on the same test period, while being 2.8x faster. This project serves as an important reminder that quantum computing, while promising, isn't yet a silver bullet for real-world optimization problems.

## Project Overview

I benchmarked multiple optimization approaches on a comprehensive dataset of 680 securities (stocks and ETFs) spanning 2011-2024. The goal wasn't to build a production trading system, but to rigorously compare quantum and classical optimization methods across different problem formulations.

**Dataset Specifications**:
- 680 securities (S&P 500 stocks + major ETFs)
- 2011-2024 historical data
- 10 randomly selected 30-day test periods for walk-forward testing
- Pure historical backtesting (no forecasting) to isolate optimization method performance

**Optimization Approaches Tested**:
1. **QUBO Optimization**: Quadratic Unconstrained Binary Optimization formulations
2. **Sharpe Ratio Maximization**: Risk-adjusted return optimization
3. **Pareto Optimization**: Multi-objective efficient frontier exploration

## Methods Compared

### Quantum Approach
- **D-Wave CQM (Constrained Quadratic Model)**: Quantum annealing on D-Wave Advantage system
- Access to 5000+ qubits
- Hybrid quantum-classical solver

### Classical Baselines
- **Equal Weights**: Simple 1/n allocation (naive baseline)
- **Genetic Algorithm**: Evolutionary optimization
- **Scipy SLSQP**: Sequential Least Squares Programming (gradient-based)
- **Hierarchical Risk Parity (HRP)**: Risk-based diversification

## The Results: Classical Methods Dominate

### QUBO Optimization Performance

The first comprehensive test used QUBO formulations optimizing across 7 test periods:

![QUBO Average Performance](/images/quantum-portfolio/qubo_average_performance.png)

**D-Wave CQM Results**:
- Average return across 7 periods: **-1.90%**
- Average execution time: **7,943 seconds** (~2.2 hours per optimization)
- 100% completion rate (all 7 periods completed)

**Classical Methods Results**:
- **Scipy SLSQP**: Consistently positive returns, significantly faster
- **Genetic Algorithm**: Outperformed quantum in both speed and returns
- **Equal Weights**: Even the naive baseline performed better

### Detailed Period 0 Comparison

Looking at a single test period in detail reveals the performance gap:

| Method | Final Return | Execution Time | Performance Ratio |
|--------|--------------|----------------|-------------------|
| **Scipy SLSQP** | **+8.66%** | 3,344s | **Best performer** |
| **Genetic Algorithm** | +3.89% | 4,346s | 2.2x faster than D-Wave |
| **D-Wave CQM** | -1.86% | 9,518s | Slowest, negative returns |

The quantum solver not only underperformed on returns but was also significantly slower than classical methods.

### Cumulative Returns Across All Periods

The cumulative returns visualization across all test periods shows consistent classical superiority:

![QUBO Cumulative Returns All Periods](/images/quantum-portfolio/qubo_cumulative_returns_all_periods.png)

Notice how the classical methods (particularly Scipy SLSQP shown in blue) consistently track above the quantum approach across different market periods.

### All Methods Comprehensive Comparison

When testing across multiple optimization formulations (including Sharpe maximization and HRP), the pattern persists:

![Comprehensive Comparison All Methods](/images/quantum-portfolio/comprehensive_comparison_all_methods_all_periods.png)

This comprehensive view shows cumulative returns, final returns bar chart, execution times, and summary statistics. The classical methods occupy the top performers consistently.

![All Methods Average Performance](/images/quantum-portfolio/all_methods_average_performance.png)

The average performance comparison reinforces that classical solvers not only achieve better returns but do so more efficiently.

### Pareto Frontier Analysis

Multi-objective optimization testing the efficient frontier (risk-return tradeoffs) showed similar results:

![Pareto Efficient Frontier Combined](/images/quantum-portfolio/pareto_efficient_frontier_combined.png)

The classical methods (Scipy SLSQP, Genetic Algorithm, Riskfolio) explore more of the efficient frontier and achieve better risk-return profiles than D-Wave's quantum approach.

## Why Did Quantum Underperform?

Several factors contributed to the quantum solver's underperformance:

### 1. Problem Formulation Challenges

Portfolio optimization is naturally a continuous optimization problem. Converting it to QUBO (required for D-Wave's quantum annealer) introduces:
- **Discretization errors**: Continuous weights must be binned into discrete qubit states
- **Precision loss**: Limited qubit precision constrains weight granularity
- **Constraint encoding overhead**: Portfolio constraints (budget, diversification) become penalty terms that may not be optimally balanced

### 2. Quantum Noise and Coherence

Current quantum annealers face:
- **Environmental noise**: Quantum states are fragile
- **Limited coherence time**: Quantum advantage requires maintaining superposition long enough
- **Calibration challenges**: Qubit coupling strengths may not perfectly encode the problem

### 3. Classical Solvers Are Mature

Decades of development mean classical optimizers:
- Use sophisticated heuristics (warm starts, adaptive step sizes)
- Leverage problem structure (sparsity, convexity)
- Benefit from highly optimized numerical libraries
- Run on well-understood, stable hardware

### 4. Problem Size Sweet Spot Not Reached

Quantum advantage may require:
- Larger problem sizes than 680 assets
- Different problem structures (more combinatorial, less continuous)
- Next-generation quantum hardware with better connectivity

## What Did We Learn?

### Quantum Computing Isn't Ready for Production Finance

While quantum computing shows theoretical promise, current quantum hardware isn't competitive with classical methods for portfolio optimization. The gap is substantial both in solution quality and execution speed.

### Classical Methods Are Remarkably Effective

Modern classical optimization (Scipy SLSQP, genetic algorithms) handles high-dimensional portfolio problems (680+ assets) efficiently and effectively. They should remain the default choice for real-world applications.

### Benchmarking Matters

Rigorous, reproducible benchmarks are essential for cutting through hype. This project demonstrates the importance of:
- Testing on realistic problem sizes
- Comparing against strong classical baselines
- Measuring both solution quality AND execution time
- Being transparent about negative results

### The Right Problem Formulation Matters

Some lessons for future quantum finance applications:
- **Continuous vs discrete**: Financial optimization is often naturally continuous, poorly suited to current quantum annealers
- **Hybrid approaches**: Quantum-classical hybrids may be more promising than pure quantum
- **Problem engineering**: Success may require redesigning problems to fit quantum strengths rather than forcing quantum into classical problem structures

## Limitations and Future Directions

This study has limitations worth noting:

1. **No forecasting**: We tested pure optimization on historical data, not predictive models
2. **Limited quantum testing**: D-Wave runs were limited to reduce costs (quantum compute isn't cheap)
3. **Single quantum platform**: Only tested D-Wave; gate-based quantum computers (IBM, Google) might perform differently
4. **No transaction costs**: Real trading has costs that affect optimal rebalancing

**Future work could explore**:
- Gate-based quantum computers (QAOA on IBM/Google hardware)
- Hybrid quantum-classical algorithms specifically designed for finance
- Different problem formulations better suited to quantum annealing
- Synthetic data testing to isolate specific market conditions
- Next-generation quantum hardware (D-Wave Advantage 2, error-corrected qubits)

## Conclusions

This benchmark provides a sobering reality check on quantum portfolio optimization. While quantum computing remains promising for certain problem classes, **classical optimization methods are decisively superior for portfolio allocation across 680 securities** in terms of both solution quality and computational efficiency.

**Key Takeaways**:

1. **Classical wins (for now)**: Scipy SLSQP achieved 8.66% returns vs D-Wave's -1.86% while being 2.8x faster
2. **Quantum advantage remains elusive**: Current quantum hardware doesn't provide competitive performance for this problem class
3. **Problem formulation is critical**: Continuous optimization problems may be poorly suited to quantum annealing
4. **Transparency is essential**: Sharing negative results helps the research community make informed decisions

The quantum computing journey is far from over, but this project demonstrates we're not yet at the point where quantum solvers should replace classical methods for portfolio optimization. As quantum hardware improves and algorithms mature, this calculus may change—but for now, stick with proven classical approaches for production finance applications.

## Reproducibility

All code, data, and results are available on [GitHub](https://github.com/bvcmartins/quantum_portfolio). The project uses:
- Fixed random seeds for reproducibility
- Version-controlled dependencies
- Documented hyperparameters
- Publicly available market data

If you're working on quantum finance applications, I encourage you to:
- Run rigorous benchmarks against strong classical baselines
- Share negative results (publication bias favors positive results)
- Test on realistic problem sizes
- Measure both solution quality and execution time

Let's build quantum computing applications on solid empirical foundations, not hype.

---

**Questions or feedback?** Open an issue on the [GitHub repository](https://github.com/bvcmartins/quantum_portfolio) or reach out on [LinkedIn](https://linkedin.com/in/bvcmartins).

**Citation**: If you use this work in your research:

```bibtex
@software{quantum_portfolio_2024,
  title = {Quantum Portfolio Optimization: Benchmarking Classical and Quantum Methods},
  author = {Martins, B.V.C.},
  year = {2024},
  url = {https://github.com/bvcmartins/quantum_portfolio}
}
```
