# 02 · OmniHub

[Wiki home](README.md) · previous: [Architecture](01-architecture.md) · next: [Persona Layer](03-persona.md)

OmniHub is the execution runtime. It turns a chosen action into an external effect and returns a receipt. It is shared with the commercial side (workstream WS-X) and carries no research KR of its own; what it owes research is clean, complete evidence.

| Surface | How |
|---|---|
| Official messaging APIs | the platform's own business and messaging endpoints |
| GUI agents on real phones | LittleA's own devices, driven over ADB, using apps the way people do |
| Cloud sandboxes | the same GUI agents on virtual devices, for scale and isolation |
| Live video | planned; needs higher-frequency real-time decisions than Phase 1 targets |

Inside OmniHub, actions are written in a typed **Action DSL** and checked by a **verifier**. Navigation, budgets and accounting are deterministic programs, not model calls.

## The compliance gate

Every outbound action passes the gate first. The gate is a runtime filter, not a model preference.

| Rule | Behaviour |
|---|---|
| AI disclosure | Where law or platform rules require it, the first message discloses that LittleA is an AI. |
| Stop requests | Once a user clearly asks to stop contact, blocks, or unsubscribes, no further pushing messages are sent. Recognition rules and example phrases are registered in the spec. |
| Privacy isolation | Each person's data stays isolated. |

Both disclosure and stop-request handling are audited on the full log, with a 100% compliance target. A user's stop request is different from the agent's own **E stop** decision.

## Intent → Attempt → Receipt

~~~mermaid
stateDiagram-v2
    [*] --> Intent: typed action with its ID
    Intent --> Attempt: gate passed
    Intent --> Refused: gate refused
    Attempt --> CONFIRMED
    Attempt --> REJECTED
    Attempt --> UNKNOWN: timeout, lost response
    UNKNOWN --> Reconcile: read-only check, same ID
    Reconcile --> CONFIRMED
    Reconcile --> REJECTED
    CONFIRMED --> [*]
    REJECTED --> [*]
    Refused --> [*]
~~~

- **Exactly-once effects.** An UNKNOWN result is reconciled read-only under the same ID. It is never resent under a new ID, so a lost response cannot become a duplicate message or a double charge.
- **Receipts are events.** CONFIRMED, REJECTED and UNKNOWN flow back into the event stream, so Sensing sees what actually happened, not what was intended.
- **Closure.** Receipt closure for external write actions must be 100% (KR 1.2).

## What OmniHub writes to the Ledger

| Field | Content |
|---|---|
| decision_id | the decision moment this action belongs to |
| actions before and after the gate | what the policy chose, and what was actually allowed |
| Intent / Attempt / Receipt IDs | the full execution trail |
| component versions | policy, persona, gate, templates, models |
| cost counts | counts of known and unknown costs |

## Platform constraints we design for

The rules of each channel are moving, and OmniHub treats them as hard inputs.

| Platform | Constraint |
|---|---|
| Instagram | the messaging API can only reply to conversations the user started ([docs](https://developers.facebook.com/docs/instagram-platform/instagram-api-with-instagram-login/messaging-api)) |
| WhatsApp | Meta runs its own Business Agent, and its messages are billed per token from 2026-08-01 ([pricing](https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing/non-template-messages/)) |
| WeChat personal accounts | unauthorized automation is prohibited from 2026-10-15 ([rules](https://weixin.qq.com/agreement/personal_account)) |
| EU | AI Act Article 50 transparency obligations apply from 2026-08-02 ([text](https://artificialintelligenceact.eu/article/50/)) |

## Repositories

| Repository | Role |
|---|---|
| <b>littlea-agent-infra</b> | canonical harness monorepo: Coreline, Gateway, device fleet, Inside Harvest, OmniHub, contracts, evidence |
| <b>littlea-android-agent-harness</b> | auditable Android harness: multi-device ADB, GUI-Owl proposal-based inference, exactly-once effects, Device Console |
