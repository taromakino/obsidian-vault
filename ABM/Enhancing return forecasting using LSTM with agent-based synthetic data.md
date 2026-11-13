**Authors:** Lijian Wei, Sihang Chen, Junqin Lin, Lei Shi\
**Year:** 2025

# Market setup and agent arrivals

Consider a historical daily price series indexed by $t=1,\ldots,T$. Its five-day moving average is the fundamental value $p_t^f$.

The artificial market contains $N=1{,}000$ agents indexed by $i$. Their arrivals follow independent Poisson processes. Agent $i$ has mean interarrival time $\tau_i=\tau/(1+\omega_i)$, where $\tau=240$ and $\omega_i$ is its chartist-belief weight. The arrival rate is $\lambda_i=1/\tau_i$.

Taking each simulation period as one day, setting $\omega_i=0$ solely to illustrate arrival frequency gives $0.00417$ expected arrivals per agent per day, or $4.17$ across all agents. If each arrival generates one order, this is also the expected number of daily order submissions.

An arriving agent submits an order at price $p_t^i$ and signed size $z_t^i$: positive for buys and negative for sells. Submitted orders interact through a continuous double-auction limit order book to determine the synthetic price $p_t$. If no trade occurs, $p_t$ is the midpoint of the best bid $bq_t$ and ask $aq_t$.

# Price forecast

Agent $i$ forecasts the log price at its investment horizon:

```math
\mathbb E_t^i[\log p_{t+\tau_i}\mid\mathcal F_t]=\log p_t^f+\widetilde\theta_t^i+\omega_i(\log p_t-\log p_{t-\tau_i}).
```

Here $\mathcal F_t$ denotes available market information. The first term is the fundamental log value, the second is a uniformly distributed belief error over $[-\sigma_v\sqrt{\tau_i},\sigma_v\sqrt{\tau_i}]$, and the third is the chartist component.

# Return variance

Agent $i$ estimates risk by combining two variance components through a weighted harmonic mean. The fundamental component is $\sigma_v^2\tau_i$, where $\sigma_v$ is estimated from the historical training data and scaled to the $\tau_i$-period horizon. The chartist component is the trailing sample variance of the simulated log prices $\log p\_{t-1},\ldots,\log p\_{t-\tau_i}$, using a lookback of $\tau_i$ periods. The harmonic-mean weights are $1-\zeta_i$ and $\zeta_i$, with $\zeta_i=\omega_i/(1+\omega_i)$. The resulting perceived variance is $(\sigma_t^i)^2$.

# Order price

The agent samples a nonnegative profit target $\epsilon_t^i$ uniformly from $[0,\alpha_i(\sigma_t^i)^2]$, where $\alpha_i=\alpha/(1+\omega_i)$, and sets its order price to

```math
p_t^i=\exp\!\left(\mathbb E_t^i[\log p_{t+\tau_i}\mid\mathcal F_t]-\epsilon_t^i\right).
```

# Optimal portfolio and order size

Let $W_t^i$ be current wealth, $\pi_t^i$ the share of wealth invested in the risky asset, and $R\_{t,t+\tau_i}$ the return over the investment horizon. The agent maximizes expected constant absolute risk aversion (CARA) utility:

```math
\max_{\pi_t^i}\ \mathbb E_t^i\!\left[-\exp\!\left(-\alpha_i[W_t^i+W_t^i\pi_t^i R_{t,t+\tau_i}]\right)\right].
```

Assume the future return is normally distributed, with expected return $\widehat r\_{t,t+\tau_i}^i=\mathbb E_t^i[\log p\_{t+\tau_i}\mid\mathcal F_t]-\log p_t^i$ and variance $(\sigma_t^i)^2$, using the paper's approximation of simple returns by log returns. Writing $q_t^i=W_t^i\pi_t^i$ for the dollar allocation, the Gaussian moment-generating function reduces the objective to maximizing its certainty equivalent:

```math
q_t^i\widehat r_{t,t+\tau_i}^i-\frac{\alpha_i}{2}(q_t^i)^2(\sigma_t^i)^2.
```

The first-order condition gives the optimal dollar allocation:

```math
W_t^i\pi_t^{i,*}=\frac{\widehat r_{t,t+\tau_i}^i}{\alpha_i(\sigma_t^i)^2}.
```

Dividing by the submission price converts the allocation to the target number of shares. Subtracting the existing holding $s_t^i$ gives the signed order size:

```math
z_t^i=\frac{W_t^i\pi_t^{i,*}}{p_t^i}-s_t^i=\frac{\widehat r_{t,t+\tau_i}^i}{p_t^i\alpha_i(\sigma_t^i)^2}-s_t^i.
```
