# Architecture doc format

Everything under `docs/architecture/` is an [OKF v0.2](https://github.com/GoogleCloudPlatform/knowledge-catalog/tree/main/okf) bundle whose root is `docs/architecture/`.

## Layout

```
docs/architecture/
├── index.md                          # bundle root index: the module map's top level
├── inter-domain-communication.md     # decision (system-wide)
├── persistence.md                    # decision (system-wide)
├── orders/                           # same domain names as docs/specs/
│   ├── index.md
│   ├── refund-policy.md              # module
│   └── order-lifecycle.md            # module
├── invoices/
│   ├── index.md
│   └── invoice-ledger.md
└── shared/                           # modules that belong to no single domain
    ├── index.md
    └── payment-gateway.md
```

- **Domain folders** use the same names as `docs/specs/`.
- **`shared/`** holds modules that serve several domains and belong to none: a payment port, notification delivery.
- **Module files** are named after the module, in kebab-case. One deep module per file.
- **Decision files** sit at the bundle root, named after their subject, in kebab-case. No numbers, no `decisions/` folder. `grill-with-docs` also writes decision files here, in the same format with `generated.by` set to `grill-with-docs/<model-id>`.

## Module file

````markdown
---
type: Module
title: Refund policy
description: Decides whether a refund request is accepted and for how much.
tags: [refunds]
status: stable
lifecycle: architected
generated: { by: to-architecture/<model-id>, at: 2026-10-30T23:30:00Z }
verified:
  - { by: "human:<user-id>", at: 2026-10-30T23:40:00Z }
---

# Purpose

Owns every rule about whether, and how much, money goes back to a customer for a delivered order. Callers ask one question and get one decision; they never compute refund amounts themselves.

# Interface

```ts
evaluate(request: RefundRequest): RefundDecision

type RefundRequest = { orderId: OrderId; items: OrderItemId[]; requestedAt: Instant }
type RefundDecision =
  | { kind: "accepted"; amount: Money; items: OrderItemId[] }
  | { kind: "rejected"; reason: "refund_period_expired" | "not_delivered" | "already_refunded" }
```

- Pure decision: `evaluate` never moves money and never writes state. Executing the refund is the caller's job.
- Items already refunded are rejected as a whole request, never partially accepted.
- Never throws for a business outcome; every business outcome is a `RefundDecision`.

# Seam and dependencies

- **Seam:** `evaluate`. All refund rules are tested here.
- **Order history**: local-substitutable. Read through the order store; tests use the in-memory store.
- **Clock**: in-process. `requestedAt` is passed in, so no clock dependency.

# Data

Owns no entities. Reads orders and refunds owned by [order lifecycle](/orders/order-lifecycle.md).

# Rules owned

| Rule | Seam |
| --- | --- |
| `orders.refund_window` | `evaluate` |
| `orders.partial_refund_allowed` | `evaluate` |

# Left to implementation

- How order history is queried and cached.
- Rounding of partial amounts, as long as the sum of partial refunds never exceeds the amount paid.
````

### Sections

All six sections appear, in this order. A section with nothing to say holds a single line, `None.`

- **Purpose:** the module's responsibility in one or two sentences, plus what callers are spared. If the reason the module exists would surprise a reader of the code, say it here in one or two sentences.
- **Interface:** the external interface only. Types and signatures in the project's language, followed by every fact a caller must know: invariants, ordering constraints, error modes, performance expectations. No internal functions, no bodies.
- **Seam and dependencies:** where the module is tested, and each dependency with its category from the `codebase-design` skill and its adapters (production and test).
- **Data:** the entities the module owns, at a logical level (names, key fields, relationships). Entities owned elsewhere are linked, not described. No table definitions or migrations.
- **Rules owned:** every rule this module owns, as `<domain>.<rule_name>`, with the seam its examples are tested at. Every rule in `docs/specs/` that is not retired and not in a deprecated spec is owned by exactly one module.
- **Left to implementation:** decisions deliberately delegated to the implementer. Anything not listed here and not decided above is a gap.

### What never goes in

- File paths, directory structure, function bodies, internal helpers.
- History: no "new", "changed" or "previously". The doc holds the latest decision only. What changed is visible from the spec's `pending` map and the `lifecycle` fields.
- Rejected alternatives, unless someone would otherwise propose them again. Then one line: what and why not.

## Decision file

```markdown
---
type: Decision
title: Inter-domain communication
description: Domains talk to each other through domain events, never through direct calls.
status: stable
lifecycle: architected
generated: { by: to-architecture/<model-id>, at: 2026-10-30T23:30:00Z }
verified:
  - { by: "human:<user-id>", at: 2026-10-30T23:40:00Z }
---

Orders and Invoices are deployed and changed by different people, so a module in one domain never calls a module in another directly; it publishes a domain event and the other domain reacts. Synchronous calls were rejected because a failing Invoices deploy would block order placement.
```

A decision file is one to three paragraphs: the context, the decision, and why. When the why rests on a research file under `docs/research/`, link it in that sentence. Write one only for a **system-wide** decision that passes all three gates:

1. **Hard to reverse:** changing your mind later has a real cost.
2. **Surprising without context:** a future reader would wonder why it was done this way.
3. **The result of a real trade-off:** there were genuine alternatives, and this one was picked for specific reasons.

A decision that concerns a single module goes into that module's Purpose, not into a decision file.

## Frontmatter

Module and decision files share the same keys.

| Key | Value |
| --- | --- |
| `type` | `Module` or `Decision` |
| `title` | Human-readable name |
| `description` | One sentence, repeated in the index |
| `tags` | Optional |
| `status` | OKF meaning: `stable` once confirmed, `deprecated` when the module or decision should no longer exist |
| `lifecycle` | See below |
| `generated` | `{ by: to-architecture/<model-id>, at: <ISO 8601> }`, updated on every write |
| `verified` | A one-element list with the user's confirmation, replaced on every write |

### `lifecycle`

The same values as specs, so the ticket skill can treat both the same way:

| Value | Meaning | Written by |
| --- | --- | --- |
| `architected` | The doc changed and the code does not match it yet | `to-architecture` |
| `ticketed` | Tickets exist for the gap | the ticket skill |
| `implemented` | The code matches the doc at contract level | the step that verifies the implementation |

- Any edit to a module or decision file sets `lifecycle: architected`, whatever it was before.
- **Removing a module or decision:** do not delete the file. Set `status: deprecated` and `lifecycle: architected`, and state in Purpose what replaces it. The ticket skill reads this as "remove from the code". The step that sets `implemented` deletes the file once the code no longer contains it.

### Specs

On every in-scope spec, `to-architecture` sets `lifecycle: architected` and leaves `pending` untouched. A deprecated spec keeps `status: deprecated`, which tells the ticket skill to remove the feature. It never edits anything else in a spec.

## Index files

Index files have no frontmatter, except the bundle root, which declares the OKF version. Each entry's description is the `description` from the linked file's frontmatter.

`docs/architecture/index.md`:

```markdown
---
okf_version: "0.2"
---

# Domains

* [Orders](orders/) - Placing, changing, cancelling and refunding orders
* [Invoices](invoices/) - Issuing and correcting invoices
* [Shared](shared/) - Modules that serve several domains

# Decisions

* [Inter-domain communication](inter-domain-communication.md) - Domains talk to each other through domain events, never through direct calls.
* [Persistence](persistence.md) - One Postgres database, one schema per domain.
```

`docs/architecture/<domain>/index.md`:

```markdown
# Orders

* [Refund policy](refund-policy.md) - Decides whether a refund request is accepted and for how much.
* [Order lifecycle](order-lifecycle.md) - Owns an order's states and the transitions between them.
```

Together, the indexes are the module map.
