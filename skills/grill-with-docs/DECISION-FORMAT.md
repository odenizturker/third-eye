# Decision File Format

Decision files live at the root of `docs/architecture/`, named after their subject in kebab-case: `docs/architecture/inter-domain-communication.md`. No numbers, no `decisions/` folder. `docs/architecture/` is an [OKF v0.2](https://github.com/GoogleCloudPlatform/knowledge-catalog/tree/main/okf) bundle shared with the `to-architecture` skill, which writes module docs and decision files in the same place.

Create `docs/architecture/` and its `index.md` lazily: only when the first decision file is needed.

## Template

```md
---
type: Decision
title: Inter-domain communication
description: Domains talk to each other through domain events, never through direct calls.
status: stable
lifecycle: architected
generated: { by: grill-with-docs/<model-id>, at: 2026-10-30T23:30:00Z }
verified:
  - { by: "human:<user-id>", at: 2026-10-30T23:40:00Z }
---

{1-3 paragraphs: what's the context, what did we decide, and why.}
```

That's it. A decision file can be a single paragraph. The value is in recording *that* a decision was made and *why*, not in filling out sections. Mention a rejected alternative only when someone would otherwise propose it again: one line, what and why not.

## Frontmatter

| Key | Value |
| --- | --- |
| `type` | Always `Decision` |
| `title` | Human-readable name |
| `description` | One sentence, repeated in the index |
| `tags` | Optional |
| `status` | `stable` once the user has confirmed it; `deprecated` when the decision should no longer hold |
| `lifecycle` | `architected` on every write: the decision is recorded and the code may not match it yet |
| `generated` | `{ by: grill-with-docs/<model-id>, at: <ISO 8601> }`, updated on every write |
| `verified` | A one-element list with the user's confirmation: `{ by: "human:<id>", at: <ISO 8601> }`. Replaced on every write, never appended. Use the user's git `user.email` as the id if they have not given one |

When a decision is reversed, don't delete its file: set `status: deprecated` and `lifecycle: architected`, and say what replaces it.

## Index

List every decision file under `# Decisions` in `docs/architecture/index.md`, with the `description` from its frontmatter. Leave the rest of the index as it is. If the index doesn't exist yet, create it:

```md
---
okf_version: "0.2"
---

# Decisions

* [Inter-domain communication](inter-domain-communication.md) - Domains talk to each other through domain events, never through direct calls.
```

## When to offer a decision file

Only for a **system-wide** decision. A decision that concerns a single module belongs in that module's doc, which `to-architecture` writes.

All three of these must also be true:

1. **Hard to reverse**: the cost of changing your mind later is meaningful
2. **Surprising without context**: a future reader will look at the code and wonder "why on earth did they do it this way?"
3. **The result of a real trade-off**: there were genuine alternatives and you picked one for specific reasons

If a decision is easy to reverse, skip it: you'll just reverse it. If it's not surprising, nobody will wonder why. If there was no real alternative, there's nothing to record beyond "we did the obvious thing."

### What qualifies

- **Architectural shape.** "We're using a monorepo." "The write model is event-sourced, the read model is projected into Postgres."
- **Integration patterns between contexts.** "Ordering and Billing communicate via domain events, not synchronous HTTP."
- **Technology choices that carry lock-in.** Database, message bus, auth provider, deployment target. Not every library: just the ones that would take a quarter to swap out.
- **Boundary and scope decisions.** "Customer data is owned by the Customer context; other contexts reference it by ID only." The explicit no-s are as valuable as the yes-s.
- **Deliberate deviations from the obvious path.** "We're using manual SQL instead of an ORM because X." Anything where a reasonable reader would assume the opposite. These stop the next engineer from "fixing" something that was deliberate.
- **Constraints not visible in the code.** "We can't use AWS because of compliance requirements." "Response times must be under 200ms because of the partner API contract."
- **Rejected alternatives when the rejection is non-obvious.** If you considered GraphQL and picked REST for subtle reasons, record it; otherwise someone will suggest GraphQL again in six months.
