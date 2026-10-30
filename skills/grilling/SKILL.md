---
name: grilling
description: Grill the user relentlessly about a plan, decision or design, in rounds of numbered questions with recommended answers. Use when the user wants their thinking stress-tested, says "grill", or another skill needs decisions put to the user.
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Format a round like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

Word each question so "yes" accepts your recommended answer.

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

## Facts are yours, decisions are the user's

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent with a fresh context window to find it; don't ask the user for anything you could look up yourself. When the fact lives outside the working directory (third-party documentation, an API's limits, what a library or service supports), call the Skill tool with `research` instead. Don't block on either: a running lookup is an unsettled prerequisite, so only the questions downstream of it wait for the report; ask the rest of the frontier now.

The _decisions_ are the user's: put each to them and wait.

## Done

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding.

---

_Credit: adapted from [Matt Pocock](https://github.com/mattpocock)'s [`grilling`](https://github.com/mattpocock/skills/tree/main/skills/productivity/grilling) skill in [mattpocock/skills](https://github.com/mattpocock/skills) (commit `a496f55`), adding the route to `research`. Used under the MIT License: see [LICENSE](./LICENSE)._
