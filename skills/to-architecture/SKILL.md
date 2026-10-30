---
name: to-architecture
description: "Design the architecture for feature specs in draft (or for an architecture-only change): deep modules, interfaces, seams, dependencies and rule ownership, written as living target-state docs under docs/architecture/."
disable-model-invocation: true
---

# To Architecture

Decide the **shape** of the system for a feature: which modules own which rules, what each module's interface is, where its seam sits, and how it reaches its dependencies. Write it down as living docs under `docs/architecture/`, so that later sessions can cut tickets and implement without guessing at any of it.

Three artifacts, three roles:

- **Specs** (`docs/specs/`) say what the system must do. This skill reads them and never edits their content.
- **Architecture docs** (`docs/architecture/`) hold the **target state**: how the system should be shaped, at contract level. They are never versioned and never deleted when work ships; they always hold the latest decision.
- **The code** is the **current state**. The gap between the architecture docs and the code is what the ticket skill turns into tickets.

Division of labour:

| Step | Decides | Never decides |
| --- | --- | --- |
| This skill | Modules, interfaces (types, invariants, ordering, errors), seams, dependencies and adapters, data ownership, rule ownership | Order of work, slicing |
| Ticket skill | Order, vertical slices, blockers, prefactoring (from the gap between docs and code) | Anything architectural. If it finds a gap, it stops and asks |
| Implementer | Everything behind an interface | Interfaces, seams, data ownership |

Call the Skill tool with `codebase-design` before designing, and read [FORMAT.md](FORMAT.md) before writing.

## Process

### 1. Choose the scope

- If the user named a feature, domain or module, take it.
- Otherwise, list the specs with `lifecycle: draft` (with their `pending` rules) and those with `lifecycle: deprecated`, and ask which to take. Default: one feature per session, to stay in the smart zone. Take several only if the user asks and they are small.
- The skill can also run with no spec change at all, for an architecture-only change ("merge these two modules", "introduce a port for payments"). Then start from the named domain or module.

### 2. Gather

1. Read the in-scope spec(s), `docs/specs/index.md` and the architecture docs of every domain the spec touches, plus `docs/architecture/index.md` and the decision files it lists.
2. If the spec has a `pending` map, those rules start the work. Otherwise every rule in the spec is in scope. In a deprecated spec (`status: deprecated`), every rule counts as retired: the feature was removed.
3. Dispatch a sub-agent with a fresh context window to explore the code and report back concisely:
   - how the in-scope rules are implemented today, if at all: which code owns them, where they are tested;
   - for every module doc with `lifecycle: implemented` in the touched domains, whether the code contradicts its interface, seam or rule ownership;
   - candidates for duplication, modules that should merge, or a missing abstraction, across the touched domains and the specs and modules next to them.

Facts are yours to find, never the user's. Use sub-agents for any further lookups so the main context stays small, and the `research` skill for facts outside the repo (a third-party API, a library, a hosted service).

### 3. Grill the design

Call the Skill tool with `grilling` and walk the design tree with the user.

The tree covers, for the work in scope:

- **Ownership:** which module owns each rule. Every rule is owned by exactly one module.
- **Depth:** does each module put a lot of behaviour behind a small interface? Apply the deletion test.
- **Interface:** types, invariants, ordering constraints, error modes.
- **Seam:** where each rule's examples will be tested. Prefer existing seams and the highest seam possible.
- **Dependencies:** the category of each dependency and the adapters it needs.
- **Data:** which module owns which entities.
- **Left to implementation:** what is deliberately delegated.

When there are genuinely different ways to shape something, lay out the two or three real alternatives in their own question, grounded in what the code does today, say what each trades off, and recommend one. When the fork is a module's interface, design it twice instead, following DESIGN-IT-TWICE.md from the `codebase-design` skill.

Also put to the user, each as its own item:

- every change your exploration suggests to a module **outside** the feature (a merge, an extracted abstraction, removed duplication);
- every contradiction between the code and an `implemented` module doc: which one is right?
- every **decision file** you propose. Propose one only for a system-wide decision that passes all three gates in FORMAT.md.

The session is done when nothing is left to decide. Do not write until the user confirms the shared understanding.

### 4. Check rule coverage

Build the coverage table for every touched domain: each non-retired rule → owning module → seam. Every rule must appear exactly once. Rules marked `retired` in the spec, and every rule of a deprecated spec, are removed from their module's Rules owned. Do not continue while any rule is unowned or owned twice; take the gap back to the user.

### 5. Write

Write or update the module docs, decision files and indexes following FORMAT.md. Edit existing docs in place; they hold only the latest decision. A module that should no longer exist is marked for removal as described in FORMAT.md, not deleted.

### 6. Builder check

Dispatch a fresh sub-agent with this prompt, filling in the paths:

```
You are an experienced software engineer. You will cut implementation
tickets from these documents and then implement them, without being able to
talk to their author. Read: [SPEC PATHS], [MODULE DOC PATHS],
[DECISION FILE PATHS]. You may read the code.

List only the questions you would still have to guess at before you could
cut tickets and implement: places where a wrong guess about a module, an
interface, a seam, a dependency or data ownership would matter. Ignore
anything the documents explicitly leave to implementation, and anything
you could reasonably decide yourself behind an interface. For each question
say why the answer matters. Rank by importance. If nothing important is
left, say "nothing important left".
```

Sort its questions into three piles:

- **Already answered** by the conversation or the code: put the answer into the docs.
- **Minor:** a sensible implementer could settle it behind an interface. Add it to the module's Left to implementation section.
- **The user's:** only the user can answer it and it changes the design. Ask all of these in one message, most important first, and update the docs with the answers.

### 7. Confirm and set state

Show the user the files written and the coverage table, and ask for confirmation. Then, following FORMAT.md:

- set `lifecycle: architected`, `generated` and `verified` on every module and decision file you created or changed;
- set `lifecycle: architected` on every in-scope spec, leaving its `pending` map in place for the ticket skill;
- update the indexes.

Do not touch spec content, tickets or code.

### 8. Report

List the files written, the decisions made outside the feature, the contradictions resolved, and the choices you made from the builder check's minor pile. The next step is the ticket skill.

## Rules of thumb

- Write only what the code cannot tell a reader: responsibility, contracts, seams, ownership, and reasons that would surprise. Never file paths, never function bodies.
- Every module doc must survive a full rewrite of its implementation unchanged. If a sentence would not, it describes the implementation, not the architecture.
- Mention a rejected alternative only when someone would otherwise propose it again.
- Use the glossary's domain terms for names, and the `codebase-design` vocabulary for structure.
