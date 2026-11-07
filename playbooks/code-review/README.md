Playbook for an agentic code review pipeline
Give the coding agent a way to prove its work, have independent agents challenge it, and route human review by the consequences of a bad merge. Then make the change easy to detect and undo if it breaks in production.

This playbook combines the video’s narration and presenter commentary with the Claude Code team’s original article, which contains the workflow diagrams discussed at 7:44 and 10:45. I used the original article for the diagram content. Visual coverage is limited to that source material. claude.dev
1. Before the PR, make the agent prove the change works
Give the authoring agent the intended behavior and a concrete way to verify it. For a bug fix, have it reproduce the original failure and show the same check passing after the fix. For a UI change, have it exercise the relevant flow and capture a screenshot. For an API change, have it run an integration test that checks the affected behavior and its failure paths.

Then have a second agent review the diff in a fresh context. Give it the PR intent, diff, repo access, and review rules. Ask it to inspect callers and look for regressions independently. Keep the PR small enough that a human can understand it later. The presenter particularly emphasizes the fresh context and the value of keeping PR history legible. Video at 11:04 · Video at 15:57

A practical PR template would ask for four things:
- Intent — what behavior should change and why.
- Evidence — exact commands, results, and screenshots where relevant.
- Risk — affected callers, dependencies, and what the checks could miss.
- Recovery — who owns the rollout and how to disable or revert it.
2. On the PR, run three separate checks
Check	What it should do	Merge rule
CI	Run lint, type checks, relevant tests, and build or packaging checks against the proposed change.	Required checks must pass.
Agent bug review	Read the diff in repo context. Check intent, root cause, callers, and regressions. On deeper reviews, run parallel bug hunters and have separate agents verify candidate findings before posting.	Resolve blocking findings with a fix or a recorded reason.
Security review	Run a dedicated pass with its own criteria and findings.	Resolve blocking security findings before merge.


Make each finding actionable. Require a severity, file and line, triggering condition, consequence, and supporting evidence. Deduplicate findings, suppress style-only comments, and label pre-existing bugs separately. A useful review can end with zero findings. The Claude Code team uses separate agents to verify findings, and the presenter suggests surfacing evidence in a compact format that a person can scan. Video at 14:54 · claude.dev

Keep the instructions short. Use your agent’s repo instructions for how to build, test, and verify. Use a REVIEW.md for the failures that matter in each area. For example, a Kafka worker review might ask whether a retry can publish the same event twice. If the same finding keeps coming back, move the rule into a test, linter, or authoring instruction where it can prevent the issue earlier. Video at 13:30
3. Route human approval by blast radius
The presenter’s leaf versus trunk analogy is a useful starting point. A change to an isolated feature has fewer downstream dependencies. A change to a shared authorization layer, release workflow, or widely used interface can affect many paths even when the diff is small. Video at 8:49
Risk	Example	Approval policy to work toward
Low	Copy edit, isolated internal change, additive behavior behind a flag	Eligible for automated merge after required checks pass, under an explicit trusted-author policy
Medium	Ordinary product behavior change or a change to a shared dependency with a contained impact	One human approval
High	Broad permission change, billing behavior, or a sensitive release path	Two human approvals, including a relevant owner where appropriate


Start with explicit path rules and let a classifier assess the rest of the diff. Protected paths set a minimum review requirement. Uncertain classifications go to a human. A bot can satisfy an automated check, and human approval applies to the specific commit the reviewer examined. The Claude Code team also uses a second classifier to identify changes that merit another reviewer. Video at 19:23 · claude.dev
4. Bound the review-and-fix loop
The article describes a PR that went through more than 50 rounds overnight. The reviewer kept discovering issues in code it had already seen. That makes rounds to pass a metric worth tracking alongside the bugs caught. Video at 27:22
- Run the first review against the full PR and its surrounding code.
- On follow-up rounds, focus on the new diff and verify that earlier findings are resolved.
- Group multiple instances of the same bug so the author can fix the whole pattern in one pass.
- After three rounds, escalate to the author to reconsider the approach. Keep unresolved blockers open.
- Run broader searches for old bugs separately so routine PRs can converge.
5. Make rollout and recovery part of review
Use a merge queue to rerun required checks against the current base branch with the PR applied. Roll out meaningful behavior changes behind a flag or to a small cohort first. Name the person watching the rollout, the signal they’ll watch, and the action they’ll take if it regresses. That could be disabling the flag, reverting the commit, or rolling back the release. Video at 22:35

When an issue escapes, turn the original PR and its fix or revert into a review evaluation case. Ask which layer could have caught it and what evidence that layer needed. Include known-clean PRs to measure false positives. Track review wait time and rounds to pass alongside revert and incident rates, so a faster review process doesn’t simply move the work to on-call. Video at 25:38
How I’d roll this out on one repo
1. Keep the current approval rules. Add the PR evidence template, a fresh-context reviewer, and a separate security pass. Run them on real PRs while collecting feedback on which findings were useful.
2. Tune the review before making it a gate. Add repo-specific failure examples, filter noise, and test against past incidents and known-clean PRs. Then require resolution of verified blocking findings.
3. Automate the narrowest merge path first. Start with clearly low-risk changes from trusted authors once CI, merge queue checks, and rollback are in place. Expand eligibility when review wait time improves and revert and incident rates hold steady.
For a backend repo, a good first comparison is a docs-only PR versus a Kafka retry change. The first tests whether the pipeline can stay quiet on a clean change. The second tests whether it can trace a real failure across boundaries, such as a publish succeeding before a retry and causing duplicate downstream processing. That gives you two concrete cases to evaluate before tuning the pipeline for higher volume.