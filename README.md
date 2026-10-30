# third-eye

My agentic harness: skills, commands and other configuration for Claude Code.

## Skills

| Skill | What it does |
| --- | --- |
| [grill-with-docs](skills/grill-with-docs/SKILL.md) | Interviews you about a plan or design in rounds until you share one understanding, writing the agreed terms to `GLOSSARY.md` and hard system-wide decisions to `docs/architecture/` as it goes. |
| [to-spec](skills/to-spec/SKILL.md) | Turns the decisions from a grilling session into feature specs under `docs/specs/<domain>/`: business rules with concrete examples and user stories, no technical decisions. Creates new specs or updates existing ones in place. |
| [to-architecture](skills/to-architecture/SKILL.md) | Designs the architecture for draft or deprecated specs, or for an architecture-only change: deep modules, interfaces, seams, dependencies and rule ownership, written as living target-state docs under `docs/architecture/`. |
| [improve-codebase-architecture](skills/improve-codebase-architecture/SKILL.md) | Surveys the codebase for deepening opportunities and presents them as a visual HTML report. The candidate you pick goes on to an architecture-only `to-architecture` session. |

The skills above are the ones you run. The skills below are loaded by the model when a task needs them (you can run them too):

| Skill | What it does |
| --- | --- |
| [grilling](skills/grilling/SKILL.md) | The interview itself: rounds of numbered questions, each with a recommended answer, until every branch of the design tree is resolved. Used by `grill-with-docs`, `to-spec` and `to-architecture`. |
| [codebase-design](skills/codebase-design/SKILL.md) | The deep-module vocabulary and design rules, plus "design it twice": parallel sub-agents that design an interface several ways before you choose. |
| [research](skills/research/SKILL.md) | Answers a question about something outside the repo from primary sources, in a background agent, and saves the cited findings under `docs/research/`. |

## Installation

Pick one route per agent: installing more than one gives you every skill twice.

### Claude Code plugin

```sh
claude plugin install third-eye --marketplace odenizturker/third-eye
```

Or, inside a session, `/plugin install third-eye --marketplace odenizturker/third-eye`. This adds the `odenizturker` marketplace and installs the `third-eye` plugin. Skills are namespaced by the plugin, so you run `/third-eye:grill-with-docs`. Update with `claude plugin update third-eye`.

### skills.sh (any agent)

```sh
npx skills@latest add odenizturker/third-eye
```

Copies editable skill files into your project for Claude Code, Cursor, Codex and other agents. Update with `npx skills@latest update`.

### Manually

Each skill is a self-contained folder under `skills/`. From a clone of this repo, symlink one into Claude Code's skills directory so it stays in sync:

```sh
mkdir -p ~/.claude/skills
ln -s "$PWD/skills/grill-with-docs" ~/.claude/skills/grill-with-docs
```

Use `~/.claude/skills/` for every project, or `<project>/.claude/skills/` for one project. Copy the folder (`cp -R`) instead for a fixed snapshot.

## Releasing

Bump `version` in `.claude-plugin/plugin.json` with every change you want plugin users to receive; installed copies only update when the version changes.

## Credits and license

`grill-with-docs`, `grilling`, `codebase-design`, `research` and `improve-codebase-architecture` are adapted from [Matt Pocock](https://github.com/mattpocock)'s [skills](https://github.com/mattpocock/skills) (MIT). Adapted skills credit their author at the end of their `SKILL.md` and carry the upstream license in their own folder, so the notice travels with the skill when it is copied.

Everything else is [MIT](LICENSE) licensed.
