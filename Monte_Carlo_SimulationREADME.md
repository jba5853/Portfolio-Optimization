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

---

### Acknowledgements

Research conducted at the **Penn State Office of Investment Management** under **Trajan Robertson**, with oversight from **Jocelin Reed** and **Joseph Cullen**.  
All underlying data and scripts remain proprietary under OIM’s NDA.
