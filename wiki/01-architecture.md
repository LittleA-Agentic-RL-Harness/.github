# 01 · Architecture

[Wiki home](README.md) · next: [OmniHub](02-omnihub.md)

<img src="https://raw.githubusercontent.com/LittleA-Agentic-RL-Harness/.github/main/profile/assets/architecture.svg" width="100%" alt="LittleA layered model" />

## Six components

| Component | Input → output | Owner | Notes |
|---|---|---|---|
| **Sensing** | raw events → state z<sub>t</sub> | WS-P | Most signals are not text: likes, posting rhythm, reply latency, left-on-read, silence. Only events observed at or before t are used. |
| **Persona Layer** | z<sub>t</sub> → User Persona u → Agent Persona p\* → context c | WS-P · O2 | c conditions both the Decision Layer and the Output core. Weights stay fixed; the action distribution and the tone change with c. |
| **Decision Layer** | (z<sub>t</sub>, c) → one of six actions | WS-D · O3 | Fast path: a Student under 20B when max p ≥ τ. Slow path: action F hands the moment to the Teacher and frontier reasoning. |
| **Output core** | (action, c) → reply text | integrated | A frontier LLM. LittleA does not train this layer; providers can be swapped. |
| **OmniHub** | action → typed external effect → receipt | WS-X | Official messaging APIs, GUI agents on real phones and cloud sandboxes. Compliance gate, then Intent → Attempt → Receipt. See [OmniHub](02-omnihub.md). |
| **Judge** | candidate version → verdict | WS-E · O1 | Uses only the sealed holdout and matured outcomes. Never trained, never conditioned on a persona; tested separately for persuasion and sycophancy. |

Training rewards come from matured outcomes in the Evidence Ledger, never from the Judge's scores. Keeping the Judge outside the persona lens is what stops the system from grading itself.

## The decision moment

A **decision moment** is any point at which LittleA must decide its next action:

1. a user message arrives;
2. the user does something observable (a like, a read receipt, a profile change);
3. silence exceeds a pre-registered length, 24 hours by default.

Each decision moment yields exactly one action and exactly one Evidence Ledger row.

| Step | What happens | Objective |
|:---:|---|:---:|
| 1 | Likes, posts, DMs, reply latency, left-on-read and silence enter as events. | — |
| 2 | Sensing rebuilds the person's state z<sub>t</sub> from events observed up to t. | O2 |
| 3 | The Persona Layer updates the User Persona; the Matcher picks the Agent Persona with the highest expected matured outcome, exploring within top-k and logging π<sub>b</sub>. | O2 |
| 4 | The Decision Model gives calibrated p(a) over six actions; the executor takes the arg-max with probability 1 − ε and explores with ε, logging the executed probability π<sub>b</sub>. | O3 |
| 5 | A frontier LLM writes the reply from the action and the persona context. | integrate |
| 6 | The compliance gate checks disclosure and stop requests; OmniHub compiles a typed action and runs Intent → Attempt → Receipt. The receipt flows back as an event. | WS-X |
| 7 | The Ledger records p(a), π<sub>b</sub>, the as-of state hash, component versions and the receipt. | O1 |
| 8 | Outcomes mature T1 → T2 → T3 → T4; anything still inside its window at the cutoff is censored. | O1 |
| 9 | The Judge evaluates candidate versions on the sealed holdout and matured outcomes. | O1 |
| 10 | Only versions that pass ship; the next moment uses the updated persona priors and Decision Model. | all |

~~~mermaid
stateDiagram-v2
    [*] --> Observed: message, behaviour or silence over 24 h
    Observed --> Sensed: z_t as-of t
    Sensed --> Matched: Agent Persona, context c
    Matched --> Decided: p(a), π_b
    Decided --> Fast: max p ≥ τ
    Decided --> Slow: F defer
    Slow --> Gated: Teacher picks action
    Fast --> Gated: Student action
    Gated --> Executed: Intent → Attempt → Receipt
    Gated --> Blocked: gate refuses
    Executed --> Logged
    Blocked --> Logged
    Logged --> [*]
~~~

<sub>Blocked moments are logged too: Ledger coverage counts every production decision moment, including those the compliance gate stopped.</sub>

## Six RSI capabilities and where they live

The product describes LittleA by six capabilities. Each maps to one module, and each module has a different maturity today.

| # | Capability | What it does | Module | Status |
|:---:|---|---|---|---|
| 01 | **Executing** | Uses social apps on its own phones the way people do, learning by exploring rather than by scripts | OmniHub | running |
| 02 | **Sensing** | Takes social signals in natively, puts actions out natively, and decides when to act | Sensing → Decision Model; target: Sensing–Action Model | Phase 1 uses an LM backbone with a decision head; native pretraining is research |
| 03 | **Matching** | Places a person in persona space and becomes the matching LittleA | Persona Layer | O2 · R0 |
| 04 | **Bonding** | Finds a path to long-term trust; long, medium and short horizon strategies score each option | Long–Short Strategy Model | research |
| 05 | **Evolving** | Improves from real outcomes: each generation rolls out, is evaluated, harvested and mutated | RSI core, version lineage | lineage running; self-evolution is research |
| 06 | **Coexisting** | Lives alongside people one relationship at a time; trusted relationships link into a network | the relationships themselves | the outcome being measured |

## What is already running

| | What | Where |
|---|---|---|
| **Commerce loop** | Discover potential users, identify needs, multi-turn conversation, recommend, guide to checkout; end-to-end in the books category | product |
| **OmniHub · execution** | Official messaging APIs and GUI agents on real phones and cloud sandboxes; navigation, budgets and accounting are done by deterministic programs | littlea-agent-infra, littlea-android-agent-harness |
| **Lineage · persona** | Versions branch as Mother / Child and keep iterating; User Personas and persona assignment are structured and used online | product |

## Build, integrate, rent

LittleA builds only the top of the stack: the one-to-one relationship with each user, the Persona Layer, the Evidence Ledger, the Eval Set and the Decision Model. Everything below is supplied by others and changes fast.

| Layer | How LittleA uses it |
|---|---|
| Relationship and results (Persona Layer, Evidence Ledger, Eval Set, Decision Model) | **build** · O1–O3 |
| Distribution and messaging platforms | compliant access through official APIs and real devices |
| Checkout protocols and merchants | integrate, do not build |
| Third-party decision models | baseline in the same protocol as the Student |
| Closed frontier models | Teacher and the Output core's language |
| Open-weight models | Student bases, post-training only in Phase 1 |
| Compute and cloud | rented on demand |

<sub>As decision layers and base models get cheaper, the difference comes from two things: the long-term relationship with each user, and the ability to prove that relationship produces verifiable results. The first is the product; the second is A-Lab's research question.</sub>
