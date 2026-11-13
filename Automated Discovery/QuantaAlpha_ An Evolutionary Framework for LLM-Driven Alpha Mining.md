# QuantaAlpha: An Evolutionary Framework for LLM-Driven Alpha Mining

**Authors:** Jun Han, Shuo Zhang, Wei Li, Yifan Dong, Tu Hu, Yumo Zhu, Xiaomin Yu, Xin Guo, Zhaowei Liu, Kunyi Wang, Jingping Liu, Tianyi Jiang, Ruichuan An, Sen Hu, Zhi Yang, Ronghao Chen, Huacan Wang  
**Year:** 2026  
**Paper:** https://arxiv.org/abs/2602.07085

QuantaAlpha uses LLM agents and evolutionary search to discover interpretable stock-return factors. It tests whether the resulting factor library improves daily stock selection.

## Problem setup

The goal is to find a factor—a mathematical expression that turns market data into a score for each stock—that predicts relative future returns. Equation (1) is:

$$
f^*=\arg\max_{f\in\mathcal F}\left[\mathcal L(f(\mathbf X),\mathbf y)-\lambda\mathcal R(f)\right].
$$

Here, $f$ is a candidate factor; $f^*$ is the best factor under the objective; $\mathcal F$ is the search space of allowed expressions; $\mathbf X$ is market data across stocks, dates, and features; $\mathbf y$ is the realized future stock-return target; $\mathcal L$ measures predictive usefulness; $\mathcal R$ penalizes complexity and redundancy; and $\lambda$ controls that penalty. The equation **maximizes a reward**, not a conventional training loss. The paper does not specify a unique functional form for $\mathcal L$.

The target is the close-to-close return from day $t+1$ to day $t+2$, rather than from $t$ to $t+1$. Factors can use day-$t$ closing data, so assuming a trade at that same close after observing the data would introduce look-ahead bias.

A representative choice for predictive utility would be mean daily information coefficient (IC): the average, across dates, of the Pearson correlation across stocks between factor scores and next-day returns. Equivalently, one could minimize **negative mean IC**. This is an illustrative objective, not an explicitly specified implementation of Equation (1).

## Alpha mining process

1. **Hypothesis generation:** An LLM proposes an economic or market-behavior explanation for a possible predictive signal, drawing on market context and prior findings.
2. **Factor construction:** Another agent converts the hypothesis into a symbolic formula using a restricted library of time-series and cross-sectional operators, then compiles it into code.
3. **Verification and filtering:** The system checks that the formula and code match the hypothesis, repairs failures, and rejects overly complex or redundant factors.
4. **Evaluation:** It backtests the factor, measuring predictive correlations and strategy performance. The feedback informs later searches.

## Evolutionary alpha mining

The search starts from diverse seed ideas drawn from the public **Alpha158(20)** factor subset. A planning agent proposes ten complementary research directions, varying signal sources, lookback windows, and market mechanisms. Each hypothesis is turned into a candidate factor and evaluated.

The framework then repeats two operations:

- **Mutation:** Inspect a weak research step and revise that part—for example, change the hypothesis, time horizon, formula, or implementation—then reevaluate the result.
- **Crossover:** Combine useful reasoning and construction choices from several high-performing prior attempts into a new coherent hypothesis and factor.

The main experiment uses five iterations. Factors are also filtered for structural similarity and output correlation, so the final library is not filled with near-duplicates.

## Experimental setup

The main market is the **CSI 300**, a universe of 300 large-cap Chinese stocks. Inputs are daily open, high, low, close, volume, and volume-weighted average price. The target is a continuous next-day return, defined in the paper as the close-to-close return from $t+1$ to $t+2$. Features and labels undergo cross-sectional rank normalization.

The chronological split is **training: January 1, 2016–December 31, 2020; validation: January 1–December 31, 2021; and testing: January 1, 2022–December 26, 2025**.

## Trading strategy and evaluation

Roughly **150 validated factors** are calculated for every stock and fed into a downstream **LightGBM** model, which predicts next-day relative returns. Each day, stocks are ranked by predicted score. The **TopkDropoutStrategy** holds **50 equally weighted stocks**, sells the **5 lowest-ranked current holdings**, and replaces them with high-ranked candidates. Trades use the next day's opening price.

The reported annualized return (ARR) is **annualized excess return relative to buy-and-hold CSI 300**, after modeled trading costs (0.05% to buy and 0.15% to sell). Thus, an ARR of 4.68% does **not** mean a 4.68% absolute portfolio return. The paper also reports the information ratio and maximum drawdown of cumulative excess returns, plus factor IC and Rank IC. It does not list buy-and-hold CSI 300 as a separate strategy row in its main comparison.

## Main results

- **Factor prediction:** With GPT-5.2, QuantaAlpha reaches IC **0.0472** on the CSI 300 test period, versus **0.0286** for RD-Agent and **0.0347** for AlphaAgent with the same LLM.
- **Trading:** The GPT-5.2 version earns **4.68% annualized excess return**, with an information ratio of **0.6453** and **11.80% maximum drawdown** of cumulative excess returns.
- **Compared with other approaches:** QuantaAlpha improves over the competing LLM-based factor-mining systems in this setting, but the deep-learning **TRA** baseline reports a higher annualized excess return (**6.81%**) and lower drawdown (**8.51%**).
- **Transfer:** Factors mined on CSI 300 are also tested on CSI 500 and S&P 500. The paper reports approximately **40.28%** and **19.1%** cumulative excess returns, respectively, over the four-year test window.

The main contribution is automated discovery of interpretable predictive features. The downstream prediction model and 50-stock trading rule are conventional; the evolutionary search is the novel component.
