# Feature spec format

Everything under `docs/specs/` is an [OKF v0.2](https://github.com/GoogleCloudPlatform/knowledge-catalog/tree/main/okf) bundle whose root is `docs/specs/`.

## Layout

```
docs/specs/
├── index.md                  # bundle root index
├── orders/
│   ├── index.md              # domain index
│   ├── partial-refund.md     # one file per feature
│   └── order-cancellation.md
└── invoices/
    ├── index.md
    └── invoice-correction.md
```

- **Domain folders** are named after glossary concepts, in kebab-case. A new domain folder is only created after the user approves it.
- **Feature files** are named after the feature, in kebab-case. One feature per file.

## Spec file

```markdown
---
type: FeatureSpec
title: Partial refund
description: Customers can get money back for some of the items in a delivered order.
tags: [refunds, payments]
status: stable
lifecycle: draft
pending:
  refund_window: changed
  partial_refund_allowed: new
generated: { by: to-spec/<model-id>, at: 2026-10-30T21:00:00Z }
verified:
  - { by: "human:<user-id>", at: 2026-10-30T21:05:00Z }
---

# Purpose

The problem this feature solves, from the user's point of view, in two or three sentences.

# Actors

- **Customer**: a person who placed an order.
- **Support agent**: handles refund requests the system cannot decide.

# User stories

- As a customer, I want to refund a delivered order, so that I get my money back for items I don't want.
  Rules: refund_window, partial_refund_allowed

# Rules

## refund_window
A refund can be requested within 30 days of delivery.
- Requested 30 days after delivery → accepted
- Requested 31 days after delivery → rejected with "refund period expired"
- Order not yet delivered → not a refund; handled by [order cancellation](/orders/order-cancellation.md)

## partial_refund_allowed
A customer may refund some items of an order without refunding the rest.
- 2 of 3 items refunded → refund for those 2 items only; the order stays delivered

# Out of scope

- Refunds to a payment method other than the original one.

# Retired

- `gift_card_refund`: gift cards are no longer refundable.
```

### Sections

Purpose, Actors, User stories, Rules and Out of scope always appear, in this order. A section with nothing to say holds a single line, `None.` Retired is added at the end once the first rule is retired. There is no section for open questions.

- **Purpose**: the problem and why it matters, from the user's perspective. No solution design.
- **Actors**: every actor that appears in a user story or rule, using glossary terms.
- **User stories**: short and free of filler. Each one lists the rules it relies on on a `Rules:` line. Every rule appears in at least one story, and every actor has at least one story. Write a story only if it adds a who or a why that the rules don't already make obvious.
- **Rules**: see below.
- **Out of scope**: things deliberately excluded. Things the user refused belong here; they are often the most useful lines on the page.
- **Retired**: one line per removed rule: its name in backticks and why it was removed. Retired names are never reused.

### Rules

- A rule is a level-2 heading with its name, a single-sentence statement, and a bullet list of examples in the form `situation → outcome`.
- Examples are concrete: real numbers, real dates, real states. Each example should be usable as a test case as written. Include the boundary cases that were decided (the 30th and the 31st day, not just "within 30 days").
- A rule that depends on behaviour owned by another domain links to that domain's spec instead of restating it. Links are bundle-relative: `/orders/order-cancellation.md`.
- No technical content: no modules, tables, endpoints, file paths or code.

### Rule names

- `snake_case`, no prefix, no number. Unique within the domain folder, across all its feature files.
- A name describes the rule's **subject**, never its value: `refund_window`, not `refund_within_30_days`. When the value changes, the name stays true.
- Names are permanent. A rule keeps its name when its behaviour changes and when it moves to another feature file in the same domain.
- A rename happens only with the user's explicit approval, and it updates the spec, its user stories and the test titles that reference it in the same change.

### Test references

Tests refer to a rule by putting `[<domain>.<rule_name>]` at the start of the test title:

```ts
test("[orders.refund_window] rejects a refund requested 31 days after delivery", ...)
```

Inside specs, rules are referred to by their bare name. The domain qualifier is only needed outside the domain folder.

## Frontmatter

| Key | Value |
| --- | --- |
| `type` | Always `FeatureSpec` |
| `title` | Feature name, human-readable |
| `description` | One sentence. Repeated in the domain index |
| `tags` | Optional, short list |
| `status` | OKF meaning. `stable` once the user has confirmed the spec; `deprecated` when the feature is removed. Never `draft`, because this skill only writes after confirmation |
| `lifecycle` | Workflow state. See below |
| `pending` | Change queue for downstream skills. See below. Omitted when empty |
| `generated` | `{ by: to-spec/<model-id>, at: <ISO 8601> }`, updated on every write |
| `verified` | A one-element list with the user's confirmation: `{ by: "human:<id>", at: <ISO 8601> }`. Replaced on every write, never appended. Use the user's git `user.email` as the id if they have not given one |

### `lifecycle`

| Value | Meaning | Written by |
| --- | --- | --- |
| `draft` | Written or changed, architecture not yet extracted | `to-spec` |
| `architected` | Architecture has been extracted | the architecture skill |
| `ticketed` | Tickets have been created | the ticket skill |
| `implemented` | Built and shipped | decided when designing the downstream skills |
| `deprecated` | The feature was removed | `to-spec` |

`to-spec` only ever writes `draft` and `deprecated`:

- Creating a spec → `lifecycle: draft`, no `pending`.
- Updating a spec, whatever its current lifecycle → `lifecycle: draft`, and record the changes in `pending`.
- Removing a feature → `status: deprecated`, `lifecycle: deprecated`, `pending` removed. The file stays, and its rules stay as they are.
- Reviving a deprecated feature → treat it as an update and confirm with the user first.

### `pending`

A map from rule name to `new`, `changed` or `retired`. It tells downstream skills which rules to handle, since specs keep no history. The skill that sets `lifecycle: implemented` clears it.

- If the spec was created and has never left `draft` (no `pending` key), every rule is still new: keep it without `pending`.
- Otherwise, merge into the existing entries:

| Existing | This change | Result |
| --- | --- | --- |
| none | any | this change |
| `new` | `changed` | `new` |
| `new` | `retired` | `retired` |
| `changed` | `changed` | `changed` |
| `changed` | `retired` | `retired` |
| `retired` | revived | `changed` |

- A renamed rule appears under its new name. The rename itself is reported to the user, not recorded here.

## Index files

Index files carry no frontmatter, except the bundle root, which declares the OKF version.

`docs/specs/index.md`:

```markdown
---
okf_version: "0.2"
---

# Domains

* [Orders](orders/) - Placing, changing, cancelling and refunding orders
* [Invoices](invoices/) - Issuing and correcting invoices
```

`docs/specs/<domain>/index.md`:

```markdown
# Orders

* [Partial refund](partial-refund.md) - Customers can get money back for some of the items in a delivered order.
* [Order cancellation](order-cancellation.md) - Customers can cancel an order before it ships.
```

Each entry's description is the `description` from the spec's frontmatter. Indexes never list rules, so rule changes never touch them.
