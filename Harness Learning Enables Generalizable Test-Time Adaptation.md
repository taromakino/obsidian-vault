**Authors:** Alvin Zhang, Xuecheng Liu, Zixuan Wang, Fahim Tajwar, Daman Arora, Ruslan Salakhutdinov, Daniel Khashabi, Yuda Song, Andrea Zanette  
**Year:** 2026
## Problem setup

The paper trains a **proposer** model to edit the *harness* (the code controlling an AI agent's workflow) used by a **frozen solver** language model. The harness can determine how the solver is prompted, uses tools, checks its work, and produces an answer. The solver's weights do not change.

The proposer receives the current harness, a task description, and feedback from running the solver. It outputs a code edit intended to make the harness perform better. The key contribution is showing that a proposer trained on some tasks can learn an editing strategy that also improves harnesses for unseen task families.

## Teacher, SFT, and RL

1. **Teacher:** A larger, roughly 35B-parameter model generates examples of successful harness edits. It is also evaluated as a proposer baseline.
2. **Supervised fine-tuning (SFT):** A smaller 4B proposer is trained to imitate teacher-generated edits.
3. **Reinforcement learning (RL):** Starting from the SFT proposer, the authors use GRPO. For each current harness and execution report, the proposer generates several candidate edits. Each revised harness is run with the frozen solver and scored. The candidates' relative rewards are used to update the proposer's weights.

The reward is based on the **performance of the edited harness**, plus validity/execution-related reward, rather than explicitly on the score difference from the previous harness.

## Single-step vs. multistep RL

Represent one revision as a training example $(x_t, y_t, r_t)$, where

$$
x_t = (h_t, \text{feedback}_t, \text{task}),
$$

- $h_t$ is the current harness.
- $\text{feedback}_t$ reports how that harness performed on development questions.
- $y_t$ is the proposer's suggested code edit, producing $h_{t+1} = \operatorname{Apply}(h_t,y_t)$.
- $r_t$ is the reward for the edited harness: its score on evaluation questions used for training, plus a validity/execution-related reward. In shorthand,

$$
r_t = J(h_{t+1};Q_{\mathrm{score}}) + v_t.
$$

The reward depends on **the immediate revised harness $h_{t+1}$**, not on the improvement relative to $h_t$ or the final result of a longer editing trajectory. GRPO compares rewards of multiple edits proposed for the same input.

**Single-step RL:** The proposer trains on examples $(x_0,y_0,r_0)$ that start from the seed harness $h_0$. It learns to make one useful edit given the seed harness and its execution feedback.

**Multistep RL:** The system first builds a revision sequence:

$$
h_0 \rightarrow h_1 \rightarrow h_2 \rightarrow \cdots \rightarrow h_T.
$$

After each revision, it runs the current harness and obtains new feedback. This generates training examples

$$
(x_0,y_0,r_0),\ (x_1,y_1,r_1),\ \ldots,\ (x_{T-1},y_{T-1},r_{T-1}).
$$

Each is optimized as an **individual revision step with its own immediate reward**. Thus, multistep training exposes the proposer to *already modified harnesses*, but does **not** train it using a reward based on final harness performance $J(h_T)$.

**What if they optimized complete multistep trajectories using the final outcome instead?**

- **Potential advantage:** The proposer could learn long-term editing strategies, including edits that initially hurt performance but enable later improvements.
- **Potential disadvantages:** Credit assignment becomes harder, reward estimates may be noisier, and full trajectories are computationally expensive to evaluate.

The important distinction is **immediate versus long-horizon credit assignment**, not whether training examples are physically stored separately. One can process individual steps while still assigning rewards based on future outcomes. The paper does not report an ablation that optimizes final-harness rewards over full trajectories.

## Domain adaptation, not pure domain generalization

At test time, the proposer edits the harness for a new task family using **labeled development questions or automatic correctness feedback from that new family**. Final evaluation uses separate held-out questions.

Therefore, the results show **transfer of a learned adaptation procedure** to unseen task families, not zero-shot generalization without target-domain feedback. This is closer to supervised domain adaptation than pure domain generalization. The model weights stay frozen during this adaptation; the harness code changes.

## Reasoning Gym: training and held-out families

Across the paper's selected Reasoning Gym splits, there are **50 distinct task families**:

| Split | Task families |
| --- | ---: |
| SFT training | 21 |
| RL training | 5 |
| Overlap between SFT and RL | 2 |
| **Unique training families** | **24** |
| **Held-out OOD families** | **26** |

So the authors first train the proposer with SFT on 21 families, then RL on 5 families (3 new and 2 overlapping). The other 26 families are unseen during proposer training.

For each held-out family, the authors use **25 labeled development questions** to produce feedback and **75 separate held-out questions** to evaluate edited harnesses. Each proposer produces **8 independent edits** starting from the same seed harness. The reported score averages across the 8 edits and then across the task families.

**Results across all 26 held-out families** (average held-out score; higher is better):

| Proposer | Score |
| --- | ---: |
| Base 4B | 0.282 |
| SFT 4B | 0.415 |
| 35B teacher | 0.501 |
| **RL 4B** | **0.528** |

The RL-trained 4B proposer is better than the larger teacher by **0.027 absolute score points** (2.7 percentage points). This is a **modest OOD gain over the teacher**, although RL improves substantially over the untrained and SFT-only 4B proposers.

## Takeaway

The paper's main result is that a small proposer can learn a reusable strategy for improving a frozen solver's harness, and that strategy transfers to unseen task families. The gains over a larger teacher on all 26 held-out Reasoning Gym families are relatively modest. Two important limitations are the need for **target-family correctness feedback at test time** and the lack of an experiment optimizing **final outcomes of complete multistep revision trajectories**.
