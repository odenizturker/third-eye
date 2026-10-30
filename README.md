# third-eye

My agentic harness: skills, commands and other configuration for Claude Code.

## Skills

| Skill | What it does |
| --- | --- |
| [grill-with-docs](skills/grill-with-docs/SKILL.md) | Interviews you about a plan or design in rounds until you share one understanding, writing the agreed terms to `GLOSSARY.md` and hard system-wide decisions to `docs/architecture/` as it goes. |
| [to-spec](skills/to-spec/SKILL.md) | Turns the decisions from a grilling session into feature specs under `docs/specs/<domain>/`: business rules with concrete examples and user stories, no technical decisions. Creates new specs or updates existing ones in place. |
| [to-architecture](skills/to-architecture/SKILL.md) | Designs the architecture for draft or deprecated specs, or for an architecture-only change: deep modules, interfaces, seams, dependencies and rule ownership, written as living target-state docs under `docs/architecture/`. |

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

`grill-with-docs`, and the design rules in `to-architecture`, are adapted from [Matt Pocock](https://github.com/mattpocock)'s [skills](https://github.com/mattpocock/skills) (MIT). Adapted skills credit their author at the end of their `SKILL.md` and carry the upstream license in their own folder, so the notice travels with the skill when it is copied.

Everything else is [MIT](LICENSE) licensed.
