
**Authors:** Stefan Thurner, J. Doyne Farmer, John Geanakoplos  
**Published:** 2012

# Noise traders dollar demand

The dollar amount noise traders want to invest follows:
$$
\log \xi_{nt}(t)=\rho\log\xi_{nt}(t-1)+\sigma\chi(t)+(1-\rho)\log(VN)
$$
Here $\xi_{nt}(t)$ is the dollar amount noise traders want to invest in the asset in period $t$. Its logarithm depends on last period's dollar demand, $\xi_{nt}(t-1)$, with persistence controlled by $\rho$. A random shock $\chi(t)$, drawn from a standard normal distribution (mean 0, standard deviation 1), changes demand; $\sigma$ controls the size of that shock. The final term pulls log demand toward $\log(VN)$, where $V$ is the perceived fundamental value per asset unit and $N$ is the fixed number of units available. When $\rho<1$, this gives demand a tendency to return toward $VN$.

The paper uses $\rho=0.99$, $\sigma=0.035$, $V=1$, and $N=1000$.

The demand measured in asset units is:
$$
D_{nt}(t,p(t))=\frac{\xi_{nt}(t)}{p(t)}
$$
Here $p(t)$ is the price per unit at time $t$ and $D_{nt}$ is the number of units demanded, not the dollar amount.

# Funds

The model also has funds, indexed by $h$, that buy the asset when it is underpriced. Their dollar demand is $D_h(t)p(t)$, where $D_h(t)$ is the number of asset units fund $h$ wants to hold. This is analogous to the noise traders' dollar demand $\xi_{nt}(t)$.

Funds respond to the mispricing $m(t)=V-p(t)$, the difference between fundamental value and market price. When $m(t)>0$, their dollar demand increases linearly with mispricing, according to $D_h(t)p(t)=\beta_h m(t)W_h(t)$, where $\beta_h$ measures how aggressively fund $h$ responds. They hold no asset when it is overpriced ($m(t)<0$), and their demand stops increasing once they reach their leverage limit.

Fund $h$ has wealth $W_h(t)$, equal to the value of its asset holdings plus its cash position $C_h(t)$:
$$
W_h(t)=D_h(t)p(t)+C_h(t)
$$
Cash can be negative when the fund borrows to buy the asset. Every fund starts with wealth $W_h(0)=2$.

The fund's demand for asset units is given by three cases (when its wealth is nonnegative):
$$
D_h(t,p(t))=\begin{cases}
0, & m(t)\leq 0,\\[4pt]
\dfrac{\beta_h m(t)W_h(t)}{p(t)}, & 0<m(t)<m_h^{\mathrm{crit}},\\[8pt]
\dfrac{\lambda_{\mathrm{MAX}}W_h(t)}{p(t)}, & m(t)\geq m_h^{\mathrm{crit}}.
\end{cases}
$$
Here $\lambda_{\mathrm{MAX}}$ is the maximum leverage. Before this limit is reached, fund $h$ has leverage $\lambda_h(t)=D_h(t)p(t)/W_h(t)=\beta_h m(t)$. The critical mispricing $m_h^{\mathrm{crit}}$ is defined as the value of mispricing at which leverage reaches its maximum. Thus $\beta_h m_h^{\mathrm{crit}}=\lambda_{\mathrm{MAX}}$, so $m_h^{\mathrm{crit}}=\lambda_{\mathrm{MAX}}/\beta_h$.

The fund's leverage is the dollar value of its asset holdings divided by its wealth (equation 4):
$$
\lambda_h(t)=\frac{D_h(t)p(t)}{W_h(t)}=\frac{D_h(t)p(t)}{D_h(t)p(t)+C_h(t)}.
$$
The firm is required by the bank it borrows from to maintain $\lambda_h(t)\leq\lambda_{\mathrm{MAX}}$. If leverage would exceed this limit, the fund must reduce the dollar value of its holdings, $D_h(t)p(t)$, by selling some of the asset. This is a margin call.

If $W_h(t)<0$, the fund defaults and goes out of business. It sells its assets to repay as much of its debt as possible, and the bank bears the remaining loss. For simplicity, the model assumes the bank continues lending to other funds as before, despite the loss. After some time, the defaulting fund reemerges as a new fund.

The market price is set by equating the combined demand of noise traders and funds to the fixed supply $N$ of the asset:
$$
D_{nt}(t,p(t))+\sum_h D_h(t,p(t))=N.
$$
# Fund wealth dynamics

If a fund's wealth falls below $W_h(0)/10$, it goes out of business. After $T_{\mathrm{reintro}}=100$ time steps, it is replaced by a new fund with initial wealth $W_h(0)$ and the same parameters $\beta_h$ and $\lambda_{\mathrm{MAX}}$.

A single representative investor deposits money into or withdraws money from each fund based on its recent performance. Fund $h$'s return from price changes at time $t$ is
$$
r_h(t)=\frac{D_h(t-1)[p(t)-p(t-1)]}{W_h(t-1)}.
$$
Recent performance is measured by an exponential moving average (EMA) of returns (equation 5):
$$
r_h^{\mathrm{perf}}(t)=(1-a)r_h^{\mathrm{perf}}(t-1)+a r_h(t).
$$
The investor's desired capital flow into fund $h$ is (equation 6):
$$
\widetilde F_h(t)=b\left[r_h^{\mathrm{perf}}(t)-r_b\right]\left[D_h(t-1)p(t)+C_h(t-1)\right].
$$
This depends on recent performance relative to the benchmark return $r_b$, the fund's size before investor flows, and the investor's sensitivity to performance, $b$. Positive flows are deposits; negative flows are withdrawals.

The actual flow is limited by (equation 7):
$$
F_h(t)=\max\left(\widetilde F_h(t),-\left[D_h(t-1)p(t)+C_h(t-1)\right]\right).
$$
This ensures the investor cannot withdraw more than the fund's current value before investor flows.

Fund wealth then changes through gains or losses on its existing asset holdings and through investor deposits or withdrawals (equation 8):
$$
W_h(t)=W_h(t-1)+[p(t)-p(t-1)]D_h(t-1)+F_h(t).
$$
In the simulations, the paper uses $a=0.1$ for the EMA, $b=0.15$ for investor sensitivity to performance, and benchmark return $r^b=0.005$.

The benchmark return $r^b$ determines how large funds become relative to noise traders. When it is very low, funds attract substantial wealth and buy aggressively even at small mispricings, creating an effective floor under prices. When it is very high, funds remain small and have little effect on prices. The model's most interesting behavior occurs at intermediate benchmark returns, when fund demand and noise-trader demand are comparable.
