# 04 · Decision Layer

[Wiki home](README.md) · previous: [Persona Layer](03-persona.md) · next: [Evidence and evaluation](05-evidence-and-eval.md)

> **Teach the Teacher's judgement to a Student.** Build a Teacher from frontier LLMs plus human review, with no limit on cost or latency, and run it on real users in the books category. Then distil its choice at every decision moment into a Student that is cheap and fast. The Student only chooses the next action; the reply text is still written by a frontier LLM.

Objective **O3** · workstream **WS-D** · W1–W12, books.

## The Decision Model

An LM backbone with a **decision head** that outputs calibrated probabilities over the six actions **A engage · B ask · C recommend · D support · E stop · F defer**. Two sizes: **S ≤ 4B** and **M 8–9B**, both post-trained from open-weight bases (no pretraining in Phase 1).

## Why books first

| | Reason |
|---|---|
| **Standard supply** | The same edition is sold through many channels, so the difference lies in recommendation and conversation. |
| **Low price** | Each decision carries little risk, which suits large randomized tests. |
| **Visible repurchase** | Buying can be watched after T4, a test of the long-term relationship. |
| **Fast preference reveal** | What and how someone reads shows within a few turns. |
| **Huge SKU space** | The long tail is long enough that matching quality shows. |

## Three stages

~~~mermaid
flowchart TB
    T0["<b>Stage 0 · Teacher System</b><br/>frontier LLM · multi-step reasoning · tools<br/>Persona Layer · case memory · human review<br/>real users, books, version frozen per round"]
    TR[("Trajectories<br/>z_t · c · π_T before review · human choice<br/>rationale · executed action · receipt · T1–T4")]
    SA["<b>Stage A · off-policy</b><br/>SFT on Teacher soft labels π_T"]
    subgraph SB["Stage B · on-policy, 2+ rounds"]
        direction LR
        P1["Student proposes"] --> P2["Teacher reviews<br/>takes over if unacceptable"] --> P3["Teacher labels the<br/>Student's own states"] --> P4["Aggregate<br/>D ← D ∪ D_student"] --> P5["Retrain Student"]
    end
    G1{"Gate 1 · sealed Decision Eval"}
    G2{"Gate 2 · live randomized"}
    DEP["Deploy: fast path Student · slow path F → Teacher · words → frontier LLM"]
    T0 --> TR --> SA --> SB --> G1 -->|pass| G2 -->|pass| DEP
    G1 -.->|fail| SB
~~~

| Stage | What | Detail |
|---|---|---|
| **0 · Teacher** | Frontier LLM, multi-step reasoning and tools, Persona Layer, case memory, human review; no limit on cost or latency; real users | Moves gradually from human-in-the-loop to engineered. The Teacher version used for each distillation round is frozen and recorded in the Ledger. |
| **A · off-policy** | Student learns the Teacher's soft labels π<sub>T</sub>: the frontier model's action distribution before human review | The human's final choice is kept as a separate hard label. The work is mostly data cleaning and human checking. |
| **B · on-policy** | On small controlled traffic, the Student proposes, the Teacher reviews every action before it runs and takes over when needed | Trajectories are now driven by the Student; the Teacher labels the full distribution on those states. DAgger-style aggregation, then retrain. |

## Objectives

~~~math
\text{off-policy:}\quad \min_{\theta}\ \mathbb{E}_{s\sim d^{\pi_T}}\Big[D\big(\pi_T(\cdot\mid s),\ \pi_\theta(\cdot\mid s)\big)\Big]
~~~

~~~math
\text{on-policy:}\quad \min_{\theta}\ \mathbb{E}_{s\sim d^{\pi_\theta}}\Big[D\big(\pi_T(\cdot\mid s),\ \pi_\theta(\cdot\mid s)\big)\Big]
~~~

The only difference is the state distribution. Off-policy learns only the states the Teacher reaches, so the Student has no data once it drifts; on-policy moves the training distribution toward the deployment distribution and limits behaviour cloning's compounding error. Because the six actions are a closed set, the Teacher's full distribution is available and the divergence is computed exactly per state, with no sampling error. The default D is forward KL (cross-entropy on Teacher soft labels), a special case of GKD; reverse KL and JSD are ablations.

**Calibration.** Decision Model probabilities are calibrated by temperature scaling. If a Student is further fine-tuned with RL, binary rewards alone hurt calibration, so a Brier term is added to the reward. The calibrated p(a) is not the executed probability π<sub>b</sub>; π<sub>b</sub> is logged by the executor as described in [Evidence and evaluation](05-evidence-and-eval.md).

## Fast and slow paths

~~~math
\operatorname{route}(s)=\begin{cases}\text{fast: } a=\arg\max_a p_\theta(a\mid s) & \text{if } \max_a p_\theta(a\mid s)\ge\tau \text{ and } a\neq F\\[2pt] \text{slow: F defer} \rightarrow \text{Teacher, frontier reasoning} & \text{otherwise}\end{cases}
~~~

τ is chosen on the Decision Eval **development** set (never the sealed holdout) from the coverage–risk curve, so that on the moments the fast path covers, the Student's acceptable-action rate is no lower than the Teacher's on the same moments minus 2 pp. This is selective prediction applied to dual-process agents. Decision Model inference must be **p95 ≤ 100 ms**, server-timed, excluding text generation and message delivery.

## Two gates

| | Gate 1 · offline | Gate 2 · online |
|---|---|---|
| **Where** | sealed Decision Eval, recomputed by the Eval Steward | Live Eval Harness, randomized by user, four arms |
| **Pass** | acceptable-action rate non-inferior (paired 95% CI lower bound > −2 pp) · E stop and D support recall ≥ 98% · ECE ≤ 0.05 · baselines in the same protocol | T1 non-inferior to the Teacher (margin −2 pp, ITT) · cost per 1,000 decisions ≤ 1/20 · p95 ≤ 100 ms · first deal with no human in the loop |
| **Unlocks** | Student may act without per-action review | deployment of the fast/slow split |

Gate 2's four arms are **Teacher**, **Student**, **Student + default persona** and **Student, fast path only**. Student vs Teacher is the primary comparison; the others feed the validity check (KR 1.3).

## Key results

| KR | Target | Definition and acceptance |
|---|---|---|
| **3.1** Teacher live with trajectories | acceptable-action rate ≥ 90% | Teacher on Decision Eval; trajectory field completeness 100% (z<sub>t</sub>, c, π<sub>T</sub>, human final choice, rationale, executed action, receipt, T1–T4; unmatured outcomes as censored). Done before W8; session count pre-registered in W3 from the data needed for 70–80% top-1 agreement in Stage A. |
| **3.2** Student fidelity | offline non-inferior · ECE ≤ 0.05 | M size after ≥ 2 on-policy rounds: top-1 agreement with the Teacher ≥ 80% on Student-reached states; acceptable-action rate paired-difference 95% CI lower bound > −2 pp; E/D recall ≥ 98% with CI, non-pushing moments reported separately; top-label ECE ≤ 0.05 (15 equal-width bins). S size reported in the same protocol, alongside third-party decision-model and prompted six-way baselines. |
| **3.3** Online non-inferiority and cost | T1 non-inferior · cost ≤ 1/20 | Decision Layer cost per 1,000 decisions, counting Student inference plus Teacher calls and human review time for moments sent via F; text generation and execution, equal across arms, are excluded. The share sent to the slow path is reported. Checkpoint: the first T2 in which every action was chosen by the Student, with no human review or rewrite and no F handover to a Teacher that includes humans. |

**Not adopted.** Training the Decision Model end-to-end on synthetic users. Until a Student passes Gate 1, it runs only on small controlled traffic, and the Teacher reviews every action before it executes.

## References

- Hinton, Vinyals, Dean. [Distilling the Knowledge in a Neural Network](https://arxiv.org/abs/1503.02531). 2015.
- Ross, Gordon, Bagnell. [DAgger: A Reduction of Imitation Learning and Structured Prediction to No-Regret Online Learning](https://arxiv.org/abs/1011.0686). AISTATS 2011.
- Kelly et al. [HG-DAgger: Interactive Imitation Learning with Human Experts](https://arxiv.org/abs/1810.02890). 2018.
- Agarwal et al. [On-Policy Distillation of Language Models (GKD)](https://arxiv.org/abs/2306.13649). ICLR 2024.
- Gu et al. [MiniLLM: Knowledge Distillation of Large Language Models](https://arxiv.org/abs/2306.08543). ICLR 2024.
- Thinking Machines Lab. [On-Policy Distillation](https://thinkingmachines.ai/blog/on-policy-distillation/). 2025.
- Guo et al. [On Calibration of Modern Neural Networks](https://arxiv.org/abs/1706.04599). ICML 2017.
- Damani et al. [Beyond Binary Rewards: Training LMs to Reason About Their Uncertainty](https://arxiv.org/abs/2507.16806). 2025.
- Geifman, El-Yaniv. [Selective Classification for Deep Neural Networks](https://arxiv.org/abs/1705.08500). NeurIPS 2017.
- Christakopoulou, Mourad, Matarić. [Agents Thinking Fast and Slow: A Talker-Reasoner Architecture](https://arxiv.org/abs/2410.08328). 2024.
- Lin et al. [SwiftSage: A Generative Agent with Fast and Slow Thinking](https://arxiv.org/abs/2305.17390). NeurIPS 2023.
