---
name: grill-with-docs
description: A relentless interview to sharpen a plan or design, which also creates docs (glossary and decision files) as we go.
disable-model-invocation: true
---

Call the Skill tool with `grilling` and run its interview. As you go, build the project's domain model and write it down the moment terms and decisions crystallise: see **Domain modeling** below.

Updating `GLOSSARY.md` and writing agreed decision files during the session is part of the interview, not acting on it: do those as you go.

## Domain modeling

Actively build and sharpen the project's domain model as you grill. This is the *active* discipline: challenging terms, inventing edge-case scenarios, and writing the glossary and decisions down the moment they crystallise.

### File structure

Most repos have a single context:

```
/
├── GLOSSARY.md
├── docs/
│   └── architecture/
│       ├── index.md
│       ├── event-sourced-orders.md
│       └── postgres-for-write-model.md
└── src/
```

If a `GLOSSARY-MAP.md` exists at the root, the repo has multiple contexts. The map points to where each one lives:

```
/
├── GLOSSARY-MAP.md
├── docs/
│   └── architecture/                 ← system-wide decisions
├── src/
│   ├── ordering/
│   │   └── GLOSSARY.md
│   └── billing/
│       └── GLOSSARY.md
```

Create files lazily: only when you have something to write. If no `GLOSSARY.md` exists, create one when the first term is resolved. If no `docs/architecture/index.md` exists, create it when the first decision file is needed.

### During the session

#### Challenge against the glossary

When the user uses a term that conflicts with the existing language in `GLOSSARY.md`, call it out immediately. "Your glossary defines 'cancellation' as X, but you seem to mean Y. Which is it?"

#### Sharpen fuzzy language

When the user uses vague or overloaded terms, propose a precise canonical term. "You're saying 'account': do you mean the Customer or the User? Those are different things."

#### Discuss concrete scenarios

When domain relationships are being discussed, stress-test them with specific scenarios. Invent scenarios that probe edge cases and force the user to be precise about the boundaries between concepts.

#### Cross-reference with code

When the user states how something works, check whether the code agrees. If you find a contradiction, surface it: "Your code cancels entire Orders, but you just said partial cancellation is possible. Which is right?"

#### Update GLOSSARY.md inline

When a term is resolved, update `GLOSSARY.md` right there. Don't batch these up: capture them as they happen. Use the format in [GLOSSARY-FORMAT.md](./GLOSSARY-FORMAT.md).

`GLOSSARY.md` should be totally devoid of implementation details. Do not treat `GLOSSARY.md` as a spec, a scratch pad, or a repository for implementation decisions. It is a glossary and nothing else.

#### Offer decision files sparingly

Only offer to create a decision file for a **system-wide** decision, and only when all three are true:

1. **Hard to reverse**: the cost of changing your mind later is meaningful
2. **Surprising without context**: a future reader will wonder "why did they do it this way?"
3. **The result of a real trade-off**: there were genuine alternatives and you picked one for specific reasons

If any of the three is missing, skip the decision file. A decision that concerns a single module is left for `to-architecture`, which records it in that module's doc. Use the format in [DECISION-FORMAT.md](./DECISION-FORMAT.md).

---

_Credit: this skill is by [Matt Pocock](https://github.com/mattpocock), from [mattpocock/skills](https://github.com/mattpocock/skills) (commit `a496f55`). Upstream, `grill-with-docs` delegates to his [`grilling`](https://github.com/mattpocock/skills/tree/main/skills/productivity/grilling) and [`domain-modeling`](https://github.com/mattpocock/skills/tree/main/skills/engineering/domain-modeling) skills; here it delegates to this repo's `grilling` skill and inlines `domain-modeling`, with small wording changes so it reads as one skill. Decisions are written as decision files under `docs/architecture/` instead of ADRs under `docs/adr/`, so `DECISION-FORMAT.md` is adapted from his `ADR-FORMAT.md`. `GLOSSARY-FORMAT.md` is copied verbatim. Used under the MIT License: see [LICENSE](./LICENSE)._
