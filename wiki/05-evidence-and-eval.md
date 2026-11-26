# 05 · Evidence and evaluation

[Wiki home](README.md) · previous: [Decision Layer](04-decision.md) · next: [RSI and roadmap](06-rsi-and-roadmap.md)

> **First, build a ruler.** Collect real decision moments, label which actions were acceptable at each one, and seal them as a holdout. From then on every change, whether a new model, a new persona or a new policy, is compared on this set first; and we check that versions the Eval Set calls better are also better in randomized online tests.

Objective **O1** · workstream **WS-E** · W1–W12, v1 sealed in W6. Without one shared evaluation, an improvement from the method cannot be told apart from an improvement that came from swapping the base model or the sample.

Evaluation has two layers: the **Evidence Ledger** underneath, and the **Eval Set** on top.

## Evidence Ledger

One row per decision moment, written at decision time and completed as outcomes mature.

| Field | Content | Used for |
|---|---|---|
| moment_id · user_key | pseudonymous IDs; rows grouped by user_key | user-clustered CIs |
| as_of · state_hash | state rebuilt only from events observed at or before as_of, and its hash | no future leakage |
| context | last k turns plus reply latency, left-on-read, silence, likes, posts, profile changes | model input |
| persona | User Persona snapshot, Agent Persona, persona context c | O2 conditions and counterfactuals |
| p(a) · π<sub>b</sub> | calibrated probabilities over six actions; the executed action's actual probability | doubly robust estimation |
| executed · receipt | action before and after the gate; Intent / Attempt / Receipt IDs | execution closure |
| label | adjudicated acceptable-action set, preferred action, rationale | gold standard |
| strata · outcome | need type, purchase stage, non-pushing moment or not; T1–T4 status | stratified reporting |
| versions | policy, persona, gate, templates, models | attribution and reproduction |

The as-of reconstruction is guarded by a suite of **256 test cases** for future-information leakage.

## Outcomes T1–T4

<img src="https://raw.githubusercontent.com/LittleA-Agentic-RL-Harness/.github/main/profile/assets/outcomes.svg" width="100%" alt="T1 to T4 and the matured, reversed and censored cases" />

| | Outcome | Definition | Window |
|:---:|---|---|---|
| **T1** | accept | within 72 h of the decision moment, the user clicks a product or payment link, clearly says they will buy, or confirms a concrete offer (edition, price, delivery) | 72 h, fixed after pre-registration |
| **T2** | pay | an order linked to the session (dedicated link, order number or coupon) is paid; "deal" always means T2 | pre-registered |
| **T3** | fulfil | delivered, or the digital good handed over | by logistics times |
| **T4** | retain | when the refund window (counted from delivery) ends, the order has neither been refunded nor cancelled | per channel policy |

- **censored**: the observation cutoff came before the end of that tier's window. Counted neither as success nor failure, and modelled as delayed feedback. Recording unfinished refund windows as failures would under-estimate conversion.
- **reversed**: refunded or cancelled inside the window. T4 = 0; if cancelled before fulfilment, T3 = 0 too; lower tiers already reached keep their values.
- "Moved to a private chat" is a process metric and does not count as T1. **Repurchase** means the same user reaches T2 again within 90 days after T4.
- Before T4 matures, a **surrogate index** built from T1–T3 can estimate long-term effects, provided surrogacy and comparability hold; comparability is the one that can be tested on matured samples.

## Doubly robust offline evaluation

Each decision moment is one contextual-bandit step. With an accurate π<sub>b</sub>, and with every action the new policy might choose having positive probability under the behaviour policy, the doubly robust estimate is unbiased, and the more accurate the reward model, the lower its variance.

~~~math
\hat V_{\mathrm{DR}}(\pi)=\frac{1}{n}\sum_{i}\Big[\sum_{a}\pi(a\mid x_i)\,\hat q(x_i,a)+\frac{\pi(a_i\mid x_i)}{\pi_b(a_i\mid x_i)}\big(r_i-\hat q(x_i,a_i)\big)\Big]
~~~

~~~math
\pi_b(a\mid x)=(1-\varepsilon)\,\mathbb{1}\big[a=\arg\max_{a'}p(a'\mid x)\big]+\varepsilon\,q(a\mid x)
~~~

So the executor cannot simply take the arg-max. It takes the most probable action with probability 1 − ε and, with probability ε (pre-registered), samples from the calibrated distribution q over actions the compliance gate allows. The executed probability is logged as π<sub>b</sub>. Human-rewritten decisions are flagged and excluded from DR. r<sub>i</sub> is T1 within 72 h in the primary analysis, with T4 reported as it matures. The bandit view ignores how later decisions affect the outcome, which is why offline conclusions are tested online (KR 1.3).

## Eval Set v1

| | Subset | Content |
|---|---|---|
| **A** | Decision Eval | ≥ 1,200 decision moments, stratified; acceptable-action set plus preferred action; α ≥ 0.80; ≥ 300 E or D moments including non-pushing moments |
| **B** | Persona Eval | ≥ 300 users with gold personas (double-labelled, adjudicated) over need, stage, taste, tone, pace, boundaries; ≥ 300 counterfactual persona pairs |
| **C** | Outcome-linked Real Cases | real sessions with T1–T4 status, grouped by user, with logged π<sub>b</sub> and as-of state; covers Teacher, Student, third-party baselines and personas |
| **D** | Simulation Track | user simulators replay the same scenes; reports only the sim–real gap |
| **E** | Track S (pilot) | social intelligence: models and human reference groups answer the same tasks |

After the spec freezes, data is **split by time**: the holdout covers a period after the training data. The holdout is held by the **Eval Steward**; training and production accounts cannot read it. At sealing in W6 the SHA-256 of the holdout manifest is published, so anyone can verify that the holdout used in a report has not changed since. After that, changes only come as new versions. This plays the role of pre-registration; static public benchmarks are easily contaminated, and time-splitting plus sealing is our answer.

LLM user simulators tend to over-cooperate and over-estimate agent success; in a real sales setting, simulated users voiced resistance far less often than real ones. That is why the Simulation Track reports only the gap.

### Stratified sampling

| Dimension | Values | Requirement |
|---|---|---|
| need type | clear (asks for a specific product) · partial (category, no product) · none (LittleA recommends proactively) | each ≥ 20% |
| purchase stage | discover · learn · compare · decide · after-sales | each ≥ 10% |
| preferred action | A–F | each ≥ 100; E and D together ≥ 300 |
| non-pushing moments | preferred action E stop: wait · advise not to buy now (with reason) | each ≥ 60 |
| source | real history · constructed counterfactual | real ≥ 70%; counterfactual families ≥ 150 |

### Labelling protocol

Labelling manual v1 ships in W2 with operational definitions for every action and decision rules for timing actions and non-pushing moments. One calibration round on 50 moments revises the manual; after that, two labellers work independently and a third adjudicates disagreements. Agreement is computed **before adjudication**: Krippendorff's α for rotating labellers; for fixed pairs, Cohen's κ is reported as well, read on the Landis–Koch scale. Preferred action needs α ≥ 0.80; timing actions (A engage and E stop) need α ≥ 0.667, the floor for tentative conclusions; acceptable-action sets report α per action as binary judgements. Labellers include people with front-line sales experience, and their agreement with the others is reported separately.

## Validity check (KR 1.3)

At least four online pairwise comparisons, each listed in the pre-registration: matched vs default persona (KR 2.3), and from Gate 2 Student vs Teacher, Student vs Student + default persona, and Student with slow path vs fast path only. The offline metric is fixed in advance: acceptable-action rate on Decision Eval.

Only comparisons whose online 95% CI excludes 0 count. Among them, the share where the offline direction agrees must be **≥ 3/4**, and no comparison may be significant offline and significantly opposite online. With four comparisons, chance alone reaches ≥ 3 agreements about 31% of the time, so the result is reported as **preliminary** validity evidence. Fewer than three online-significant comparisons is recorded as insufficient evidence and carried into Phase 2. Disagreements are analysed one by one and written into v2's revision notes.

## Live Eval Harness

One randomized-experiment facility shared by O1, O2 and O3, following the practice of trustworthy online controlled experiments. Ready in W4, after an A/A test.

~~~mermaid
flowchart LR
    U["Users<br/>books"] --> R{"Randomizer<br/>unit = user, fixed arm<br/>p(arm) and seed logged"}
    R --> A["Arm A · control<br/>e.g. default persona, Teacher"]
    R --> B["Arm B · treatment<br/>e.g. matched persona, Student"]
    A --> O["OmniHub<br/>same execution environment"]
    B --> O
    O --> L[("Ledger and analysis<br/>T1 primary, T2–T4 back-filled<br/>ITT, user-clustered CI")]
    G["Guardrails, always-valid monitoring<br/>stop-request · complaint · block and report · refund rates"] -.->|stop the arm| R
~~~

- **Randomized by user**, arm fixed for the whole experiment; p(arm) and seed go into the Ledger.
- **Pre-registered** before the first user enrols: primary metric, MDE, sample size, analysis, guardrail thresholds.
- **One read** of the primary analysis at the pre-registered sample size (ITT, 95% CI clustered by user).
- **Always-valid confidence sequences** are used only for continuous guardrail monitoring, which can be watched at any time without inflating type-I error. Any guardrail degrading past its threshold stops that arm.

## Track S

To claim an agent's social intelligence improved, we need an evaluation that measures models and people on the same tasks. Track S adds the judgements specific to commercial relationships: when to push, when to stop, when to advise not buying.

| Task | Format |
|---|---|
| **S1** decision-moment items | the Decision Eval format, scored against the acceptable-action set |
| **S2** interactive scenes | SOTOPIA-style multi-turn scenes, rated by the Judge **and** humans (LLM-only rating over-scores socially trained agents) |
| **S3** theory-of-mind tasks | false belief and intent inference, the same items as human participants; validated measures such as Reading the Mind in the Eyes where needed |

Human reference groups: a general-population sample, consultative sales professionals and the A-Lab labelling team; their selling style is profiled with the SOCO and ADAPTS scales. Systems under test: frontier models, the Teacher System and Students S and M. Results are reported as **percentiles of the human reference distribution**; no values are assumed before the pilot. All participants give informed consent, the protocol passes ethics review, and Phase 1 includes no clinical populations. Because a GPT-4 given a little demographic information out-persuaded human opponents in a pre-registered debate study, Track S also reports E stop recall.

- **H-S1** · On interactive scenes (S2), frontier LLMs score below the median of the professional-sales reference group.
- **H-S2** · Students trained with A-Lab's method narrow that gap, and the gap narrows in the same direction as online T4 retention changes.

## Key results

| KR | Target | Acceptance |
|---|---|---|
| **1.1** Eval Set v1 sealed | W6 | Composition as above; agreement computed before adjudication; time split; holdout handed to the Eval Steward; SHA-256 published. |
| **1.2** Ledger coverage | ≥ 99% | Production decision moments carrying all three records (p(a) and π<sub>b</sub>; as-of state hash; all component versions) ÷ all production decision moments, including gate-blocked ones, computed on the full log. Receipt closure 100%. The Eval Steward also audits 200 moments a week (≤ 1 error) and runs the 256 tests; two consecutive passing weeks. |
| **1.3** Validity check | ≥ 3/4 agree | As above; recomputed by the Eval Steward in W12. |

## References

- Dudík, Langford, Li. [Doubly Robust Policy Evaluation and Learning](https://arxiv.org/abs/1103.4601). ICML 2011.
- Chapelle. [Modeling Delayed Feedback in Display Advertising](https://doi.org/10.1145/2623330.2623634). KDD 2014.
- Athey, Chetty, Imbens, Kang. The Surrogate Index. Review of Economic Studies, 2026; early version [NBER WP 26463](https://www.nber.org/papers/w26463).
- Kohavi, Tang, Xu. [Trustworthy Online Controlled Experiments](https://doi.org/10.1017/9781108653985). Cambridge University Press, 2020.
- Johari et al. [Always Valid Inference: Continuous Monitoring of A/B Tests](https://doi.org/10.1287/opre.2021.2135). Operations Research, 2022.
- Nosek et al. [The preregistration revolution](https://doi.org/10.1073/pnas.1708274114). PNAS, 2018.
- White et al. [LiveBench](https://arxiv.org/abs/2406.19314). ICLR 2025.
- Zhou et al. [Mind the Sim2Real Gap in User Simulation for Agentic Tasks](https://arxiv.org/abs/2603.11245). 2026.
- Chen. [Simulated Customers Never Walk Away](https://arxiv.org/abs/2606.20708). 2026.
- Zhou et al. [SOTOPIA](https://arxiv.org/abs/2310.11667). ICLR 2024 · Wang et al. [SOTOPIA-π](https://arxiv.org/abs/2403.08715). ACL 2024.
- Strachan et al. [Testing theory of mind in large language models and humans](https://doi.org/10.1038/s41562-024-01882-z). Nature Human Behaviour, 2024.
- Salvi et al. [On the conversational persuasiveness of GPT-4](https://doi.org/10.1038/s41562-025-02194-6). Nature Human Behaviour, 2025.
- Hayes, Krippendorff. [A Standard Reliability Measure for Coding Data](https://doi.org/10.1080/19312450709336664). 2007 · Landis, Koch. [The Measurement of Observer Agreement](https://doi.org/10.2307/2529310). 1977.
- Saxe, Weitz. [The SOCO Scale](https://doi.org/10.1177/002224378201900307). 1982 · Spiro, Weitz. [Adaptive Selling](https://doi.org/10.2307/3172551). 1990.
