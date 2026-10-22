**Authors:** Maria Cristina Recchioni, Gabriele Tedeschi, and Mauro Gallegati  
**Year:** 2015

## Research question

Can a financial agent-based model be calibrated to observed stock prices and forecast future prices?

The paper calibrates a two-strategy Brock–Hommes model against daily prices for four stock indices. It then evaluates the model on held-out returns and one- and two-day price forecasts.

## 1. Market setup

Investors can hold:

- A **risk-free asset**, earning gross return $R=1+r$. The daily interest rate $r=0.01/250$ is fixed.
- A **stock**, with price $p_t$ at time $t$. Its price is solved through market clearing.

The stock's **fundamental price** $p^*$ is a constant calibrated from observed prices.

Investors choose their stock holdings. The two types differ in their forecasts:

- **Fundamentalists** expect the price to return to $p^*$.
- **Chartists** expect past deviations from $p^*$ to persist.

## 2. Expectations

Define the deviation of the market price from fundamental value:
$$
x_t=p_t-p^*.
$$
The two forecast rules specify the expected next-period payoff from one share:

**Fundamentalists:**
$$
\mu_{f,t}=Rp^*.
$$
**Chartists:**
$$
\mu_{c,t}=Rp^*+gx_{t-1}.
$$
Here $\mu_{f,t}$ and $\mu_{c,t}$ are the two expected payoffs. The chartists' extrapolation strength $g$ is calibrated. The deviation $x_{t-1}$ is calculated from the previous price, either simulated (BH) or observed (ABH).

## 3. Investor demand

Each investor type $h\in\{f,c\}$ chooses the number of shares $z$ to maximize expected excess payoff minus a risk penalty. Risk aversion $a>0$ is calibrated, and perceived payoff variance $\sigma^2$ is fixed ($\sigma=0.1$):
$$
\max_z\; z(\mu_{h,t}-Rp_t)-\frac a2 z^2\sigma^2.
$$
Setting the first derivative with respect to $z$ equal to zero (the **first-order condition**) gives the optimal holding:
$$
z_{h,t}=\frac{\mu_{h,t}-Rp_t}{a\sigma^2}.
$$
Substituting the forecast for each investor type gives:

**Fundamentalists:**
$$
z_{f,t}=\frac{\overbrace{R(p^*-p_t)}^{\text{Mean reversion}}}{a\sigma^2}.
$$
**Chartists (trend-followers):**
$$
z_{c,t}=\frac{\overbrace{R(p^*-p_t)}^{\text{Mean reversion}}+\overbrace{g(p_{t-1}-p^*)}^{\text{Trend extrapolation}}}{a\sigma^2}.
$$
Here $\mu_{h,t}$ is the expected payoff defined in Section 2 for that investor type. Each investor takes the current price $p_t$ as given when choosing $z$.

A positive holding is a long position; a negative holding is a short position.

## 4. Market clearing

Let $n_{f,t}$ and $n_{c,t}$ be the fractions of fundamentalists and chartists. The model computes them from the strategy-switching rule in Section 6; they sum to one.

The stock has zero net outside supply, so the positions must cancel:
$$
n_{f,t}z_{f,t}+n_{c,t}z_{c,t}=0.
$$
Substitute $z_{h,t}=(\mu_{h,t}-Rp_t)/(a\sigma^2)$ for both investor types:
$$
n_{f,t}\frac{\mu_{f,t}-Rp_t}{a\sigma^2}+n_{c,t}\frac{\mu_{c,t}-Rp_t}{a\sigma^2}=0.
$$
Multiplying by $a\sigma^2$ and expanding gives
$$
n_{f,t}\mu_{f,t}+n_{c,t}\mu_{c,t}-Rp_t(n_{f,t}+n_{c,t})=0.
$$
Since $n_{f,t}+n_{c,t}=1$, this simplifies to
$$
Rp_t=n_{f,t}\mu_{f,t}+n_{c,t}\mu_{c,t},
$$
so the market-clearing price is
$$
p_t=\frac{n_{f,t}\mu_{f,t}+n_{c,t}\mu_{c,t}}{R}.
$$
Thus the market-clearing price is the discounted average forecast, weighted by the fractions of the two trader types. Market clearing gives the price directly.

## 5. Price dynamics

Substituting the forecast rules into market clearing:
$$
\begin{aligned}
p_t&=\frac{n_{f,t}Rp^*+n_{c,t}(Rp^*+gx_{t-1})}{R}\\
&=p^*+\frac{g}{R}n_{c,t}x_{t-1}.
\end{aligned}
$$
Since $x_t=p_t-p^*$,
$$
x_t=\frac{g}{R}n_{c,t}x_{t-1}.
$$
The price deviation depends on the previous deviation, the calibrated parameter $g$, and the chartist fraction calculated from past trading performance. In the simulation algorithm, the price at time $t$ uses the previously calculated fraction $n_{c,t-1}$.

## 6. Switching between strategies

Investors can switch between fundamentalist and chartist strategies based on their past profitability. Strategies that have earned higher profits attract more investors.

The model assumes a constant fundamental price $p^*$ and a dividend $y=rp^*$, where $r$ is the risk-free interest rate and $R=1+r$. The **realized excess payoff per share** from holding the stock from time $t-1$ to $t$ is
$$
q_t=p_t+y-Rp_{t-1}.
$$
Substituting $y=rp^*$ and using $r=R-1$ gives
$$
\begin{aligned}
q_t&=p_t+rp^*-Rp_{t-1}\\
&=p_t+(R-1)p^*-Rp_{t-1}\\
&=(p_t-p^*)-R(p_{t-1}-p^*).
\end{aligned}
$$
Recall that $x_t=p_t-p^*$. Therefore,
$$
q_t=x_t-Rx_{t-1}.
$$
This measures the payoff from holding the stock instead of the risk-free asset. A positive $q_t$ means the stock outperformed the risk-free asset.

The profit earned by strategy $h$ at time $t$ equals the realized excess payoff per share multiplied by the shares held from the previous period:
$$
\pi_{h,t}=q_tz_{h,t-1}.
$$
Both strategies face the same realized payoff $q_t$, but their profits differ because they chose different stock positions.

Each strategy has a fitness score $U_{h,t}$ that measures accumulated profits. In the calibrated model, strategy costs are zero and all past profits are retained (memory parameter $\omega=1$). Thus,
$$
\begin{aligned}
U_{f,t}&=U_{f,t-1}+\pi_{f,t},\\
U_{c,t}&=U_{c,t-1}+\pi_{c,t}.
\end{aligned}
$$
Fitness increases when a strategy earns profits and decreases when it incurs losses. Investors favor strategies with higher fitness. The fraction of chartists follows the **softmax** rule:
$$
n_{c,t}=\frac{e^{\beta U_{c,t}}}{e^{\beta U_{f,t}}+e^{\beta U_{c,t}}},\qquad n_{f,t}=1-n_{c,t}.
$$
The parameter $\beta\ge 0$ measures how strongly investors respond to fitness differences. At $\beta=0$, half the investors follow each strategy regardless of profitability. For small $\beta$, profitability has little influence; for large $\beta$, investors strongly favor the more profitable strategy. The initial fractions are $n_{f,0}=n_{c,0}=1/2$.

## 7. A single iteration of the model

The model evolves one period at a time. Each iteration determines a new stock price, calculates realized profits, and updates the fractions of investors following the two strategies.

At the start of period $t$, the inputs are the previous price deviation $x_{t-1}$, the previously chosen positions $z_{f,t-1}$ and $z_{c,t-1}$, the fitness scores $U_{f,t-1}$ and $U_{c,t-1}$, and the investor fractions $n_{f,t-1}$ and $n_{c,t-1}$. The parameters $R$, $g$, $p^*$, $a$, $\sigma$, and $\beta$ are fixed during the iteration.

**Step 1. Calculate the forecasts.** Using the previous price deviation, the two investor types forecast
$$
\begin{aligned}
\mu_{f,t}&=Rp^*,\\
\mu_{c,t}&=Rp^*+gx_{t-1}.
\end{aligned}
$$
**Step 2. Determine the new price and positions.** Market clearing uses the fractions from the previous period:
$$
p_t=\frac{n_{f,t-1}\mu_{f,t}+n_{c,t-1}\mu_{c,t}}{R}=p^*+\frac{g}{R}n_{c,t-1}x_{t-1}.
$$
Calculate the new deviation $x_t=p_t-p^*$ and the positions chosen at time $t$:
$$
z_{h,t}=\frac{\mu_{h,t}-Rp_t}{a\sigma^2}.
$$
These new positions are retained for calculating profits in the next period; they are not used to calculate profits realized at time $t$.

**Step 3. Calculate the realized excess payoff.** The new price determines the payoff from holding the stock during the previous period:
$$
q_t=x_t-Rx_{t-1}.
$$
**Step 4. Calculate profits.** Multiply this payoff by the positions already chosen at time $t-1$:
$$
\pi_{h,t}=q_tz_{h,t-1}.
$$
There is no need to recalculate $z_{h,t-1}$; it is an input from the previous iteration.

**Step 5. Update fitness.** Add the new profits to the accumulated scores:
$$
\begin{aligned}
U_{f,t}&=U_{f,t-1}+\pi_{f,t},\\
U_{c,t}&=U_{c,t-1}+\pi_{c,t}.
\end{aligned}
$$
**Step 6. Update investor fractions.** Apply the softmax rule to the new scores:
$$
n_{c,t}=\frac{e^{\beta U_{c,t}}}{e^{\beta U_{f,t}}+e^{\beta U_{c,t}}},\qquad n_{f,t}=1-n_{c,t}.
$$
The iteration is complete. The updated investor fractions and the positions chosen at time $t$ become inputs for period $t+1$.

## 8. Calibration

The four calibrated parameters are
$$
\theta=(\beta,g,p^*,a).
$$

| Parameter | Meaning |
|---|---|
| $\beta$ | Sensitivity to differences in strategy fitness |
| $g$ | Chartists' extrapolation strength |
| $p^*$ | Constant fundamental price |
| $a$ | Risk aversion (denoted $\alpha$ in the paper) |

For any candidate $\theta$, the model generates a price series $\hat p_t(\theta)$. The observed index price is $p_t^{\mathrm{obs}}$. Calibration minimizes squared relative errors:
$$
L(\theta)=\sum_{t=1}^{T}\left(\frac{\hat p_t(\theta)-p_t^{\mathrm{obs}}}{p_t^{\mathrm{obs}}}\right)^2.
$$
A constrained, variable-metric steepest-descent algorithm solves this problem. The model generates prices through market clearing while the optimizer varies the four parameters. The daily risk-free rate, perceived variance, memory setting, and initial trader fractions remain fixed.

The calibration uses the first 200 trading days (February 25–December 16, 2011) for each of four indices: S&P 500, Euro Stoxx 50, Nikkei 225, and CSI 300.

## 9. BH and ABH

The two versions use the same investor rules and calibration objective. They differ in **which price history is fed into the model**.

### BH: Brock–Hommes

BH calculates each new price from its own previous simulated prices. For example, chartists use the deviation
$$
\hat x_{t-1}=\hat p_{t-1}-p^*.
$$
These simulated prices also enter the lagged calculations of strategy fitness. Errors in earlier simulated prices therefore carry forward.

### ABH: Adjusted Brock–Hommes

When predicting the price at time $t$, ABH instead uses real market prices available through $t-1$. In particular, chartists use
$$
x_{t-1}^{\mathrm{obs}}=p_{t-1}^{\mathrm{obs}}-p^*.
$$
Previously observed prices also replace simulated prices in the lagged calculations of profits, fitness, and trader fractions. The new price $\hat p_t$ is still **solved by market clearing**. ABH never uses the actual price at $t$ to predict that same price.

For example, after forecasting tomorrow's price, ABH incorporates tomorrow's **actual** price once it becomes available, before making the next one-day forecast.

### Two-day forecasts

At a forecast origin $\tau$, observed prices are available only through $\tau$:

1. ABH predicts $\hat p_{\tau+1}$ using the observed history through $\tau$.
2. It predicts $\hat p_{\tau+2}$ using its simulated $\hat p_{\tau+1}$, because the actual price at $\tau+1$ is not yet known.

Thus ABH uses observed prices up to the forecast origin and simulated prices beyond it.

## 10. Evaluation

### Held-out return distributions

The first 200 trading days are used for calibration and days 201–245 (December 19, 2011–February 23, 2012) for evaluation. The paper compares observed log-return distributions with simulated BH and ABH returns. For this experiment, it generates 500 simulated paths with random shocks and reports distributional similarity using a generalized Kolmogorov–Smirnov test.

This tests whether the model reproduces the distribution of returns, not whether it predicts their exact sequence.

### Rolling forecasts

For one- and two-day forecasts, the model is **recalibrated each day** using the latest 200 observed prices. Each forecast is evaluated on data after its calibration window.

The benchmark is a random walk with a sampled Gaussian price shock. Its shock scale is estimated from observed returns in the rolling window.

#### Average relative price errors

Lower is better. The following values are percentages, converted from Tables 6–7 on page 32.

| Horizon | Index | BH | ABH | Random walk |
|---|---|---:|---:|---:|
| 1 day | S&P 500 | 0.61% | **0.25%** | 0.63% |
| 1 day | Euro Stoxx 50 | 1.06% | **0.59%** | 1.12% |
| 1 day | Nikkei 225 | 0.54% | **0.42%** | 0.73% |
| 1 day | CSI 300 | 1.03% | **0.71%** | 1.04% |
| 2 days | S&P 500 | 0.89% | **0.78%** | 0.82% |
| 2 days | Euro Stoxx 50 | 1.55% | 1.55% | **1.53%** |
| 2 days | Nikkei 225 | 0.92% | **0.87%** | 0.98% |
| 2 days | CSI 300 | **1.46%** | 1.83% | 1.50% |

#### Directional accuracy

Tables 8–9 also report how often the forecast correctly predicts whether an index rises or falls. Higher is better.

| Horizon | Index | BH | ABH | Random walk |
|---|---|---:|---:|---:|
| 1 day | S&P 500 | 74.41% | **79.07%** | 53.48% |
| 1 day | Euro Stoxx 50 | 58.13% | **72.09%** | 44.18% |
| 1 day | Nikkei 225 | **65.11%** | 55.81% | 41.86% |
| 1 day | CSI 300 | 51.16% | **67.44%** | 41.86% |
| 2 days | S&P 500 | 62.79% | **69.76%** | 58.13% |
| 2 days | Euro Stoxx 50 | 51.16% | **53.48%** | 51.16% |
| 2 days | Nikkei 225 | 34.88% | **46.51%** | 41.86% |
| 2 days | CSI 300 | 27.91% | **48.83%** | 25.58% |

### Interpretation

ABH has the lowest one-day price error for all four indices. At two days, no model wins consistently. ABH predicts the two-day direction more often than the paper's random-walk benchmark in all four indices, although its accuracy is below 50% for the Nikkei 225 and CSI 300.

These are short-horizon, out-of-sample results **with daily recalibration**. They do not establish accurate long-run forecasts with fixed parameters. The random-walk benchmark also differs from a no-change price forecast because it adds a random shock.
