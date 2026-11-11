---
name: ship-change
description: Takes a change from request to reviewed diff by settling the decisions, planning the smallest change, implementing it with tests, running the project's checks, and having the diff reviewed independently. Use when the user asks to ship, build, or implement a feature or fix and expects a result that is ready to merge.
metadata:
  dmzkit-requires: grill agent:reviewer
---

# Ship change

Take one change from request to a diff that has passed the project's checks and an independent
review. Run the steps in order. Each step has an exit condition; do not start the next step until
it holds.

This workflow uses the `grill` skill and the `reviewer` agent from this kit.

## 1. Settle the decisions

Run the `grill` skill on the request. It ends with a decision record that the user has confirmed.

Exit: a confirmed decision record exists. If `grill` reports that the change is mechanical and
needs no interrogation, write a one-line record that states the change and continue.

## 2. Plan the smallest change

Write a plan that satisfies the decision record and nothing more:

- the files to change and what changes in each;
- the tests to add or update, one per decision that changes behavior;
- the order of the edits, so the project builds after each one where practical.

Leave out anything listed as out of scope in the record. Leave out refactors the change does not
need; note them for the final report.

Exit: every decision in the record maps to at least one planned edit or test, and every planned
edit traces back to a decision. If the plan requires a decision the record does not contain,
return to step 1 with that single question.

## 3. Implement with tests

Make the edits in the planned order. Follow the conventions of the surrounding code: naming, error
handling, test layout.

Write tests that would fail without the change and that assert on behavior. Cover the error paths
and boundaries the decision record names. When fixing a defect, write the failing test first and
watch it fail.

Exit: the plan is implemented and every planned test exists.

## 4. Run the project's checks

Find the checks the project defines (CI configuration, task runner, package scripts, contribution
guide) and run them: build, tests, linter, formatter check, type checker. Run what the project
runs, not a substitute.

Fix every failure the change caused, then run the checks again. A failure that also occurs without
the change is not yours to fix here; confirm that it does, record it for the report, and continue.

Exit: all checks pass, or the only failures are confirmed to predate the change.

## 5. Delegate the review

Delegate the review to the `reviewer` agent. It starts with no knowledge of this session, so pass
it everything it needs:

- how to obtain the diff: the exact command (for example `git diff main...HEAD`, or `git diff`
  for uncommitted work), or the diff itself;
- the confirmed decision record, as the statement of what the change must do;
- the command that runs the tests relevant to the change;
- any area you are unsure about, stated as a question.

Do not tell it what to conclude, and do not describe the change as correct.

If the harness has no `reviewer` subagent available, perform the review yourself before
continuing. Re-read the full diff from the top as if someone else had written it. Check
correctness, edge cases and error paths, security, tests, and maintainability, in that order, and
write the result in the same format the agent uses: numbered findings ordered by severity, each
with where (file and line), problem, evidence, and fix; then a verdict of `approve`, `approve with
minor fixes`, or `request changes`; then what was not verified. State in the final report that the
review was performed inline.

Exit: a review in that format exists.

## 6. Resolve the findings

Handle every finding. For each one, do exactly one of these:

- **Fix it.** Apply the fix, add or adjust a test when the finding concerned behavior, and run the
  checks from step 4 again.
- **Decline it.** Only when the finding is wrong, or when it contradicts the decision record.
  Write the reason, with evidence. "Out of time" and "minor" are not reasons.

Request a second review when any fix was non-trivial: it changed logic, touched code outside the
lines the finding named, or resolved a critical or major finding. Pass the reviewer the new diff
and the previous findings. A fix confined to a name, a comment, or a message does not need one.

Exit: every finding is fixed or declined with a reason, the checks pass, and the latest review
has no unresolved critical or major finding.

## 7. Report

Report to the user, briefly:

- what changed, by file, in terms of the decision record;
- the checks that were run and their result, including failures that predate the change;
- the review verdict, each finding with how it was resolved, and each declined finding with its
  reason;
- whether the review was delegated or performed inline;
- open risks carried over from the decision record, and follow-up work that was left out.

Do not commit, push, or open a pull request unless the user asked for it.

## Stop conditions

Stop and report to the user, without continuing to the next step, when:

- the user does not confirm the decision record;
- implementing the change shows that a recorded decision cannot be satisfied. State which one and
  why, and return to step 1 once the user responds;
- the checks cannot be run in this environment. Say which checks and why; do not report the change
  as verified;
- a check keeps failing after three distinct attempts to fix it. Report the failure and what was
  tried;
- a second review still returns a critical or major finding on the same problem. The approach is
  likely wrong; present the finding and the alternatives;
- the change turns out to need work the record lists as out of scope.

Never report the change as done while a check fails because of it or a finding is unresolved.
