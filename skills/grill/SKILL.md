---
name: grill
description: Interrogates the user about a planned change until every material decision is explicit, then writes a short decision record. Use before writing code for any non-trivial feature, fix, refactor, or migration, or when a request leaves behavior, scope, data handling, or compatibility open.
---

# Grill

Question the user about a planned change until no open decision could still alter the design. Write
no code and edit no files while this skill is active. The result is a decision record that the
implementation can be checked against.

## When to use

Use it when a request can be built in more than one reasonable way and the difference is visible to
users, to callers, to stored data, or to operations.

Skip it when the change is mechanical and has one sensible implementation (a rename, a typo, a
dependency bump with no API change). Say that the change needs no interrogation and proceed.

## 1. Prepare

A question the repository can answer wastes the user's attention. Before asking anything:

1. Read the code the change will touch, its callers, and its tests.
2. Read the documents that constrain it: README, architecture notes, decision records, contribution
   rules, and the configuration of the project's checks.
3. Find how the project already solved a similar problem. An existing convention is a decision
   that has been made; follow it and do not ask about it.
4. List every decision the change requires. For each one, record what the repository already
   settles and what remains open.

Only the open decisions become questions. If none remain, skip to the decision record.

## 2. Ask

Order the open decisions by consequence and reversibility. Ask first about what is expensive to
undo: data models and migrations, public interfaces, security boundaries, deleted behavior. Ask
last about what is cheap to change later.

Each question follows these rules:

- One decision per question. Split any question that contains "and".
- Give two to four concrete options. Put the recommended option first, mark it as recommended, and
  state its trade-off in one sentence: what it costs and what it rules out.
- Ground the question in what you read. Name the file, function, or constraint that makes the
  decision necessary.
- Ask about outcomes the user can judge. Do not ask the user to choose an implementation detail
  that has no visible consequence; decide it yourself and move on.
- Never ask an open-ended question such as "how should errors be handled". Propose the handling
  and ask whether it is right.

Ask in batches of at most four questions, then wait. Later questions usually depend on earlier
answers, so a long list asked at once produces answers to questions that no longer apply.

Example of one question:

> `export_report` loads all rows into memory (`reports/export.py:41`). The new filter can match
> the full table. Which limit applies?
>
> 1. Stream rows in pages of 1,000 (recommended). Memory stays flat. The export can no longer be
>    sorted by a computed column.
> 2. Keep the in-memory load and reject requests above 50,000 rows. Simpler, and large tenants get
>    an error.

## 3. Handle answers

- Record each answer as a decision with its reason. When the user gives no reason, record the
  trade-off they accepted.
- Follow dependencies. An answer can open new decisions and close others. Update the list before
  the next batch, and drop questions that no longer apply.
- Challenge an answer that hides a risk. State the risk once, with evidence from the repository: a
  caller that breaks, a test that encodes the opposite behavior, a constraint in the documentation.
  Then ask the user to confirm or change the answer. If they confirm, accept it and list the risk
  under open risks. Do not repeat the challenge.
- When the user says "you decide", take the recommended option and record it as delegated.
- When the user does not know, name what would settle it (a measurement, a person, a document).
  If it can be settled now by reading or by running a read-only command, do that. Otherwise record
  it as an open risk together with the assumption the implementation will proceed on.
- When an answer contradicts an earlier one, show both and ask which stands.

## 4. Stop

Stop asking when no open decision would change the design: every remaining choice either has one
sensible answer or is cheap to reverse after the change ships.

Also stop when:

- the user ends the interrogation. Record every unasked decision as an assumption under open risks;
- the answers show that the change should not be made, or that it is a different change. Say so
  and restate the new request before continuing.

Do not pad the interrogation. Three questions that settle the design are better than ten that also
cover what nobody would have disputed.

## 5. Output

Finish with the decision record and nothing else. Keep it short enough to read in a minute.

```markdown
## Decision record: <change in one line>

### Decisions
1. <decision>. Reason: <why>.
2. <decision>. Reason: <why>. (delegated)

### Rejected alternatives
- <alternative>: <why it lost>.

### Open risks
- <risk or unverified assumption>: <what would confirm or remove it>.

### Out of scope
- <what this change will not do>.
```

Rules for the record:

- Every decision is testable. "Fast enough" is not a decision; "p95 under 200 ms for 10,000 rows"
  is.
- Include the decisions the repository settled when the implementation depends on them, and cite
  the file that settles each.
- Write "None." under a heading that has no entries. Do not delete the heading.
- Ask the user to confirm the record. Implementation starts from the confirmed record, not from
  the conversation.
