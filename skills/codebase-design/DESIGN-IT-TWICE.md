# Design it twice

Your first idea for an interface is unlikely to be the best one (Ousterhout). When a module's interface could take genuinely different shapes, have several sub-agents design it in parallel, each pulled in a different direction, then compare. Skip this when there is one obvious shape: it is for real forks, not for every module.

Uses the vocabulary in [SKILL.md](SKILL.md): **module**, **interface**, **depth**, **seam**, **adapter**, **leverage**, **locality**.

## 1. Frame the problem space

Write a short, user-facing explanation of what the interface has to satisfy:

- the behaviour it must carry: in this repo's flow, the spec rules the module will own, by `<domain>.<rule_name>`;
- the constraints any interface must meet;
- each dependency and its category (see [SKILL.md](SKILL.md));
- a rough code sketch that makes the constraints concrete. A sketch, not a proposal.

Show this to the user, then go straight on to step 2: they read while the sub-agents work.

## 2. Spawn the designers

Spawn three or more sub-agents in parallel, each with a fresh context window. Give each the same technical brief (the rules and their examples, the relevant code paths, neighbouring module docs, the dependency categories, the [SKILL.md](SKILL.md) vocabulary and the `GLOSSARY.md` terms) and a different design constraint:

- **Designer 1:** minimise the interface: one to three entry points, maximum leverage per entry point.
- **Designer 2:** maximise flexibility: support many use cases and extension.
- **Designer 3:** optimise for the most common caller: make the default case trivial.
- **Designer 4** (when the module has remote or external dependencies): design around ports and adapters at the seam.

Each designer returns:

1. **Interface:** types and signatures, plus invariants, ordering constraints and error modes. When the result is headed for a module doc, in the shape of that doc's Interface section.
2. **Usage:** how a caller uses it.
3. **Behind the seam:** what the implementation hides.
4. **Seam and dependencies:** where it is tested, each dependency's category and its adapters.
5. **Trade-offs:** where leverage is high and where it is thin.

## 3. Compare and recommend

Present the designs one after another, then compare them in prose by **depth**, **locality** and **seam placement**. Say which one you would pick and why; if parts of different designs combine well, propose the hybrid. Be opinionated.

Put the choice to the user as a single question, with your recommendation as the answer "yes" accepts.
