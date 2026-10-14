**Authors:** Reiner Franke and Frank Westerhoff  
**Year:** 2012
## Key takeaways

- The model has two types of investors: fundamentalists (value investors) and chartists (momentum investors).
- Investors can switch strategies, so the mix of investor types changes over time. This can produce changing volatility without explicitly programming a volatility process.
- The authors estimate behavioral parameters from real S&P 500 returns using the **method of simulated moments (MSM)**.
- Good agreement with historical summary statistics does **not** establish out-of-sample forecasting ability or prove that the assumed behavioral mechanisms are correct.

## Model structure

Let $p_t$ be the log asset price and $p^*$ its fundamental log value.

### Fundamentalists (value investors)

$$
D_t^f = \phi(p^* - p_t) + \epsilon_t^f
$$

Fundamentalists buy when the asset is undervalued and sell when it is overvalued. The coefficient $\phi$ determines how strongly they respond to mispricing; $\epsilon_t^f$ is random demand noise. Negative demand represents selling pressure, and the model does not impose an explicit short-selling constraint.

### Chartists (momentum investors)

$$
D_t^c = \chi(p_t - p_{t-1}) + \epsilon_t^c
$$

Chartists buy after prices rise and sell after they fall. The coefficient $\chi$ controls the strength of trend following; $\epsilon_t^c$ is random demand noise. The demand shocks have standard deviations $\sigma_f$ and $\sigma_c$, respectively.

### Price formation

$$
p_{t+1} = p_t + \mu\left(n_t^f D_t^f + n_t^c D_t^c\right),
\qquad n_t^f + n_t^c = 1
$$

A market maker moves the price in response to total buying or selling pressure. Here $n_t^f$ and $n_t^c$ are the fractions of fundamentalists and chartists, and $\mu$ sets how quickly prices adjust.

### Switching between strategies

$$
n_t^f = \frac{1}{1+\exp(-\beta A_t)},
\qquad n_t^c = 1-n_t^f
$$

$A_t$ is the attractiveness of fundamentalism **relative to chartism**. If $A_t$ is positive, more investors become fundamentalists. $\beta$ controls sensitivity to differences in attractiveness.

The paper considers combinations of four reasons to switch: an inherent preference for one strategy, past relative performance, herding, and the extent of mispricing. A general expression illustrating these mechanisms is:

$$
A_t = \alpha_0 + \alpha_w(w_t^f-w_t^c)
+ \alpha_n(n_t^f-n_t^c) + \alpha_p(p_t-p^*)^2
$$

Here $w_t^f-w_t^c$ represents relative accumulated performance. The **squared** mispricing term makes fundamentalism more attractive when an asset is either far above or far below fundamental value: fundamentalists can sell an overpriced asset as well as buy an underpriced one.

The paper compares **seven variants** that differ mainly in which of these switching mechanisms they include. The DCA-HPM variant uses predisposition, herding, and mispricing (but not relative wealth):

$$
A_t = \alpha_0 + \alpha_n(n_t^f-n_t^c) + \alpha_p(p_t-p^*)^2
$$

## Empirical data and calibration targets

The authors use daily **S&P 500 returns from January 1980 to March 2007** (6,866 observations). Returns are calculated from log price changes, expressed in percentage points.

Instead of matching the historical price path, they match **nine summary statistics**:

1. Lag-1 autocorrelation of returns.
2. Mean absolute return, as a measure of the typical size of daily moves.
3. A Hill tail-index statistic, describing extreme returns.
4. Autocorrelations of absolute returns at six lags: 1, 5, 10, 25, 50, and 100 trading days (implemented using nearby-lag averages).

Absolute return $|r_t|$ is a proxy for volatility. The average $|r_t|$ measures its overall scale; autocorrelations of $|r_t|$ measure **volatility clustering**. Compared with squared returns, absolute returns are less dominated by extreme observations.

## Calibration: method of simulated moments

Let $m$ be the nine moments from real returns, and $\hat m(\theta)$ the corresponding moments from the ABM simulated with parameters $\theta$.

$$
\hat\theta = \arg\min_\theta L(\theta),
\qquad
L(\theta) = \big(\hat m(\theta)-m\big)^\top
W\big(\hat m(\theta)-m\big)
$$

The weighting matrix $W$ determines how much each mismatch matters. If $W$ is diagonal, this is just a weighted sum of squared errors. A full matrix also accounts for correlation between moment-estimation errors.

For the DCA-HPM variant, the seven estimated parameters are:

$$
\theta=(\phi,\chi,\sigma_f,\sigma_c,\alpha_0,\alpha_n,\alpha_p)
$$

These represent response to value, response to momentum, the two demand-noise scales, baseline strategy preference, herding strength, and response to mispricing. Other model settings, such as the price adjustment coefficient and choice sensitivity, are fixed rather than estimated.

The authors search for parameters using **Nelder–Mead**, a derivative-free optimization method. Because the loss can have local minima, they use restarts; a global optimum is not guaranteed.

### Practical choices

- **Long simulations:** Each evaluation uses a simulated series about ten times as long as the empirical series, reducing random error in $\hat m(\theta)$.
- **Common random numbers:** The same underlying random draws are used when comparing candidate parameters, making the objective less noisy.
- **Bootstrap weights:** The authors use block resampling of historical returns to estimate uncertainty and covariance of the empirical moments, then use the inverse covariance as $W$. Blocks preserve local time dependence better than resampling individual days, but may still miss long-range dependence or structural change.

## Evaluating and comparing the variants

Besides the minimized MSM loss, the authors use a **moment coverage ratio (MCR)**: the fraction of repeated simulations in which **all nine** simulated summary statistics fall within their respective empirical confidence intervals.

- **MSM loss:** How close is the simulated average behavior to the historical targets?
- **MCR:** How often does a simulation produce moments that fall within the chosen empirical ranges?

The authors compare seven switching-rule variants using these measures. A better in-sample fit does not automatically mean a better explanation of actual trader behavior.

## Limitations and questions

- **Out-of-sample generalization:** The paper primarily calibrates and compares models against the same historical period. It does not provide a conventional train/test demonstration that the calibrated model reproduces moments in a later, unseen period.
- **Identifiability:** Different behaviors—such as leverage-driven selling or strategy switching—can generate similar aggregate return statistics. Matching returns alone may not identify the true mechanism.
- **Model complexity and overfitting:** A more flexible switching rule can fit historical moments better simply because it has more adjustable parameters.
- **Bootstrap assumptions:** Resampling contiguous blocks cannot guarantee a realistic alternative financial history, particularly under regime changes and long-range dependence. Its narrower purpose is to estimate uncertainty in the chosen moments.

**Bottom line:** This is a concrete example of fitting a financial ABM to real data. It shows how to infer behavioral parameters indirectly from market-level statistics, while also illustrating why historical fit is different from identifying a mechanism or demonstrating generalization.
