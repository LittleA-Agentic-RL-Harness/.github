# 06 · RSI and roadmap

[Wiki home](README.md) · previous: [Evidence and evaluation](05-evidence-and-eval.md) · see also: [Glossary](glossary.md)

## Recursive self-improvement

LittleA is an RSI (recursive self-improvement) agent architecture whose end goal is hyper-individualized, long-term trust. It is evolved, not designed. Three pillars, and where each stands in engineering today:

| Pillar | Idea | Today |
|---|---|---|
| **Persona Vector Universe** | Language models embed text in a vector space; we embed personas. Each LittleA's identity is distilled from the person, or group, that best matches you. | 30K+ personas created. Phase 1 matching runs on R0 Persona Cards and a 6–10 persona Pool v0; vector representations are R1 and R2. |
| **Sensing–Action Model** | Native actions in, native actions out: sense behaviour and state in the social world and answer with actions, not text. | Phase 1 Decision Model is an LM backbone with a decision head. Pretraining on behaviour sequences is research direction 2. |
| **Darwinian self-evolution** | Like a species with a collective memory: the more it interacts with people, the more experience flows back, and each new generation understands people better. | Versions branch as Mother / Child: 100+ generations so far. Making the loop self-driving is research direction 4. |

**How a generation is accepted.** In an environment that can be reset, an improvement can be accepted by replaying the same task. Real users cannot be replayed. So a new generation is accepted only on **matured outcomes under random assignment**, read through the Evidence Ledger and the Live Eval Harness, never on the system's own judgement and never on simulated users.

## Why Phase 1 runs in this order

1. **Eval Set first.** Every later "better" is measured on the same ruler, so it can be separated from "we changed the base model" or "we changed the sample".
2. **Persona Matching next.** In short sessions, the choice of Agent Persona is likely the controllable variable with the largest effect on T1; KR 2.3 tests that claim.
3. **Distil the Decision Model last.** Distillation is mature; what needs proof is that it works for action choice in real transactions, in one category first.

## Workstreams

Each workstream has a lead and a reviewer; the reviewer does not train on that line.

| Workstream | Owns | Phase 1 delivers |
|---|---|---|
| **WS-E** · Eval and Evidence | O1: Eval Set, Evidence Ledger, Live Eval Harness, Track S; includes the Eval Steward | Eval Set v1 (sealed W6), Ledger contract, Harness (W4), validity report, Track S pilot |
| **WS-P** · Persona | O2: User Persona, Agent Persona, Persona Pool, Matcher, behaviour features for R0 | R0 online, Persona Pool v0, A/B report, R1 research design |
| **WS-D** · Decision | O3: Teacher System, Student, fast/slow routing | Teacher, trajectory dataset, Students S and M, Gate 2 comparison |
| **WS-X** · Platform | OmniHub, shared with the commercial side; no research KR | Ledger fields, Harness execution environment, run stability |
| **WS-R** · Exploration | the five long-term directions, including the native Sensing–Action line | one research design per direction, reviewed in W12 |

## Five research directions

No dates and no KRs. All five share the Evidence Ledger, Eval Set and Live Eval Harness built in Phase 1, and unfold as results arrive. The binding constraints are first-party data users agree to share and matured samples; compute is added in steps as results justify it.

~~~mermaid
mindmap
  root((LittleA RSI))
    Environment and scale
      real users as a long-term learning signal
      agent cloud: many agents, each iterating
    Native Sensing–Action
      pretrain on behaviour sequences
      high-frequency real-time decisions for live video
    Native persona
      Persona Card to blurry persona to adapter
      one shared persona embedding space
    Self-evolution
      generations that improve without drift
      accept by randomized matured outcome, not replay
    Social interaction model and bonding
      predict what a person does after an action
      relationship strategy over months and years
~~~

| # | Direction | Question | Builds on |
|:---:|---|---|---|
| 1 | **Environment and scale** | With AI identity disclosed and only compliant channels, can real users supply enough learning signal over the long run? When many agents iterate on shared compute in an agent cloud, does measurable division of labour emerge? | scaling the Live Eval Harness |
| 2 | **Native Sensing–Action Model** | Can a model pretrained mainly on behaviour sequences (likes, posts, reply timing, silence) that outputs only actions, paired with a generator, beat "language model + decision head"? Can it make the high-frequency decisions live video needs? | O3's Student and routing; behaviour sequences in the Ledger |
| 3 | **Native persona** | From Persona Card to blurry persona to persona adapter: should all users' personas share one embedding space, so priors improve as data grows? | O2; Persona Eval as the shared yardstick |
| 4 | **Self-evolution** | Once the pipeline is stable, can the system improve generation after generation in an environment that cannot be reset, without drifting? Can acceptance move from replay to randomized matured outcomes? | O1's Ledger and Harness as the acceptance test; O3's Decision Model and case memory as what changes |
| 5 | **Social interaction model and bonding** | Can we predict how someone responds to an action, and learn relationship strategies over months and years? When are short-term "losses" such as advising not to buy, or not replying for now, the long-term optimum? | O2's persona representation, O3's Student; Track S for the human comparison |

## The endgame

| Stage | Form |
|---|---|
| **A2C** · personal trust entry | everyone has a LittleA they trust |
| **A2A** · agent-to-agent commerce | user agents and supply agents discover each other, negotiate and close |
| **A3A** · AI-native commerce network | fully decentralized; trust becomes the currency |

## References

- Kreps et al. [Rational Cooperation in the Finitely Repeated Prisoners' Dilemma](https://doi.org/10.1016/0022-0531(82)90029-1). 1982.
- Berg, Dickhaut, McCabe. [Trust, Reciprocity, and Social History](https://doi.org/10.1006/game.1995.1027). 1995.
- Mayer, Davis, Schoorman. [An Integrative Model of Organizational Trust](https://doi.org/10.2307/258792). 1995.
- Zhou et al. [Memento: Fine-tuning LLM Agents without Fine-tuning LLMs](https://arxiv.org/abs/2508.16153). 2025.
- Baker et al. [Emergent Tool Use From Multi-Agent Autocurricula](https://arxiv.org/abs/1909.07528). ICLR 2020.
- Park et al. [Generative Agents](https://arxiv.org/abs/2304.03442). UIST 2023.
- Zhou et al. [Social World Models](https://arxiv.org/abs/2509.00559). 2025.
- Brohan et al. [RT-2](https://arxiv.org/abs/2307.15818). 2023 · Black et al. [π0](https://arxiv.org/abs/2410.24164). RSS 2025.
