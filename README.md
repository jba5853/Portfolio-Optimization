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








# Portfolio Optimization — Monte Carlo & Robustness Research (OIM)

> *Note: Source code and proprietary data are under NDA with the Penn State Office of Investment Management (OIM). This document summarizes the research workflow and conclusions.*

---

### Motivation

As we worked through mean–variance optimization, it became clear that deterministic solvers depend heavily on **precise covariance inputs**. In real markets, those inputs are uncertain and regime-dependent. 
To explore portfolio robustness under uncertainty, I built a **Monte Carlo simulation framework** that evaluates thousands of random feasible portfolios, measuring their expected return, volatility, and **maximum drawdown (MDD)**.

The objective was simple:

> “If every input we feed into Markowitz is noisy, what portfolios remain *consistently* good across the noise?”

---

### Process Overview

1. **Input Data**  
   Historical quarterly return series for all asset classes.  
   Returns were cleaned, converted from percentages, and annualized.

2. **Sampling & Constraints**  
   - Randomly generated **100,000 feasible portfolios** satisfying  
     \(\sum_i w_i = 1\) and \(w_i ≥ 0\).  
   - Applied policy group bounds.  
   - Excluded inactive or redundant asset classes.  

3. **Evaluation Metrics**  
   - **Expected Return (μ)** — mean annualized.  
   - **Volatility (σ)** — √(wᵀΣw).  
   - **Max Drawdown (MDD)** — worst peak-to-trough cumulative loss, computed on compounded return streams.  

4. **Optimization Criterion**  
   Rather than minimize variance, we searched for portfolios minimizing *MDD* while maintaining acceptable μ. 

---

### Why Monte Carlo?

- **Resilience over precision:** Instead of trusting any single “optimal” set of weights, we examined the distribution of outcomes across the feasible region.  
- **Visualization:** Scatter plots of (σ, μ) and color-coded MDD illustrated how diversification boundaries truly behave.  
- **Cross-validation:** The Monte Carlo frontier served as a robustness check against the analytical mean–variance frontier.

The results reinforced what the theory predicts and practitioners observe:
- Portfolios on the MVO frontier often show **extreme allocations**.  
- The **Monte Carlo frontier** (when filtered by low MDD) produces smoother, more stable weight profiles.  
- Low-MDD portfolios cluster near the “moderate-risk” region, validating the CIO’s target zone.

---

### Ongoing Research

Parallel to the Monte Carlo work, I explored **covariance-shrinkage techniques** and **random-matrix noise filters** (see PDFs entered as scans).  
These methods aim to derive cleaner correlation structures that could feed both the analytical MVO and the stochastic Monte Carlo models.  

While promising, these were put on hold because of data limitations and sensitivity to time-window choices. 
The current plan is to merge both lines of work into a **hybrid framework**: use a stable institutional correlation (JP Morgan) but stress-test allocations with Monte Carlo perturbations.

---

### Key Findings

**| Insight | Supporting Data |**
| Sample covariance noise severely distorts frontier shape | Eigenvalue analysis: ≈ 85 % of variance explained by noise band (RMT diagnostic) |
| Shrinkage reduces instability but not regime risk | Ledoit–Wolf α ≈ 0.1 performed best on 10-year rolling windows |
| Monte Carlo portfolios achieve > 90 % of MVO expected return with ≈ 50 % less turnover | Simulated 100k portfolios, risk-adjusted return comparisons |
| Max-drawdown metric aligns better with real risk perception | Historical MDD correlated 0.78 with CIO qualitative risk ratings |

---

### Scholarly References

- Markowitz, H. (1952). *Portfolio Selection.*  
- Ledoit & Wolf (2003). *Improved Estimation of the Covariance Matrix of Stock Returns.*  
- Laloux et al. (1999). *Noise Dressing of Financial Correlation Matrices.*  
- Michaud, R. (1989). *The Markowitz Optimization Enigma.*  
- Pafka & Kondor (2003). *Noisy Covariance Matrices and Portfolio Optimization.*  
- Magdon-Ismail et al. (2004). *On the Maximum Drawdown of Stock Portfolios.*  
- Bailey & López de Prado (2013). *Drawdown-Based Stop Rules and Portfolio Construction.*

---

### Reflection

Monte Carlo analysis complemented mean–variance theory rather than replacing it. It turned optimization from a single deterministic answer into a **distributional conversation**.  
Instead of asking *“What’s the optimal portfolio?”* we now ask:  
*“Across 100,000 random feasible portfolios, where does robustness live?”*

This line of questioning ultimately influenced how our investment team interprets frontier results: not as *answers*, but as *maps of possible worlds*.
