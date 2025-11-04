# Portfolio Optimization — Mean–Variance (OIM)

> *Note: The code for this project cannot be publicly displayed due to a Non-Disclosure Agreement (NDA) with the Penn State Office of Investment Management (OIM). The following summary documents the workflow, rationale, and academic grounding of the model.*

---

### Background and Context

This work began as part of my summer projects at the **Penn State Office of Investment Management (OIM)**. I reported directly to **Trajan Robertson** (Investment Analyst), under the supervision of **Jocelin Reed** (Managing Director of Investments) and **Joseph Cullen**, the **Chief Investment Officer**.  
The goal was to design and test a *mean–variance optimization (MVO)* model that could inform OIM’s asset-allocation discussions and scenario testing. What started as a purely academic exercise—replicating textbook frontier curves—quickly evolved into a pragmatic conversation about **data quality**, **correlation noise**, and **institutional usability**.

---

### Early Approach: Building Our Own Correlation Matrix

Initially, I was given a **return stream of asset classes** and asked to construct the covariance and correlation matrices from scratch.  
While straightforward in principle, this step exposed one of the core weaknesses of real-world MVO:

> Even minor instability in the correlation matrix can lead to extreme swings in optimized weights.

We experimented with:
- **Annualized sample covariances** computed from historical quarterly returns  
- **Shrinkage estimators** (Ledoit–Wolf 2003) to mitigate sampling error  
- **Random Matrix Theory (RMT)** filters to separate “noise eigenvalues” from true correlations  
- **Hierarchical and factor-based structures** to improve conditioning  

Our internal analyses—supported by references such as Laloux et al. (1999) and Plerou et al. (2002)—confirmed that roughly **80–90 % of eigenvalues** in empirical financial correlation matrices fall within the Marchenko–Pastur noise band.  
In other words, most “correlations” were statistically indistinguishable from noise.
Even after shrinkage, minor perturbations in returns produced drastically different optimal weights. This mirrors the warnings of **Michaud (1989)**, who famously wrote that “mean-variance optimization is an exercise in maximizing estimation error.”  

---

### Pivot: Institutional Stability over Statistical Purity

Given that fragility, we decided to shift focus from perfecting a noisy empirical correlation matrix to **anchoring the optimizer on a stable institutional input**.
We adopted the **JP Morgan 15 × 15 correlation matrix**, supplemented with long-run volatility assumptions by asset class. This matrix—used internally across large investment offices—is curated, smoothed, and designed to remain stable through market regimes.  
We paired it with our own **expected-return CSVs**, computed from OIM’s strategic capital-market assumptions, and embedded these inputs into a private **Streamlit application** for interactive use.

---

### Model Architecture (Conceptual Overview)

The final MVO framework minimizes portfolio variance subject to long-only constraints and optional *tracking-error penalties* relative to our policy benchmark.  

\[
\min_w \; w^T \Sigma w + \gamma \|w - w_b\|^2 
\]
subject to \( \sum_i w_i = 1, \; w_i \ge 0. \)

- **Σ** — Covariance matrix (from JP Morgan correlation × volatilities)  
- **w_b** — Benchmark (policy weights across equity, fixed income, real assets, opportunistic)  
- **γ** — Hyperparameter controlling deviation from benchmark  

Adding the tracking-error term reflects institutional preference: *optimize, but don’t drift too far*. This approach is consistent with literature on **TE-constrained MVO** in institutional portfolios (see Roll 1992; Clarke et al. 2002).

---

### Validation and Results

We evaluated the optimizer across historical backtests and simulated frontier runs. Compared with pure unconstrained MVO:

- **Tracking-error control**
- **Volatility forecasts** 
- **Weight stability** 

The application now serves as an internal sandbox for discussing **risk–return trade-offs** rather than dictating deterministic weights.

---

### Why This Matters

1. **Noise reduction** through expert priors (JP Morgan correlations) provided more actionable results than statistically “pure” but unstable estimates.  
2. **Transparency** mattered as much as accuracy—CIOs prefer to see what drives changes, not just get an output.  

---

### References  

- Markowitz, H. (1952). *Portfolio Selection*. *The Journal of Finance*.  
- Michaud, R. (1989). *The Markowitz Optimization Enigma: Is ‘Optimized’ Optimal?* *Financial Analysts Journal*.  
- Ledoit, O., & Wolf, M. (2003). *Improved Estimation of the Covariance Matrix of Stock Returns.* *Journal of Empirical Finance*.  
- Laloux, L., Cizeau, P., Bouchaud, J.-P., & Potters, M. (1999). *Noise Dressing of Financial Correlation Matrices.* *Physical Review Letters.*  
- Plerou, V., et al. (2002). *Random Matrix Approach to Stock Market Correlations.* *Physical Review E.*  
- Clarke, R., de Silva, H., & Thorley, S. (2002). *Portfolio Constraints and the Fundamental Law of Active Management.* *Financial Analysts Journal.*  

---

### Acknowledgements

Developed at **Penn State OIM**, under the mentorship of **Trajan Robertson**, with oversight by **Jocelin Reed** and **Joseph Cullen**.  
All datasets, correlations, and return expectations used in this project are proprietary to OIM or JP Morgan Asset Management.
