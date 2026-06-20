---
name: self-review-loop
description: Run an autonomous post-implementation review loop using independent read-only `codex exec` subagents. Use after Codex completes a substantial code change, refactor, feature, migration, frontend build, or multi-file task and should review its own work, ask one or more focused reviewer agents for feedback, triage findings, make only main-agent edits, re-run validation, and stop for the user when reviewer feedback requires an unplanned product, design, data, security, or behavioral assumption.
---

# Self Review Loop

## Overview

Use this skill after meaningful implementation work to create an explicit self-review loop. The main agent owns all decisions and code changes; `codex exec` subagents are independent read-only reviewers that produce feedback only.

## Core Rules

- Treat subagents as reviewers, not implementers. Their prompts must explicitly forbid file edits, commits, pushes, destructive commands, and network use unless the main task already requires and permits it.
- Keep the main agent responsible for all code edits, test runs, final judgment, and user communication.
- Do not hide uncertainty behind the loop. Stop and ask the user when a proposed fix depends on a requirement, product behavior, data contract, UX choice, security posture, or compatibility promise that was not already established.
- Prefer focused reviewers over one vague reviewer for large work: correctness, tests, security, performance, UX/accessibility, migrations, or domain-specific behavior.
- Keep reviewer context minimal but sufficient: the task goal, changed files or diff, relevant constraints, and exact review focus.
- Continue the loop only while it is producing actionable signal. One review round is often enough for small work; use additional rounds after substantive fixes or when risk remains.

## Workflow

1. Establish the review surface.
   - Summarize what changed and why.
   - Inspect `git diff`, `git status`, and any test output already available.
   - Identify risk areas and choose reviewer lanes.

2. Launch read-only reviewer agents with `codex exec`.
   - Run reviewers from the repository root or task workspace.
   - Use one command per reviewer when parallelism is useful.
   - Ask each reviewer to return severity-ranked findings with file/line references, reproduction or reasoning, and a clear fix recommendation.
   - Ask reviewers to say "no findings" when appropriate.

3. Triage reviewer output.
   - Validate each finding against the current code and task intent.
   - Classify it as fix, reject, defer, or needs-user-clarification.
   - Reject false positives explicitly in the main agent's notes; do not change code to satisfy an invalid finding.

4. Stop for clarification when needed.
   - Pause before editing if the fix would choose among plausible behaviors the user did not define.
   - Present the assumption, why it matters, the options, and the recommended path.
   - Resume only after the user answers.

5. Iterate as the main agent.
   - Make the minimal code changes needed for validated findings.
   - Re-run relevant tests, type checks, linters, builds, or focused manual checks.
   - If fixes were substantial, launch a follow-up reviewer round against the new diff.

6. Finish with a concise report.
   - State what reviewer lanes ran.
   - Summarize accepted fixes, rejected findings, and any residual risks.
   - Include validation commands and whether they passed.

## Reviewer Prompt Template

Use a prompt like this, adapting the focus lane and context:

```text
You are an independent read-only reviewer. Do not edit files, run destructive commands, commit, push, or change system state. Review the work in this repository for: <focus lane>.

Task goal:
<brief original task>

Changed surface:
<changed files, diff summary, or instructions to inspect git diff>

Constraints:
<tests already run, user constraints, product assumptions already established>

Return severity-ranked findings only. For each finding include file/line reference, why it matters, how to verify it, and a recommended fix. If the fix requires an unstated assumption, call that out instead of choosing for the main agent. If there are no findings, say so clearly.
```

## Example Commands

Use `codex exec` with a single quoted prompt. Example:

```bash
codex exec "You are an independent read-only reviewer. Do not edit files, commit, push, or change system state. Review the current git diff for correctness and regression risk. Return severity-ranked findings with file/line references, verification steps, and recommended fixes. If there are no findings, say so clearly."
```

For multiple lanes, run separate prompts such as:

- Correctness and regressions
- Test coverage gaps
- Security and data exposure
- UX, accessibility, and responsive behavior
- Performance and scalability
- Migration, rollout, and compatibility risk

## Stop Conditions

Stop the loop and ask the user before changing code when:

- A reviewer identifies multiple valid fixes with different product semantics.
- The recommended fix changes public API behavior, persistence shape, pricing, permissions, security posture, generated data, or user-visible workflow beyond the original request.
- The fix requires introducing a new dependency, long-running migration, broad refactor, or destructive cleanup not already approved.
- The reviewer may be right but the evidence is insufficient and verification would be expensive or risky.
- Reviewer findings conflict with each other or with the original user instruction.

End the loop when:

- Reviewer findings are fixed, rejected with evidence, deferred with rationale, or escalated to the user.
- Relevant validation passes, or failures are unrelated and clearly reported.
- Additional reviewer rounds are unlikely to produce new actionable signal.
