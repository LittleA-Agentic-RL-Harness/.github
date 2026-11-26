# LittleA Technical Wiki

The engineering reference for the LittleA Agentic RL Harness: how LittleA senses a person, chooses a persona, decides its next action, executes it, and proves, with evidence that matures over weeks, whether that action earned trust.

[← Organization README](https://github.com/LittleA-Agentic-RL-Harness) · [中文 README](../profile/README.zh-CN.md)

~~~mermaid
flowchart LR
    ARCH["01 · Architecture"] --> OMNI["02 · OmniHub"]
    ARCH --> PER["03 · Persona Layer"]
    ARCH --> DEC["04 · Decision Layer"]
    OMNI --> EVI["05 · Evidence and evaluation"]
    PER --> EVI
    DEC --> EVI
    EVI --> RSI["06 · RSI and roadmap"]
    GLO["Glossary"] -.-> ARCH
~~~

| # | Page | Read it for |
|:---:|---|---|
| 01 | [Architecture](01-architecture.md) | The six components, the decision moment, the six RSI capabilities and what is already running |
| 02 | [OmniHub](02-omnihub.md) | Execution on official APIs and real phones, the compliance gate, Intent → Attempt → Receipt |
| 03 | [Persona Layer](03-persona.md) | User Persona Card, Agent Persona Pool, the Matcher, R0 → R1 → R2, KR 2.1–2.3 |
| 04 | [Decision Layer](04-decision.md) | Teacher System, off- and on-policy distillation, calibration, routing, Gate 1 and Gate 2, KR 3.1–3.3 |
| 05 | [Evidence and evaluation](05-evidence-and-eval.md) | Evidence Ledger, T1–T4, doubly robust evaluation, Eval Set v1, Live Eval Harness, Track S, KR 1.1–1.3 |
| 06 | [RSI and roadmap](06-rsi-and-roadmap.md) | Recursive self-improvement, workstreams, five research directions, A2C → A2A → A3A |
| — | [Glossary](glossary.md) | Every term, in English and 中文 |

## Conventions

- **The unit is the decision moment.** Ledger coverage, evaluation sampling and attribution all count decision moments.
- **"Deal" always means T2** (paid). **"Matured outcome", used as a reward, always means T4** (kept after the refund window).
- **Numbers in figures are illustrative** unless a page says they are measured.
- **Thresholds are pre-registered.** Margins, windows, ε, τ and sample sizes are written into the spec before the first user is enrolled, and changed only by a new spec version.
- **Definitions live in the [Glossary](glossary.md).** A pull request that changes a definition also updates the glossary.

<sub>Source of truth: A-Lab Research Plan, Phase 1 (from 2026-11-09). Edits go through pull requests to this repository.</sub>
