# Authoring rules

Rules for coding agents that edit this repository. It is a
[dmzkit](https://github.com/dimuziop/dmzkit) kit named `dmz`.

## Format in brief

The full specification is
[kit-format.md](https://github.com/dimuziop/dmzkit/blob/main/docs/kit-format.md). Read it before
adding a kind of content this file does not cover.

| Kind     | Location                    | Frontmatter                                                            |
| -------- | --------------------------- | ---------------------------------------------------------------------- |
| skill    | `skills/<name>/SKILL.md`    | `name`, `description`; optional `metadata.dmzkit-requires`             |
| workflow | `workflows/<name>/SKILL.md` | `name`, `description`, `metadata.dmzkit-requires`                      |
| agent    | `agents/<name>.md`          | `name`, `description`, optional `tools`, optional `model`; nothing else |

- `dmzkit.toml` holds `kit.name` and `kit.description`. Unknown keys are errors.
- `skills.sh.json` keeps `workflows/` visible to `npx skills`. Do not change it.
- Skills and workflows share one namespace. Agents have their own.
- Frontmatter is YAML. Quote a value that contains `: `, ` #`, or starts with a character YAML
  treats specially.

## Names

- Lowercase letters, digits, and hyphens. 1 to 64 characters. No leading, trailing, or consecutive
  hyphens.
- The `name` field equals the directory name (skills, workflows) or the file stem (agents).
- Name an item for what it does, with a verb or a role: `grill`, `ship-change`, `reviewer`.

## Descriptions

The description is the only text a harness reads when deciding whether to load an item.

- Say what the item does and when to use it. Both parts are required.
- Write in the third person: "Reviews a diff ...", "Use when ...". Not "I review" or "You can use".
- Stay under 1024 characters. One or two sentences is normal.
- Name the situations and terms a user would mention, so the harness can match the request.

## Skills

- Keep `SKILL.md` under 500 lines. Move detail (reference tables, long examples, scripts) into
  files next to it and reference them with relative paths, such as `references/checklist.md`.
  State when the harness should read each file.
- Write instructions for the harness, in the imperative.
- State the steps in order. State the stop conditions: when the skill is finished, and when it
  must stop early and return to the user.
- Define the output. If the skill produces a document, give its exact structure.
- Do not explain what a capable model already knows. Spend the lines on what is specific to the
  task.
- One skill does one job. If the body describes two procedures that are useful separately, make
  two skills.

## Workflows

- A workflow orchestrates. It sequences skills and agents, says what to pass to each one, and
  defines the exit condition of each step. It does not restate their instructions.
- Declare every skill, workflow, and agent it invokes in `metadata.dmzkit-requires`, as
  space-separated references: `name`, `skill:name`, `workflow:name`, or `agent:name`. A bare name
  never matches an agent.
- Requirements resolve inside this kit. A reference that matches nothing is an error.
- Agents are not emitted for every harness. Say what to do when a required agent is unavailable.

## Agents

- Use only the canonical tools: `read`, `write`, `edit`, `bash`, `grep`, `glob`, `web-fetch`,
  `web-search`.
- Use only the canonical model tiers: `inherit`, `fast`, `balanced`, `deep`.
- Grant the minimum tools the agent needs. An agent that only reports gets no `write` or `edit`.
  List `tools` explicitly; omitting the field grants every tool.
- The body is the system prompt and must not be empty. The agent starts with no knowledge of the
  parent session, so define its scope, its limits, and a strict output format.
- Unknown frontmatter keys are errors.

## Writing

- English only, in every file.
- Plain and precise. No emojis, no exclamation marks, no marketing language.
- Write original text. Do not copy or paraphrase other skill collections.
- Update the item table in `README.md` when adding, renaming, or removing an item.

## Before finishing

Run the validator from the repository root and fix every error and warning it prints:

```sh
npx dmzkit validate
```
