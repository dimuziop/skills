# dmz

The skills, workflows, and agents that Patricio Di Muzio (dmz) uses to plan, build, and review
changes with coding agents. This repository is a [dmzkit](https://github.com/dimuziop/dmzkit) kit:
a set of plain Markdown files that installs into Claude Code, Codex, Cursor, OpenCode, Gemini CLI,
GitHub Copilot, and omp.

## Install

This kit is dmzkit's default source, so no arguments are needed:

```sh
npx dmzkit add
```

dmzkit asks which items and which harnesses to install for, then copies the items into the current
project and links them into each harness.

Select items and harnesses on the command line:

```sh
npx dmzkit add dimuziop/skills --item ship-change --harness claude-code
```

Installing an item also installs everything it requires. `ship-change` brings `grill` and the
`reviewer` agent with it.

Install for your user instead of the current project with `-g`:

```sh
npx dmzkit add -g
```

`npx dmzkit list` shows what is installed and whether it has been modified.

### With `npx skills`

The kit also installs with the [`skills`](https://github.com/vercel-labs/skills) CLI:

```sh
npx skills add dimuziop/skills
```

That tool installs the skills and the workflows. It does not install agents, and it does not
install what an item requires: after installing `ship-change` you have to add `grill` yourself,
and the `reviewer` agent is not available, so the workflow performs its review inline.

## Items

| Kind     | Name          | Purpose                                                                                   |
| -------- | ------------- | ----------------------------------------------------------------------------------------- |
| skill    | `grill`       | Questions the user about a planned change until every material decision is explicit.      |
| workflow | `ship-change` | Takes a change from request to a diff that passed the project's checks and a review.      |
| agent    | `reviewer`    | Read-only code reviewer that returns findings ordered by severity and a verdict.          |

Agents are written once in a harness-neutral format and translated at install time. Harnesses that
dmzkit has no agent emitter for still receive the skills and workflows.

## Layout

```
dmzkit.toml                     kit manifest
skills.sh.json                  makes workflows/ visible to npx skills
skills/<name>/SKILL.md          skills
workflows/<name>/SKILL.md       workflows: skills that orchestrate other items
agents/<name>.md                subagent definitions
AGENTS.md                       authoring rules for this repository
```

The format is specified in
[kit-format.md](https://github.com/dimuziop/dmzkit/blob/main/docs/kit-format.md).

## Adding an item

```sh
npx dmzkit new skill <name>
```

Use `workflow` or `agent` in place of `skill` for the other kinds. Edit the generated file
following the rules in [AGENTS.md](AGENTS.md), then check the kit:

```sh
npx dmzkit validate
```

The same command runs in CI on every push and pull request.

## License

[MIT](LICENSE). Copyright (c) 2026 Patricio Di Muzio.
