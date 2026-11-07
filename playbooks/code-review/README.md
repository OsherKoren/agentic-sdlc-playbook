# Code review with AI agents

Use this playbook to review a local branch or pull request, or as a starting point for adding agent review to a repository. The target repository's rules govern access, checks, and approvals.

## Run a review

Give the reviewer the intended behavior, the base and head commits, and the repository instructions. Start with this request:

> Review this change using this playbook. Inspect affected callers and tests. Report confirmed regressions first, with evidence and impact. List what you verified and what remains unchecked. Keep the review read-only unless I ask for fixes.

1. **Establish the scope.** Read the change description and diff. If the intent is missing, say what you inferred and flag any ambiguity that could affect the verdict.
2. **Collect evidence.** Run relevant checks where authorized. For a bug fix, reproduce the failure and show that the fix closes it. For a UI change, exercise the affected flow. Record the command and outcome, including checks you could not run.
3. **Review independently.** Use a fresh reviewer context after implementation. Trace the changed behavior through callers, configuration, and tests. Look for incorrect results, broken contracts, unsafe state transitions, and missing failure handling.
4. **Verify before reporting.** For each candidate, identify a reachable trigger and concrete consequence. Reproduce it or trace the code path. On sensitive changes, ask a separate reviewer to challenge candidate findings and run a dedicated security pass.
5. **Return a decision-ready report.** List actionable findings in severity order. Give each a file and line, trigger, impact, evidence, and a suggested direction. Separate pre-existing issues and unverified questions. If there are no confirmed findings, state that and list the checks performed.

### Calibrate the depth

| Change | Review focus |
| --- | --- |
| Copy or docs | Verify that instructions and examples agree with the current behavior. |
| Product behavior | Trace affected callers and cover success and failure paths. |
| Auth, billing, data migrations, or release workflows | Trace the blast radius, verify recovery assumptions, and route to the relevant human owner under repository policy. |

For example, when a Kafka consumer changes its retry behavior, trace what happens if the downstream publish succeeds and the consumer crashes before committing its offset. Check whether a replay can duplicate a side effect and where deduplication occurs.

## Adopt it in a repository

1. **Pilot on one repo.** Keep existing CI and approval rules. Run the agent in advisory mode and record which findings were fixed, dismissed, or missed.
2. **Add repo-specific checks.** Put recurring review criteria in local reviewer instructions. Convert repeatable failures into tests or lint checks where possible. Keep dedicated security checks separate from general review.
3. **Define decision rules.** Write down protected paths, required checks, and human approval requirements. Use automated merge only where repository policy explicitly permits it. Revalidate evidence and required approvals when the reviewed commit changes.
4. **Keep review rounds bounded.** Recheck changed code and earlier findings after fixes. If the review still has blockers after three rounds, hand it back to the author to reassess the approach.
5. **Close the loop.** Check the integrated change before merge. For consequential rollouts, record the owner, monitoring signal, and rollback action. Evaluate reviewer changes against past escaped defects and known-clean diffs. Track time waiting for review, actionable findings, and post-merge regressions.

## References

Background for the workflow design:

- [How the Claude Code team does code review](https://claude.dev/blog/solving-code-review-at-anthropic/)
- [Video discussion](https://www.youtube.com/watch?v=4m8mq42M_3E)
