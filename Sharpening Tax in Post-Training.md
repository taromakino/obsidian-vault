**Authors:** Changdae Oh, Qi Zeng, Qi Qi, Andrey Zhmoginov, Deren Lei, Yun He, Hoang Phan, Hangoo Kang, Azalia Mirhoseini, Sharon Li  
**Year:** 2026  

## The question

Does post-training create new abilities, or mainly make a model use a narrower set of abilities it already had?

Pre-training produces many possible behaviors. Post-training rewards some behaviors and makes them more likely. This is often beneficial, but successful yet uncommon strategies may become harder to sample. The paper tests whether this happens even for interactive AI agents that must use tools over multiple turns.

## pass@1 versus pass@K

- **pass@1:** The chance that one attempt succeeds. This reflects first-try accuracy or reliability.
- **pass@K:** The chance that at least one of $K$ attempts succeeds. This reflects **coverage under a fixed attempt budget**.

For a task with an independent per-attempt success probability $p$:

$$
\operatorname{pass@K}=1-(1-p)^K.
$$

If post-training drives $p$ close to zero on a task, additional attempts help much less.

This gives two different senses of being “better”: succeeding **more often immediately** and being able to **eventually find a solution**.

## What the experiments found

The authors compare **14 base/post-trained model pairs** across four families (Gemma-4, Ministral-3, Qwen2.5, and Qwen3.5) on three interactive benchmarks: BFCL, WebShop, and ACEBench.

- Post-trained models generally perform better on **pass@1**.
- Base models, when given a lightweight agent harness and many attempts, often catch up to or exceed post-trained models on **pass@K**.
- For example, on WebShop with Gemma-4 31B, the paper reports **over 85% pass@128 for the harness-equipped base model versus about 56% for its post-trained counterpart**.
- The post-trained models' task-level success probabilities often become more **bimodal**: a task is either solved almost every time or almost never solved.

The harness matters: it helps base models format actions and use tools. The comparison is not between entirely unassisted base models and post-trained models.

## Intuition: distribution sharpening and bimodalization

**Distribution sharpening** means post-training concentrates the model's probability on a smaller set of preferred behaviors. RL rewards successful trajectories, making them more likely to be sampled. This often improves first-try accuracy, but can also make unusual yet useful strategies rare.

The authors observe **bimodalization** in the *distribution of success probabilities across tasks*: after post-training, more tasks cluster near either a **0%** or a **100%** chance of success on one attempt, rather than having moderate probabilities in between.

**Why this matters:** moving probabilities toward 100% helps **pass@1**, but moving some toward 0% can severely hurt **pass@K**, because even many repeated attempts will rarely discover those solutions. Bimodalization is about *which tasks are accessible*, not merely whether the model's individual responses sound more certain.

This is the central reason a post-trained model can look much stronger on a single attempt while exhibiting worse coverage under a large sampling budget.

## Sharpening Tax: the metric

Let $K$ be the largest number of attempts considered.

### Raw area: $A(K)$

$$
A(K)=\sum_{k=1}^{K-1}\left[\operatorname{pass@K}-\operatorname{pass@k}\right].
$$

Visualize the **pass@k curve**. Draw a horizontal line at its final height, $\operatorname{pass@K}$. The sum of the gaps between that line and the earlier points on the curve is $A(K)$.

**It is an area above the curve and below its final ceiling**, not the usual area under the curve. Strictly speaking, it is a discrete sum.

- Large $A(K)$: relatively more of the model's success arrives through additional attempts.
- Small $A(K)$: the curve saturates early, **or** the model never recovers many failures.

A probabilistic interpretation makes this especially concrete. Let $H$ be the attempt number of the **first success**, and set $H=\infty$ if there is no success. Then:

$$
A(K)=\mathbb E\left[(H-1)\mathbf 1(H\le K)\right].
$$

Success on the first attempt contributes nothing to this quantity, while successful retries contribute more the later the first success occurs. Tasks never solved within the budget also contribute nothing. Thus $A(K)$ measures *retry-derived success*, with later successes weighted more heavily.

### Normalized area: $S(K)$

$$
S(K)=\frac{A(K)}{(K-1)(1-\operatorname{pass@1})}.
$$

Why this denominator? Every individual gap satisfies:

$$
\operatorname{pass@K}-\operatorname{pass@k}\le 1-\operatorname{pass@1}.
$$

There are $K-1$ gaps, so:

$$
A(K)\le (K-1)(1-\operatorname{pass@1}).
$$

The denominator is therefore the area of a rectangle with:

- **Width:** $K-1$ attempted-budget intervals.
- **Height:** $1-\operatorname{pass@1}$, the initial failure rate (the room to improve).

This gives a bounded measure $0\le S(K)\le1$ when pass@1 is below 1. It also helps compare models with very different initial accuracies.

**Important nuance:** This is a useful mathematical bound, not necessarily the tightest achievable bound for curves generated by independent attempts. A larger $S(K)$ does **not** unconditionally mean a better model: successful retries that occur *later* contribute more.

### Sharpening Tax itself

$$
\operatorname{Tax}_S(K)=S_{\text{base}}(K)-S_{\text{post}}(K).
$$

A **positive** tax means the base model receives more normalized value from repeated attempts than the post-trained model. That may reflect **lost coverage**, **faster success/saturation**, or both. So the tax is a diagnostic of *test-time scaling behavior*, not a standalone quality score.

## The proposed fix: PTGS

**Posterior-Tempered Group Sampling (PTGS)** changes the sampling temperature during RL training based on how difficult the current prompt seems.

- For **hard prompts**, use a higher temperature to try more varied behaviors and find rare successes.
- For **easy prompts**, use a lower temperature to reinforce reliable behaviors.
- Estimate difficulty from previous successful and failed rollouts using a **Beta distribution**; discount older observations so estimates can change as the model learns.

The motivation is especially clear for group-based RL: if all attempts on a hard task fail, training may get little information about what works. More exploration can uncover successful trajectories and provide a useful learning signal.

The authors test PTGS with PPO and GRPO in Sokoban and FrozenLake. Compared with fixed-temperature RL baselines, PTGS generally improves both single-attempt accuracy and coverage from repeated attempts in those experiments. It **reduces**, but does not necessarily eliminate, the trade-off.

## Limitations and cautions

1. **Causality is not fully isolated.** The main checkpoint comparisons use existing base and post-trained models with partly undisclosed training recipes. Changes cannot all be attributed specifically to RL rather than other post-training steps.
2. **The inference harness matters.** Base-model agent performance depends in part on extra scaffolding, so results are not claims about raw, unassisted base-model behavior.
3. **Coverage is budget-dependent.** High pass@K matters most when many attempts are affordable and outcomes can be evaluated reliably.
4. **The score has a timing confound.** A lower $S(K)$ can mean worse eventual recovery *or* faster recovery; those have different practical implications.
5. **The PTGS demonstrations are limited.** Results on Sokoban and FrozenLake are promising, but do not establish effectiveness for all large-scale agent or reasoning training runs.
