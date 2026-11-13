**Authors:** Mert Yuksekgonul, Daniel Koceja, Xinhao Li, Federico Bianchi, Jed McCaleb, Xiaolong Wang, Jan Kautz, Yejin Choi, James Zou, Carlos Guestrin, Yu Sun
**Year:** 2026
# Preliminaries

## Discovery problem

A discovery problem begins with a text description d, which is always given to the policy. A state s is a candidate solution, such as a GPU kernel implementation. The reward R(s) is a real number that measures the solution's quality, with higher values being better. The best-known existing solution is s_sota, with reward r_sota = R(s_sota). If no solution exists yet, the starting solution can be empty. A discovery occurs when a new solution has R(s) > r_sota. Its significance is measured by the improvement R(s) − r_sota.

## Actions and transitions
$$
a \sim \pi_\theta(\cdot\mid d,s), \qquad (s,a)\to s'.
$$
The policy samples an action a given the problem d and current solution s. In coding tasks, a contains code and may contain a reasoning trace. The transition to s' simply parses the code from a. In the paper's mathematics tasks, it also executes that code.

## Best-of-N
$$
s=s_{\mathrm{sota}}\ \text{or}\ \langle\mathrm{empty}\rangle,\qquad a_i\sim\pi_\theta(\cdot\mid d,s),\quad i=1,\ldots,N.
$$
Sample N independent solutions (rollouts) from the same fixed starting state. The index i, rather than a time index, emphasizes that the samples are independent. Starting from the empty solution encourages exploration; starting from s_sota may favor improving an existing result.

## State reuse
$$
s_i\sim\operatorname{reuse}(H_i),\qquad a_i\sim\pi_\theta(\cdot\mid d,s_i),\qquad H_{i+1}=H_i\cup\{(s'_i,r_i)\},\quad r_i=R(s'_i).
$$
Here H_i is a buffer of previous solutions and their rewards. The reuse heuristic selects a previous solution as the next starting point. It favors high-reward states but still sometimes selects lower-reward ones. This balances exploitation and exploration. Reusing a state lets search continue from previous progress.

## State-action reuse
$$
(s_i,c_i)\sim\operatorname{reuse}(H_i),\qquad a_i\sim\pi_\theta(\cdot\mid d,s_i,c_i),\qquad H_{i+1}=H_i\cup\{(s_i,a_i,s'_i,r_i)\}.
$$
This also reuses previous actions. Actions may contain reasoning traces and intermediate results that are not part of the final solution state. The reuse heuristic turns useful information from those actions into natural-language context c_i—effectively memory—for the next policy call.

# 3.1 Naive RL at Test Time

The naive RL baseline trains the policy on the test problem. At each training step, it starts from the empty state, samples a batch of actions (candidate solutions), evaluates their rewards, estimates the gradient of expected reward, and updates the policy parameters. It then repeats with the updated policy, again starting from the empty state.
$$
J(\theta)=\mathbb{E}_{a\sim\pi_\theta(\cdot\mid s)}[R(s,a)],\qquad s=\langle\mathrm{empty}\rangle
$$
$$
\theta_{i+1}=\theta_i+\eta\nabla_\theta J(\theta_i).
$$
In standard RL, the **policy is the final artifact**. The goal is good expected reward, or average performance. In discovery, the **single best solution is the final artifact**. The policy is only a means to find it. Average performance does not matter if the search finds one solution that improves on the state of the art.

This creates three limitations for naive RL. First, optimizing expected reward does not directly prioritize rare, exceptional solutions. Second, always starting from the empty state prevents search from continuing directly from promising earlier solutions, giving each attempt a short effective horizon. Third, expected-reward training may favor safe actions over risky attempts that could lead to a breakthrough. At the reuse level, naive prioritization can over-exploit a few promising states at the expense of diversity.

# 3.2 TTT-Discover

## Entropic objective

The entropic objective gives more importance to high-reward actions:
$$
J_\beta(\theta)=\mathbb{E}_{s\sim\operatorname{reuse}(H)}
\left[\log\mathbb{E}_{a\sim\pi_\theta(\cdot\mid s)}
\left[e^{\beta(s)R(s,a)}\right]\right].
$$
The exponential amplifies reward differences, so an exceptional solution matters much more than an ordinary one. The parameter $\beta$ controls this effect: small values behave more like expected reward, while large values focus on the best outcomes. The log brings the expression back to a manageable scale.

For large $\beta$, the largest reward dominates the sum of exponentials:
$$
\log\sum_i p_i e^{\beta R_i}\approx\beta\max_{i:p_i>0}R_i.
$$
Therefore,
$$
\lim_{\beta\to\infty}\frac{1}{\beta}
\log\sum_i p_i e^{\beta R_i}
=\max_{i:p_i>0}R_i.
$$
Strictly, the expression divided by $\beta$ approaches the maximum. The paper omits the division by $\beta$; for a fixed positive $\beta$, this scaling does not change which policy maximizes the objective. In the full objective, $\beta(s)$ may differ by starting state.

## Gradient and advantage

The policy gradient has the usual score-function form, but each action is weighted by its normalized exponential reward:
$$
\nabla_\theta J_\beta(\theta)=\mathbb{E}_{s\sim\operatorname{reuse}(H),\,a\sim\pi_\theta(\cdot\mid s)}\left[w_{\beta(s)}(a)\nabla_\theta\log\pi_\theta(a\mid s)\right].
$$
$$
w_{\beta(s)}(a)=\frac{e^{\beta(s)R(s,a)}}{\mathbb{E}_{a'\sim\pi_\theta(\cdot\mid s)}[e^{\beta(s)R(s,a')} ]}.
$$
For a fixed starting state s, sample G independent actions from the policy. In practice, the denominator is estimated by the sample mean of their exponentiated rewards:
$$
\widehat w_{\beta(s)}(a_i)=\frac{G e^{\beta(s)R(s,a_i)}}{\sum_{j=1}^{G}e^{\beta(s)R(s,a_j)}}.
$$
The weights have mean 1 within the group, and high-reward actions receive larger weights. This gives the advantage used for training:
$$
A(a;s)=\underbrace{w_{\beta(s)}(a)-1}_{\text{entropic advantage}}-\underbrace{\lambda\log\frac{\pi_\theta(a\mid s)}{\pi_{\theta_0}(a\mid s)}}_{\text{KL penalty}}.
$$
Subtracting 1 is a baseline because $\mathbb{E}_{a\sim\pi_\theta(\cdot\mid s)}[w_{\beta(s)}(a)]=1$; it leaves the expected policy gradient unchanged and can reduce variance. The KL term discourages the policy from moving too far from the reference policy π_θ₀. This is a KL-regularized policy-gradient advantage, similar in form to REINFORCE with a baseline; the advantage alone does not determine whether the update uses PPO, GRPO, or another optimizer.

## Adaptive beta

A large β early in training can make learning unstable, while a small β later can make the advantage too weak. The paper chooses β(s) separately for each starting state by constraining the KL divergence of the reward-reweighted policy from the current policy. This adapts how strongly the objective favors the best actions.

## PUCT state reuse

TTT-Discover chooses which previous solution to improve using a PUCT-inspired score:
$$
\operatorname{score}(s)=Q(s)+c\,P(s)\frac{\sqrt{1+T}}{1+n(s)}.
$$
Here Q(s) is the highest reward among solutions generated from s, or R(s) if s has not been expanded. P(s) is a prior based on the state's reward rank in the buffer. The count n(s) records how often s or its descendants have been expanded, T is the total number of expansions, and c controls exploration.

The first term favors states that have led to strong solutions. The second favors promising states that have been explored less. Using the maximum child reward rather than the average matches the discovery goal: find one exceptional solution while preserving exploration.

## Putting them together

TTT-Discover combines the entropic objective for policy updates with PUCT for selecting starting states:
$$
\theta_{i+1}=\theta_i+\eta\nabla_\theta J_{\beta(s_i)}(\theta_i),\qquad s_i\sim\operatorname{PUCT}(H_i).
$$
Each attempt adds a new solution and its reward to the reuse buffer, and the policy learns from the attempt. Search reuses promising states while training changes the policy itself.

