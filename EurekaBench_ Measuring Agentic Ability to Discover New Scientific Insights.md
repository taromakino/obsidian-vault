
**Authors:** Jiayi Geng, Zhengxuan Wu, Kevin S. Chen, Seungone Kim, Joseph Janssen, Zora Zhiruo Wang, Bhupalee Kalita, Runtian Gao, Aaron Ho, Andrew Oakleigh Nelson, Olexandr Isayev, Francisco Villaescusa-Navarro, Ching-Yao Lai, Howard Chen, and Graham Neubig  
**Year:** 2026  

## 1. Research Question and Motivation

EurekaBench asks whether autonomous AI agents can discover scientifically meaningful mechanisms, rather than merely fit observations. A model may predict a phenomenon accurately without explaining why it occurs or what would happen under a new intervention. The paper argues that scientific discovery should be judged partly by the useful conclusions that a mechanism enables, not just by its prediction score.

## 2. Benchmark Design

The benchmark contains 26 research tasks across neuroscience, geophysics, plasma physics, astrophysics, computer science, and chemistry. Each task starts with observations that call for an explanation. An agent can use a documented computational environment to conduct experiments and refine hypotheses. The evaluated agents received up to four hours, 1,000 iterations, and one H100 GPU per task.

The agent submits `mechanism.md`, a free-form explanation of its proposed mechanism; `mechanism.py`, an executable Python implementation; and `experiments.log`, a record of its investigation. The benchmark compares these submissions with existing human-developed scientific mechanisms. The human reference is prior scientific work, not researchers working under the same four-hour budget.

## 3. Evaluation Methodology

Each task has its own expert-designed rubric covering scientific constraints, predictive accuracy, and scientific insights. Scientific constraints test whether a proposed mechanism respects basic domain assumptions. Predictive accuracy is measured by running `mechanism.py` on held-out data. Scientific insight is measured using expert-written yes-or-no questions about conclusions that should follow from a useful mechanism. There are 306 such questions across the benchmark.

Two LLM judges, Claude Opus 4.8 and GPT-5.6 Sol, examine the proposed mechanism and experimental record and can conduct further experiments. A scientific-insight question earns credit when the judge finds evidence that its conclusion can be derived from the mechanism. The combined score is set to zero for a mechanism that fails the scientific-constraint checks. Importantly, the insight evaluation can credit implications derived by the judges even if the original agent did not explicitly state those insights during its own investigation.

## 4. Main Empirical Results

The strongest AI systems came close to the human reference on the benchmark's normalized held-out predictive-accuracy metric. GPT-6 Astra scored 47.4%, versus 48.8% for the human reference. Yet its scientific-insight score was 29.4%, versus 69.7% for human-developed mechanisms. Claude Fable 5.1 also scored 47.4% on predictive accuracy but reached 42.4% on scientific insights. On the combined score, the best AI result was 29.8%, compared with 63.7% for the human reference.

Many AI submissions also failed the scientific-constraint checks. These results suggest that strong prediction on held-out observations does not automatically imply a scientifically adequate explanation. They do not establish that an AI mechanism is always different from, or less correct than, the human mechanism.

## 5. Authors' Explanation of the Performance Gap

The authors argue that agents often approach mechanism discovery as numerical optimization. They tune parameters until a model reproduces observed behavior, while paying less attention to whether the parameters have a justified scientific interpretation. In one neuroscience example, an agent matched a target behavior by changing synaptic parameters without establishing that the fitted settings explained the underlying process.

The authors also find that agents adopt insufficiently tested assumptions, neglect competing explanations, and fail to run informative experiments after recognizing uncertainty. In a plasma-physics example, an agent noticed a possible numerical artifact but did not perform the test that could help resolve it. Agents sometimes identify interesting observations without exploring their broader implications. The central shortcoming is therefore not simply imperfect prediction, but weak experimental reasoning about mechanisms and uncertainty.

## 6. Limitations and Alternative Explanations

The benchmark cannot fully separate a failure to discover a good mechanism from a failure to describe or demonstrate one. The explanation in `mechanism.md` is free-form, and the paper does not formally verify that it is semantically equivalent to `mechanism.py`. An AI agent could conceivably implement a mechanism similar to the human reference while explaining it poorly. Conversely, code could fit observations without representing the mechanism claimed in prose. The LLM judges inspect code and may run experiments, which reduces dependence on prose alone, but does not eliminate this ambiguity.

The insight score is also an imperfect measure of discovery. It depends on expert-selected questions and on LLM judges interpreting the mechanism correctly. A valid but unexpected insight may not appear in the rubric, while a judge might award credit too generously. The authors acknowledge these issues. The benchmark covers only 26 tasks, with uneven domain coverage, and the human comparison uses established research rather than a matched experimental budget.

Finally, held-out predictive accuracy is not the same as robustness to major distribution shift. The paper does not primarily study training-to-test generalization gaps or performance across changing environments. Its headline result concerns the gap between predictive performance and measured scientific insight, even when predictions are evaluated on held-out observations.

## 7. Implications and Open Research Questions

EurekaBench suggests that autonomous discovery agents need to choose experiments for the information they provide, not only for immediate improvements in a prediction metric. An informative experiment might distinguish two competing hypotheses, expose a confounder, or falsify a seemingly successful model. This links the problem to causal reasoning, active learning, and experimental design under uncertainty.

Several questions remain open. How can we verify that executable code faithfully implements a stated mechanism? How can an evaluator assess scientific implications directly from code without depending heavily on the quality of the written explanation? How can we recognize valid insights outside a predefined rubric? How should agents select experiments that reduce uncertainty, and how can discovered mechanisms be tested under distribution shift?

The main takeaway is that **predictive success and scientific insight are different objectives**. EurekaBench provides evidence of a large gap between them in current agents. But how much of that gap reflects shortcomings in discovery itself, explanation quality, or the evaluation procedure is not yet conclusively resolved.
