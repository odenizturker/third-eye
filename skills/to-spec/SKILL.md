---
name: to-spec
description: "Turn the decisions from the current grilling session into feature specs under docs/specs/<domain>/: business rules with examples, user stories, no technical decisions. Creates new specs or updates existing ones in place."
disable-model-invocation: true
---

# To Spec

Turn the decisions made in this conversation (usually a `grill-with-docs` session) into **feature specs**: what the system must do and why, in the project's own domain language. This skill records decisions. It does not make new ones.

A feature spec contains **no technical decisions**: no modules, interfaces, schemas, endpoints, file paths or code. Architecture and tickets are extracted later, in separate sessions, by other skills. If the conversation settled technical questions, leave them out; the architecture session grills them again.

Specs are living documents. Each one always holds the latest truth about a feature's behaviour and is never versioned: when behaviour changes, the existing spec is edited in place.

The exact shape of every file this skill writes is in [FORMAT.md](FORMAT.md). Read it before writing.

## Preconditions

- The conversation must contain resolved decisions. If there was no grilling session in this conversation, do not write anything: tell the user to run `grill-with-docs` first.
- Read `GLOSSARY.md` (or `GLOSSARY-MAP.md` and the glossaries it points to) and the decision files listed in `docs/architecture/index.md`, if they exist. Use glossary terms everywhere. Never create or edit the glossary; that is `grill-with-docs`'s job.

## Process

### 1. Locate

1. Identify every feature and domain the conversation touched. A **domain** is a top-level concept from the glossary (Order, Invoice) and maps to a folder `docs/specs/<domain>/`. If there is no glossary, propose domains and let the user confirm them.
2. Each rule lives in exactly one domain: the one that owns the behaviour. Other domains link to it rather than restating it.
3. Read `docs/specs/index.md`, the `index.md` of every touched domain, and **every** spec file in those domains. Rule names are unique per domain, and a conflict can hide in a different feature file than the one you are editing.

### 2. Extract and classify

For every rule the conversation settled, classify it against what already exists:

| Class | Meaning |
| --- | --- |
| `new` | No existing rule covers it |
| `changed` | An existing rule, same subject, different behaviour |
| `unchanged` | Already captured exactly |
| `conflicting` | The conversation contradicts an existing rule the user may not have had in mind. Never resolve this yourself: ask |

Also collect rules the conversation removed (to be **retired**) and features it removed (to be **deprecated**).

Name new rules following the naming rules in FORMAT.md. Never rename an existing rule silently. If an existing name no longer describes its subject, propose the rename as its own item.

For every `changed`, retired or renamed rule, search the test suite for its reference `[<domain>.<rule_name>]` and collect the matching tests.

Find the **gaps**: behaviour the conversation did not settle (an example with no agreed outcome, an edge case nobody decided, an ambiguous term). Facts you can look up (existing specs, code, glossary) are yours to find; never ask the user for them. Decisions are the user's.

### 3. Present one round

Send a single message containing:

- Each file to write: its path, whether it is created or updated, and its `lifecycle` transition.
- Each rule: its class, its name and its one-sentence statement. For `conflicting` and `changed` rules, show the current text next to the new text.
- New domain folders and rule renames, each as an item needing approval.
- The tests that reference each `changed`, retired or renamed rule.
- A warning for every spec whose `lifecycle` is `architected` or `ticketed`: work based on it is in flight, these rules changed, and existing tickets may be out of date. Tickets are not touched by this skill.
- The gaps, as numbered questions, each with your recommended answer, worded so that "yes" accepts the recommendation.

Then wait. If the user defers a gap, leave that rule out of the spec entirely and say explicitly which rules were left out. Ask another round only if the answers open new gaps.

### 4. Write

Only after the user confirms:

1. Create or update the spec files following FORMAT.md. When updating, edit rules in place, leave unchanged rules as they are, move retired rules to the Retired section and fix any user story that referenced a changed or retired rule.
2. Set the frontmatter (`status`, `lifecycle`, `pending`, `generated`, `verified`) following the rules in FORMAT.md.
3. For an approved rename, update the rule, every user story that references it, and every test title containing the old reference.
4. Create or update the index files.

Do not touch anything else: no tickets, architecture documents, decision files, glossary or code. The only code change this skill ever makes is renaming references in test titles after an approved rename.

### 5. Report

List the files written, the rules left out because of deferred gaps, and the tests the user should revisit. The next step for every spec now in `draft` is an architecture session.

## Rules of thumb

- Every sentence must trace back to a decision the user made. Anything invented to fill a section is a defect.
- Technical-content test: would this sentence still be true if the implementation were rewritten from scratch? If not, it does not belong in a spec.
- Examples must be concrete enough to become test cases as written.
- There is no "open questions" section. Gaps are asked, never written down.
