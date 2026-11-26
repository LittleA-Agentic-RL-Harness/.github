# Glossary · 术语表

[Wiki home](README.md)

| Term | 中文 | Definition |
|---|---|---|
| decision moment | 决策时刻 | A point where LittleA must choose its next action: a user message, an observable behaviour, or silence past a pre-registered length (24 h by default). One moment, one action. The unit of Ledger coverage and Eval sampling. |
| session | session | A continuous stretch of interaction between one user and one channel; the gap threshold between messages is pre-registered. The unit of outcome attribution. |
| six actions | 六个动作 | A engage (respond, no push) · B ask (clarify the need) · C recommend (a specific product) · D support (orders and after-sales) · E stop (stop pushing: wait, or advise not to buy now) · F defer (hand to the slow path). A closed set. |
| acceptable-action set | 可接受动作集合 | After adjudication, the set of actions acceptable at a moment (can be more than one), plus one preferred action. Offline primary metric: acceptable-action rate. |
| non-pushing moment | 不推进时刻 | A moment whose preferred action is E stop; two kinds, "wait" and "advise not to buy now". At least 60 of each. |
| T1–T4 · matured outcome | T1–T4 · matured outcome | T1 accept · T2 pay (a deal) · T3 fulfil · T4 retain. An outcome whose window has closed is matured; as a reward, matured outcome means T4. |
| censored · reversed | censored · reversed | Censored: the cutoff came before the window closed; neither success nor failure. Reversed: refunded or cancelled inside the window; T4 = 0. |
| π<sub>b</sub> | 行为策略执行概率 | The probability with which the behaviour policy actually executed an action (or persona), including pre-registered ε exploration. Not the model's calibrated p(a). The precondition for DR estimation. |
| compliance gate | 合规门禁 | Runtime filter: disclose AI identity in the first message where required; no pushing messages after a stop request; privacy isolation. 100% compliance. |
| stop request | 停止请求 | A user explicitly asks to stop contact, blocks, or unsubscribes. Enforced by the compliance gate; different from E stop. |
| no manipulation | 不操纵 | No invented scarcity or urgency, no hidden price or return terms, no impersonating a real person, no using expressed emotional vulnerability to push a purchase. A hard constraint. |
| OmniHub | OmniHub | Execution runtime: official messaging APIs, GUI agents on real phones and cloud sandboxes; every external action runs Intent → Attempt → Receipt. |
| Sensing | Sensing | Encodes a user's messages and behaviour into state z<sub>t</sub>; behaviour and state signals are the majority. |
| User Persona | User Persona | Structured per-user attributes: need, stage, taste, tone, pace, boundaries, each with confidence, evidence and as-of time. |
| Agent Persona · Persona Pool | Agent Persona · 人设库 | A reusable agent persona (role, voice, expertise depth, pacing, disclosure, boundaries) and the pool of them, with priors P(T1 given segment, persona) from real sessions. |
| Persona Matching | Persona Matching | Choosing, under disclosure and no-manipulation constraints, the Agent Persona with the highest expected matured outcome for each user; explores within top-k by π<sub>b</sub>. |
| Persona Vector Universe | 人格向量空间 | Personas embedded as vectors, the way language models embed text; 30K+ so far. |
| Decision Layer · Decision Model | Decision Layer · Decision Model | The layer that chooses actions, including fast/slow routing; the trained model inside it is an LM backbone with a decision head that outputs calibrated probabilities over six actions. |
| Teacher System | Teacher System | Frontier LLMs plus human review, with no limit on cost or latency; the reference the Student learns from. Frozen per distillation round. |
| off- / on-policy distillation | off/on-policy 蒸馏 | Learning first on the Teacher's trajectories, then on states the Student itself reaches, with the Teacher reviewing and labelling every action. |
| Sensing–Action Model | 感知与行动模型 | The target model: native behaviour in, native actions out. Research direction 2. |
| Judge | Judge | Evaluates only on the sealed holdout and matured outcomes; never the trained model; tested separately for persuasion and sycophancy. |
| Evidence Ledger | 证据账本 | Per decision moment: p(a), π<sub>b</sub>, as-of state hash, component versions, receipt, T1–T4 status. Coverage ≥ 99%. |
| Eval Set · holdout | 评测集 · holdout | Decision Eval, Persona Eval and Outcome-linked Real Cases, plus the Simulation Track and Track S; the sealed part is the holdout, held by the Eval Steward. |
| Track S | Track S | Social-intelligence pilot: models and human reference groups answer the same tasks; reported as percentiles of the human distribution. |
| Live Eval Harness | Live Eval Harness | The online randomized-experiment facility shared by O1–O3: randomized by user, pre-registered, guardrail-monitored. |
| Eval Steward | Eval Steward | The role independent of training that holds the holdout, recomputes acceptance results and audits the Ledger. |
| RSI | 递归自我改进 | Recursive self-improvement. |
| trust tax | 信任税 | Creator commissions, traffic ads, ad bids and platform take rate: what brands pay to borrow trust they do not have. |
| ROT | 代币回报率 | Return on token expenses: revenue ÷ token spend. |
| zero-friction matching network | 零摩擦匹配网络 | Matching people to supply by intent, not by bid. |
| coexist with human beings | 与人类共生 | LittleA's user-side promise. |
| Bonding · Self-evolution | Bonding · Self-evolution | Relationship strategy on a scale of months to years; generation-over-generation improvement accepted on randomized matured outcomes. Long-term research directions. |
| A2C · A2A · A3A | A2C · A2A · A3A | Personal trust entry → agent-to-agent commerce → AI-native, decentralized commerce network with trust as the currency. |
