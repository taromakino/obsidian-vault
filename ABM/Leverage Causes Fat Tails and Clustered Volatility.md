**Authors:** Stefan Thurner, J. Doyne Farmer, and John Geanakoplos  
**Year:** 2012

## Research question

Can leverage limits turn small, random changes in asset demand into large price declines and heavy-tailed returns?

The paper studies a market with noise traders and value-investing funds. The notes focus on the leverage mechanism illustrated in Figure 3: funds can stabilize prices when they buy undervalued assets, but borrowing can force them to sell during price declines, amplifying losses.

## 1. Market setup

The market has one risky asset and cash. There are $N$ shares of the asset, its constant fundamental value is $V$ per share, and its market price at time $t$ is $p(t)$. Cash pays no interest.

Two types of traders participate:

- **Noise traders** hold a randomly changing dollar amount of the asset.
- **Funds** buy when the asset is below fundamental value. They may borrow to enlarge their holdings, but cannot short the asset.

For fund $h$, $D_h(t)$ denotes shares held, $C_h(t)$ cash (negative when the fund borrows), and $W_h(t)$ wealth. The fund also has a performance score and an active or inactive status.

## 2. Noise-trader demand

At each time $t$, draw a new standard normal shock $\chi(t)$. Noise traders' dollar demand follows
$$
\log\xi_{\mathrm{nt}}(t)=\rho\log\xi_{\mathrm{nt}}(t-1)+\sigma\chi(t)+(1-\rho)\log(VN).
$$
The parameter $\sigma$ controls the size of random changes, while $\rho$ controls their persistence. Dollar demand tends to return toward $VN$, the cost of all $N$ shares at fundamental value.

Given a trial market price $p$, noise traders demand
$$
D_{\mathrm{nt}}(t;p)=\frac{\xi_{\mathrm{nt}}(t)}{p}
$$
shares. The shock changes their dollar demand; the trial price converts it to shares.

## 3. Fund wealth, performance, and investor flows

The market price at time $t$ is not known in advance. The model tests a candidate price $p$ and evaluates all fund quantities at that price. The notation $W_h(t;p)$ means the wealth fund $h$ would have if the current price were $p$.

First, the fund's previous shares are revalued at the trial price. Its wealth before investor deposits or withdrawals is
$$
A_h(t;p)=D_h(t-1)p+C_h(t-1).
$$
The return on the fund's previous position is
$$
r_h(t;p)=\frac{D_h(t-1)[p-p(t-1)]}{W_h(t-1)}.
$$
This return is larger in magnitude for a fund holding more shares relative to its wealth.

The fund's performance score is an exponential moving average of returns:
$$
r_h^{\mathrm{perf}}(t;p)=(1-a)r_h^{\mathrm{perf}}(t-1)+a r_h(t;p).
$$
The parameter $a$ sets the weight on the newest return.

Investors add money to funds whose performance exceeds a benchmark $r^{\mathrm b}$ and withdraw money from funds that underperform it. The desired flow is
$$
\widetilde F_h(t;p)=b[r_h^{\mathrm{perf}}(t;p)-r^{\mathrm b}]A_h(t;p).
$$
The parameter $b$ controls how strongly flows respond to performance. Actual withdrawals cannot exceed the fund's available wealth:
$$
F_h(t;p)=\max\{\widetilde F_h(t;p),-A_h(t;p)\}.
$$
The fund's candidate wealth after this flow is
$$
W_h(t;p)=A_h(t;p)+F_h(t;p).
$$
## 4. Fund demand and leverage limits

Mispricing at the trial price is
$$
m(t;p)=V-p.
$$
Fund $h$ targets the following leverage:
$$
\lambda_h(t;p)=\begin{cases}
0, & m(t;p)\leq0,\\
\min\{\beta_hm(t;p),\lambda_{\mathrm{MAX}}\}, & m(t;p)>0.
\end{cases}
$$
When the asset is at or above fundamental value, the fund does not hold it. When it is underpriced, desired leverage rises with the size of the discount, at a rate controlled by $\beta_h$, but cannot exceed $\lambda_{\mathrm{MAX}}$.

An active, solvent fund demands
$$
D_h(t;p)=\frac{\lambda_h(t;p)W_h(t;p)}{p}.
$$
The numerator is the target dollar position; dividing by $p$ converts it to shares. Closed funds and funds below the survival threshold have zero demand.

Leverage is the value of shares held divided by fund wealth. A limit of $\lambda_{\mathrm{MAX}}=1$ permits an unleveraged position, while $\lambda_{\mathrm{MAX}}=10$ permits assets worth up to ten times fund wealth.

## 5. Market clearing

For a trial price $p$, the total excess demand for shares is
$$
Z_t(p)=D_{\mathrm{nt}}(t;p)+\sum_h D_h(t;p)-N.
$$
The numerical solver changes $p$ until it finds a price satisfying
$$
Z_t(p)=0.
$$
That solution is the market price $p(t)$. Noise traders and all funds together then demand exactly the fixed supply $N$.

All quantities evaluated at $p(t)$ become the period's realized returns, performance scores, investor flows, wealth, and positions. Each fund's cash balance is
$$
C_h(t)=W_h(t)-p(t)D_h(t).
$$
A positive balance is cash; a negative balance represents borrowing of $-C_h(t)$.

## 6. A single iteration of the model

At the start of time $t$, the model knows $p(t-1)$, noise traders' previous dollar demand, and each fund's previous shares, cash, wealth, performance, and active status. The parameters are fixed. Only the new noise-trader shock is random.

**Step 1. Update noise-trader dollar demand.** Draw $\chi(t)$ and calculate $\xi_{\mathrm{nt}}(t)$ from the equation in Section 2.

**Step 2. Try a candidate price.** For each trial price $p$, use the previously held positions and cash to calculate marked wealth $A_h(t;p)$, fund return $r_h(t;p)$, and updated performance $r_h^{\mathrm{perf}}(t;p)$.

**Step 3. Calculate flows and fund wealth.** Determine $\widetilde F_h(t;p)$, cap withdrawals to obtain $F_h(t;p)$, and calculate $W_h(t;p)$.

**Step 4. Calculate demand.** Given the same trial price, compute each fund's mispricing, target leverage, and share demand $D_h(t;p)$. Also calculate noise-trader share demand $D_{\mathrm{nt}}(t;p)$.

**Step 5. Solve for the price.** Evaluate $Z_t(p)$ and repeat Steps 2–4 with another trial price until $Z_t(p)=0$. All prices and fund demands must be consistent simultaneously; the intermediate trial values are not separate trading periods.

**Step 6. Finalize the state.** At the solved price $p(t)$, record the new fund holdings, wealth, returns, performance, flows, and cash balances. Apply the fund closure and re-entry rules. These values become the inputs for time $t+1$.

If a fund's wealth becomes negative, it defaults because it cannot repay its debt. A fund is also closed when wealth falls below $W_h(0)/10$. It remains inactive for $T_{\mathrm{reintro}}$ periods, then returns with its original strategy and initial wealth $W_h(0)$.

## 7. Initialization and parameters

The baseline values in the supplied summary are:

| Quantity | Value | Meaning |
|---|---:|---|
| $V$ | $1$ | Fundamental value per share |
| $N$ | $1000$ | Fixed supply of shares |
| $\rho$ | $0.99$ | Persistence of noise-trader dollar demand |
| $\sigma$ | $0.035$ | Size of demand shocks |
| $W_h(0)$ | $2$ | Initial wealth per fund |
| $\beta_h$ | $5,10,\ldots,50$ | Fund-specific response to mispricing |
| $a$ | $0.1$ | Weight on latest fund return |
| $b$ | $0.15$ | Investor flow sensitivity |
| $r^{\mathrm b}$ | $0.005$ | Performance benchmark |
| $T_{\mathrm{reintro}}$ | $100$ | Periods before a closed fund returns |

The initial state used in the summary is
$$
\begin{aligned}
\xi_{\mathrm{nt}}(0)&=VN, & p(0)&=V,\\
W_h(0)&=2, & D_h(0)&=0,\\
C_h(0)&=W_h(0), & r_h^{\mathrm{perf}}(0)&=0.
\end{aligned}
$$
Thus, noise-trader demand initially supports the fundamental price, and each fund starts with cash but no shares, debt, or performance history. The paper does not specify every initial condition; early periods are excluded from measurements to reduce dependence on those choices.

## 8. Leverage experiment

The main comparison holds the other parameters fixed and varies $\lambda_{\mathrm{MAX}}$. Figure 3 compares:

| Market | Leverage limit | Interpretation |
|---|---:|---|
| Noise traders only | Not applicable | No stabilizing fund demand |
| Funds without borrowing | $\lambda_{\mathrm{MAX}}=1$ | Funds can hold assets up to their wealth |
| Leveraged funds | $\lambda_{\mathrm{MAX}}=10$ | Funds can hold assets worth ten times their wealth |

Intermediate leverage limits are also examined.

Without borrowing, funds tend to stabilize prices: when noise-trader demand falls and the asset becomes cheaper, funds want to buy more of it. Their demand offsets part of the initial price decline.

With borrowing, funds can initially buy larger positions. This also increases their exposure to price changes. If a fund has leverage $\lambda$, a price fall of fraction $x$ reduces its wealth by approximately fraction $\lambda x$, before accounting for other flows or rebalancing.

As prices fall, the value of shares declines while the debt does not. Leverage therefore rises. To respect the cap, the fund may have to reduce its position to at most
$$
\lambda_{\mathrm{MAX}}W_h(t)
$$
dollars of risky assets. If this is less than its existing position, it must sell shares and use the proceeds to reduce debt.

That sale adds selling pressure during an existing decline. The lower equilibrium price can cause more wealth losses and more forced sales, especially when several funds hit their limits together. A modest noise-trader shock can therefore produce a large negative return.

## 9. Main interpretation

**Leverage creates an asymmetric feedback mechanism.** Value investors can dampen ordinary price fluctuations by buying undervalued assets. But once they are highly leveraged, falling prices shrink their wealth and can force them to sell the same asset. This reinforces declines and produces a heavier negative tail in the return distribution.

The crucial feature is simultaneous market clearing: forced fund sales and noise-trader demand determine one equilibrium price, rather than unfolding as an exogenous sequence of trades.
