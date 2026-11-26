# 03 · Persona Layer

[Wiki home](README.md) · previous: [OmniHub](02-omnihub.md) · next: [Decision Layer](04-decision.md)

> **Read the person first, then choose the persona.** From public profiles, DMs and interaction behaviour, infer what someone needs, where they are in buying and what they like; then choose, from the Persona Pool, the Agent Persona with the best expected outcome for them. A persona must change which action LittleA takes, not only how it sounds.

Objective **O2** · workstream **WS-P** · W1–W12: R0 online, then a randomized A/B.

~~~mermaid
flowchart LR
    SIG["profile · posts<br/>comments · likes · DMs<br/>reply latency · silence"] --> SEN["Sensing<br/>event encoder"]
    SEN --> UP["User Persona u<br/>need · stage · taste<br/>tone · pace · boundaries"]
    UP --> M{"Matcher<br/>score(u, p) → top-k<br/>softmax(score / β)"}
    POOL[("Persona Pool<br/>priors from real sessions")] --> M
    M -->|"log π_b"| LED[("Evidence Ledger")]
    M --> C["Agent Persona p*<br/>→ context c"]
    C --> DM["Decision Model<br/>p(a given z, c)"]
    C --> LLM["Frontier LLM<br/>reply text and tone"]
~~~

## User Persona Card (R0)

Every attribute stores a **value**, a **confidence** in [0, 1], its **evidence** (a list of signal IDs) and an **as-of** time.

| Attribute | Values | Main evidence |
|---|---|---|
| need / intent | own reading · gift · study or exam · collecting · just browsing, plus one free-text line | DMs, posts |
| purchase stage | discover · learn · compare · decide · after-sales | what they say, how they ask |
| taste | genres and authors; familiarity: beginner · regular · expert | posts, follows, past purchases |
| tone | short and direct · chatty · formal · playful | wording, message length |
| pace | fast · medium · slow, estimated from the reply-latency distribution | reply latency, active hours |
| boundaries | no recommendations · no price talk · certain hours only · stop contact (closed set; anything else goes to free text) | the user's explicit words |

<sub><b>Left on read</b>: shown as read, no reply within 24 h. <b>Silence</b>: not read, no reply within 24 h. Both thresholds are pre-registered. "Stop contact" is set only from the user's explicit words, never inferred from being left on read.</sub>

## Agent Persona and the Persona Pool

An Agent Persona is a reusable card with the same discipline: **role, voice, expertise depth, pacing, disclosure, boundaries**. Pool v0 holds 6–10 of them.

The cold-start prior P(T1 | segment, persona) is estimated from **real historical sessions**: each past session is first labelled with the persona it used, then a Beta prior shrinks small cells so a lucky streak in a rare combination does not dominate. A segment is need × stage. Simulations are used for diagnosis only and never enter the prior. Once T4 matures, it replaces T1 and the prior is re-estimated.

## The Matcher

Formally this is a recommendation problem in which the items are Agent Personas, so collaborative filtering and ranking methods apply.

~~~math
p^{*}=\arg\max_{p\in P}\ \mathbb{E}\big[r^{\mathrm{matured}}\mid u,p\big]\quad\text{s.t. disclosure}=1,\ \text{no manipulation},\ \text{boundaries}(u)\ \text{respected}
~~~

~~~math
\pi_b(p\mid u)=\operatorname{softmax}_{p\in\text{top-}k(u)}\big(\operatorname{score}(u,p)/\beta\big),\qquad c=g(u,p^{*})
~~~

R0 scores with standard recommender machinery: features, similarity, logistic regression. Because the Matcher explores inside top-k and logs π<sub>b</sub>, the Evidence Ledger can compare matching policies offline, but only policies whose choices stay inside top-k.

Most of what we know about a person is not text. Implicit feedback such as clicks is shaped by presentation (position, what else was on screen) and reads more reliably as relative preference than as absolute judgement; generative recommenders that take behaviour sequences as input already work at industrial scale. KR 2.1 therefore reports **text-only** and **text + behaviour** side by side.

## One message, three people

The same incoming line, "I'd like to read something lately, any suggestions?", under three User Personas. Examples are fictional; the point is that the action changes, not just the wording.

| User Persona | Agent Persona | Most likely action | Reply |
|---|---|:---:|---|
| expert reader, fast pace | seasoned book friend: direct, talks editions | **C** recommend | "If you've read the first two, go straight to the new translation; the notes are far better." |
| rarely reads, slow pace | patient guide: asks first, no jargon | **B** ask | "What kind of shows do you like? I'll pick you something easy to get into." |
| just browsing, discover stage | easy chat: does not push | **A** engage / **E** stop | "I'll save you a short list. Pick it up whenever you feel like it." |

## Representation: R0 → R1 → R2

| | Representation | Phase |
|---|---|---|
| **R0** | Persona Card: schema plus confidence, matched to agent nodes | Phase 1 |
| **R1** | Blurry persona: a user as mixture weights over a few prototypes; continuous, interpretable | research design due W12 |
| **R2** | Persona adapter: a PEFT module (LoRA by default, or last layers) per persona, from per-group to per-person | research |

Per-user PEFT modules have precedent, and activation directions corresponding to personality traits can already be measured and steered. Persona vectorization and adapters do not start inside Phase 1.

## Key results

| KR | Target | Definition and acceptance |
|---|---|---|
| **2.1** Persona inference | macro-F1 ≥ 0.80 | Mean of per-attribute macro-F1 over need, stage, tone and boundaries (boundaries as multi-label); confidence ECE ≤ 0.10 (15 equal-width bins); text + behaviour non-inferior to text-only (paired bootstrap). ≥ 90% of new users have a usable Card before LittleA's first reply. Recomputed by the Eval Steward on the sealed Persona Eval. |
| **2.2** Persona changes the action | ≥ 80% of pairs | On counterfactual pairs (same state, only the persona differs), the action distribution moves in the labelled direction with total-variation distance ≥ 0.1. Removing c must lower acceptable-action rate on Decision Eval. Tested in W4 before the A/B, re-tested on the holdout after W6, and again at Gate 1. If it fails, fix the conditioning before starting the A/B. |
| **2.3** Online T1 lift | 95% CI lower bound > 0 | Live Eval Harness, randomized by user: matched persona vs the default persona currently used online (version pre-registered). ITT, read once at the pre-registered sample size, CI clustered by user. Guardrails: stop-request, complaint, block and report rates non-inferior. Read in W10, reviewed in W12. |

## Open questions

1. **R1's representation.** How many prototypes, how to estimate them stably from social signals, how they update during a conversation.
2. **Where persona acts.** As a condition of the Decision Model, as text style, or one shared representation for both.
3. **Short-term vs long-term.** Which personas win T1 but lose on T4 retention or repurchase. There is no direct study yet; simulators that model a user's latent state are a first tool.

## References

- Joachims et al. [Accurately Interpreting Clickthrough Data as Implicit Feedback](https://doi.org/10.1145/1076034.1076063). SIGIR 2005.
- Zhai et al. [Actions Speak Louder than Words: Trillion-Parameter Sequential Transducers for Generative Recommendations](https://arxiv.org/abs/2402.17152). ICML 2024.
- Tan et al. [Democratizing Large Language Models via Personalized Parameter-Efficient Fine-tuning](https://doi.org/10.18653/v1/2024.emnlp-main.372). EMNLP 2024.
- Chen et al. [Persona Vectors: Monitoring and Controlling Character Traits in Language Models](https://arxiv.org/abs/2507.21509). 2025.
- Wang, de Vries, Reinders. [Unifying User-based and Item-based Collaborative Filtering Approaches by Similarity Fusion](https://doi.org/10.1145/1148170.1148257). SIGIR 2006.
- Zhang et al. [Personalizing Dialogue Agents (PersonaChat)](https://arxiv.org/abs/1801.07243). ACL 2018.
- Salemi et al. [LaMP: When Large Language Models Meet Personalization](https://doi.org/10.18653/v1/2024.acl-long.399). ACL 2024.
