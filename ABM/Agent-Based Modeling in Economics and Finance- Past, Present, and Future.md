**Authors:** Robert L. Axtell and J. Doyne Farmer  
**Year:** 2025

## Financial agent-based models

### 1.1 Santa Fe Stock Market — Arthur et al. (1997)

**Question:** Can interactions among investors produce clustered volatility even without corresponding bursts of external news?

**Agents and setting:** Investors choose between a stock with changing dividends and a bond with a fixed interest rate. Each investor uses a set of forecasting rules, called a *classifier system*. Rules can use dividend information, recent prices, or both. Rules that perform well are retained and adapted; unsuccessful ones are discarded.

**Mechanism:** Investors learn which strategies work in different market conditions. Some become trend followers, while others focus on fundamentals. As the profitability of these strategies changes, traders and capital shift between them. When trend following becomes more important, it destabilizes prices and increases volatility. Volatility clustering therefore emerges from changes in the mix of trading strategies.

**Evaluation:** The researchers compare adaptive investors with investors given correct beliefs about the dividend process. With correct beliefs, prices stay close to fundamental value and mostly follow dividend shocks. With adaptive learning, prices exhibit clustered volatility even though returns have little autocorrelation. The model reproduces these market patterns qualitatively.

*Survey: Section II.B.1, pp. 23–24.*

### 1.2 Heterogeneous expectations — Brock and Hommes (1997, 1998); Franke and Westerhoff (2012)

**Question:** How can differences in investors' expectations generate realistic financial-market fluctuations?

**Agents and mechanism:** Investors use different approaches to forming expectations, broadly including fundamental-based and trend-following strategies. As market conditions change, the relative importance of these strategies changes too. Brock and Hommes obtained results similar to the Santa Fe model using a simpler setting.

**Evaluation:** Later models in this tradition were fitted quantitatively to market data. The survey highlights three relevant statistical properties: autocorrelation of returns, autocorrelation of volatility, and the distribution of price changes. It reports that the Franke–Westerhoff model can be estimated from empirical data and that Barde (2016) found a substantially better fit than standard GARCH volatility models.

*Survey: Section II.B.1, pp. 23–24.*

### 1.3 Leverage-driven volatility — Thurner, Farmer and Geanakoplos (2012); Poledna et al. (2014)

**Question:** Can leverage generate clustered volatility and extreme price movements even when investors trade on fundamentals?

**Agents and mechanism:** The model studies fundamentalist investors, comparing markets where leverage is prohibited with markets where borrowing is allowed. Without leverage, the model does not produce clustered volatility. Once leverage is permitted, clustered volatility emerges. Borrowing makes positions more sensitive to price movements, allowing financial stress and trading responses to amplify market fluctuations.

**Evaluation:** At realistic levels of leverage, the model provides a good match to the tails of observed price-change distributions. Poledna et al. extend the model and find that it matches the decay of volatility after large spikes better than standard GARCH models. They also compare leverage regulations. At high leverage, fixed limits perform better than limits that change with market conditions, such as Value-at-Risk-based rules.

*Survey: Section II.B.1, p. 24.*

## Generalization to unseen data

The survey distinguishes between reproducing market patterns and predicting data or outcomes beyond those used to calibrate a model. Financial ABMs have progressed from qualitative explanations toward models that quantitatively match observed market statistics and can be used for volatility forecasting.

The Franke–Westerhoff and leverage-based models are particularly relevant because the survey reports comparisons with conventional GARCH models. These comparisons show promising empirical performance, while the survey treats reliable forecasting as an ongoing challenge rather than a settled achievement.

### Main challenges

**Estimating agent behavior:** Model parameters can be selected by searching for values that make simulated outcomes resemble observed data. This becomes harder as the number of parameters grows. Individual-level data and experiments can help ground behavioral rules.

**Initializing a forecast:** An ABM needs a starting state for each agent, not just aggregate values such as prices or market volatility. Many possible agent configurations can fit the same aggregate data. If the initial states are inconsistent with the model's dynamics, the simulation can produce misleading short-run behavior.

**Updating the model:** As model forecasts diverge from observed data, individual-agent states need to be initialized again. The survey identifies this repeated initialization problem as an important obstacle to time-series forecasting.

**Sensitivity to model design:** Agents' behavioral rules, interactions, and the values of model parameters can affect emergent outcomes. This makes empirical grounding and careful evaluation especially important.

*Survey: Sections III.H (pp. 43–44), IV.G (p. 47), and IV.H (pp. 47–48).*

## Takeaway

These three financial ABM families explain market instability through adaptive trading strategies, heterogeneous expectations, and leverage. Their strongest results are explanations of volatility clustering and quantitative matches to market statistics. The survey presents empirical forecasting as promising, but emphasizes the difficulty of reliably preparing and updating agent-level states for future predictions.
