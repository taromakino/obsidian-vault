**Author:** Sébastien Barde  
**Year:** 2016

## 1. Research objective

Barde asks whether agent-based financial models can describe real return data as well as conventional ARCH-type volatility models. This note focuses on the comparison involving the Franke–Westerhoff (FW) model.

A good model in this setting is not one whose simulated prices trace the historical S&P 500 price path. It is one whose simulated returns behave like real returns, including the way today's return probabilities depend on recent market history.

## 2. Why compare conditional return distributions?

One possibility would be to compare the overall distribution of simulated returns with the overall distribution of observed S&P 500 returns. But that ignores the order of observations. A return series with clusters of large price changes and a shuffled version of the same series have identical overall distributions, even though their volatility dynamics are different.

Both FW and ARCH are meant to capture changes in volatility over time. Barde therefore examines **conditional return probabilities**: given the recent sequence of returns, how much probability does a model assign to the return that happens next?

This lets the comparison assess temporal dependence, not just the frequency of small and large returns.

## 3. Generating synthetic returns from FW

FW models an artificial market containing two broad trading strategies:

- **Fundamentalists** trade according to the gap between the market price and a fundamental value.
- **Chartists** trade according to recent price movements.

Their demands contain random disturbances. The market price responds to aggregate excess demand, while the relative presence of chartists and fundamentalists changes over time. These shifts help produce alternating periods of high and low volatility.

To generate data, the researcher chooses a parameter configuration and starting state, runs the model forward, and records the resulting prices. Consecutive simulated prices are then converted into simulated returns. The simulated dates do not need to line up with the dates of actual S&P 500 returns.

## 4. Generating synthetic returns from ARCH

ARCH starts from an explicit statistical rule for conditional variance. For a simple zero-mean ARCH model,

$$
\sigma_t^2=\omega+\alpha r_{t-1}^2
$$

Here, $\sigma_t^2$ is the conditional variance of today's return, and $r_{t-1}$ is yesterday's return. The parameters $\omega$ and $\alpha$ are estimated from observed returns.

A second equation specifies how a return is generated from that conditional variance:

$$
r_t=\sigma_t\epsilon_t
$$

The random shock $\epsilon_t$ has mean zero and variance one. Thus $\sigma_t^2$ is the variance of $r_t$ conditional on previous returns.

Once the model is fitted, generate a new return series by repeating the following steps:

1. **Initialize:** Choose a starting synthetic return $r_0$.
2. **First period's variance:** Calculate $\sigma_1^2=\omega+\alpha r_0^2$, then set $\sigma_1=\sqrt{\sigma_1^2}$.
3. **First period's return:** Draw a new random shock $\epsilon_1$ and calculate $r_1=\sigma_1\epsilon_1$.
4. **Second period's variance:** Use the *simulated* return $r_1$ to calculate $\sigma_2^2=\omega+\alpha r_1^2$.
5. **Second period's return:** Draw a fresh shock $\epsilon_2$ and calculate $r_2=\sigma_2\epsilon_2$, where $\sigma_2=\sqrt{\sigma_2^2}$.
6. **Continue:** At each period $t$, calculate $\sigma_t^2$ from the previous synthetic return, draw $\epsilon_t$, and generate $r_t=\sigma_t\epsilon_t$.

Historical S&P 500 returns are used to fit $\omega$ and $\alpha$. Once simulation starts, each new return is based on the *previous synthetic return*, not the corresponding historical return.

## 5. Why discretize returns?

ARCH has an explicit probability model. FW generates returns through simulation, so its exact conditional probability distribution is difficult to write down.

Barde puts both models on common ground by turning continuous returns into discrete categories, or **bins**. The same bin boundaries must be applied to simulated FW returns, simulated ARCH returns, and observed S&P 500 returns. Otherwise, scores would refer to different outcomes and would not be directly comparable.

Barde uses a return range of **−30% to +30%**, divided into **256 equally spaced bins** (an 8-bit discretization). Each bin is about **0.234 percentage points** wide. Returns outside the range are assigned to the nearest boundary bin. The same settings are applied to FW, ARCH, and the observed returns.

Binning makes it possible to learn conditional probabilities from sequences of simulated categories. Its cost is the loss of information about the exact return within a bin.

## 6. Context-Tree Weighting (CTW)

After discretization, the question is how much history to use when estimating the probability of the next return category.

A very short history has many observations behind it but may miss important patterns. A long, highly specific history may be more informative but occurs rarely, so its estimated probabilities can be unreliable.

**Context-Tree Weighting (CTW)** balances those possibilities. It combines probability estimates based on histories of different lengths rather than relying on a single fixed lookback window. Longer histories contribute when supported by sufficient data; shorter histories help when detailed patterns are too sparse.

Barde applies this procedure to synthetic sequences from each candidate model. It produces an estimate of the probabilities that the model assigns to possible next return bins, conditional on earlier bins.

## 7. Evaluating against observed S&P 500 returns

Once the synthetic FW and ARCH data have each been used to construct their CTW probability estimates, Barde evaluates them on the **same observed S&P 500 return sequence**.

For each observed day:

1. Convert the historical return and its preceding returns into bins.
2. Use the model's CTW estimates to obtain the probability of the observed bin, given the preceding bins.
3. Record the probability assigned to what actually occurred.

Repeat this over the historical sequence. A model performs better when it consistently assigns more probability to observed outcomes. The historical prices themselves need not match any synthetic price path.

## 8. Markov Information Criterion (MIC)

The **Markov Information Criterion (MIC)** turns the conditional-probability evaluations into a comparable information score.

The basic intuition is that an outcome assigned high probability is unsurprising under the model, while an outcome assigned low probability is costly to explain. Negative log probabilities express that cost: a lower value is better.

For intuition, a simplified score is

$$
-\sum_t \log P_M(r_t\mid\text{earlier returns})
$$

where $P_M$ refers to the probabilities associated with model $M$. In the actual calculation, CTW works with discretized returns and the MIC accounts for the probability-estimation procedure; the displayed expression is not the complete MIC formula.

Because both FW and ARCH are evaluated using the same return categories and scoring framework, **lower MIC means a better statistical fit** to the observed return sequence.

## 9. Parameter selection and statistical significance

The comparison is not limited to one arbitrary FW setup. Barde considers multiple candidate parameter configurations and ARCH-type benchmark specifications, then evaluates them using the common scoring procedure.

Choosing whichever model has the lowest score is not enough to show a meaningful advantage. Differences could be small relative to sampling uncertainty.

Barde therefore uses a **Model Confidence Set** procedure to identify models whose performance cannot be statistically distinguished from the best performers. This separates a small numerical lead from evidence of a statistically reliable improvement.

## 10. Results and interpretation

FW is the strongest of the agent-based model families examined in Barde's comparison. Its statistical fit is competitive with the ARCH-type benchmarks. For the S&P 500, FW can achieve a slightly better MIC score, but the difference does not establish a statistically significant advantage over ARCH.

The paper also finds that the relative performance of agent-based models can improve during market turmoil, consistent with the relevance of the herding mechanisms they represent.

**What the comparison establishes:** FW can generate a return process whose conditional statistical behavior is about as convincing as that of conventional ARCH-type alternatives under this evaluation.

**What it does not establish:** It is not a demonstration that FW produces more accurate forecasts for future, independently held-out S&P 500 returns or volatility. Estimating models, selecting configurations, and scoring the observed historical sequence are different from freezing a model and evaluating genuinely unseen future data.