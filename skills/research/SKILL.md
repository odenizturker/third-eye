---
name: research
description: Investigate a question against primary sources in a background agent and save the cited findings under docs/research/. Use when a decision depends on a fact outside the repo (third-party docs, an API's limits, what a library or service supports), or when the user wants reading legwork delegated.
---

Spin up a **background agent** with a fresh context window to do the research, so the session keeps going while it reads. Only the questions whose answer depends on the finding wait for it.

Its job:

1. Investigate the question against **primary sources**: official docs, source code, specs, first-party APIs, not a secondary write-up of them. Follow every claim back to the source that owns it.
2. Write the findings to `docs/research/<slug>.md`, named after the question in kebab-case, in the format below. Every claim carries its source.
3. Report back the path and the short answer.

A research file is **source material**, not a living doc: once written, it stays as it is. When the facts change or a new question comes up, write a new file. A decision file whose reasoning rests on a finding links to the research file.

## Format

```md
---
title: <short name of the question>
question: <the question, in one sentence>
researched: <ISO 8601 date>
---

# Answer

<the short answer> ([<source title>](<url or repo path>))

# Findings

- <claim> ([<source title>](<url or repo path>))
- <claim> ([<source title>](<url or repo path>))
```

- **Answer:** the short answer to the question, as a decision-maker needs it, cited.
- **Findings:** the supporting claims, one per line, each cited. Note where sources disagree, and which one owns the fact.

---

_Credit: adapted from [Matt Pocock](https://github.com/mattpocock)'s [`research`](https://github.com/mattpocock/skills/tree/main/skills/engineering/research) skill in [mattpocock/skills](https://github.com/mattpocock/skills) (commit `a496f55`), with a fixed location and format for the findings. Used under the MIT License: see [LICENSE](./LICENSE)._
