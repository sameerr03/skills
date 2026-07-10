---
name: self-review-loop
description: Run an autonomous post-implementation review loop using independent read-only reviewer agents. Use after Codex completes a substantial code change, refactor, feature, migration, frontend build, or multi-file task and should review its own work, obtain focused adversarial feedback, triage findings, make only main-agent edits, re-run validation, and stop when a fix requires an unplanned product, design, data, security, or behavioral assumption. Prefer native subagents and use `codex exec` only when native agent tools are unavailable.
---

# Self Review Loop

## Purpose

Run an independent adversarial review after meaningful implementation work and before PR review. Reviewer agents produce feedback only; the main agent owns all decisions, edits, validation, and user communication.

Independence is required. Prefer native subagents. If native agent tools are unavailable, use separate read-only `codex exec` reviewers. If neither is available, report that the loop could not run; never substitute inline self-review by the implementing agent.

## Choose Reviewers From the Change

Select reviewer expertise from the contracts and failure modes affected by the change, not from diff size or a fixed reviewer count. Almost always include:

- **Correctness and regressions:** verify required behavior, edge cases, and compatibility with established assumptions.
- **Code simplicity:** find duplicated logic; constants, configuration, literals, or types duplicated across files; unnecessary abstractions; and defensive, generic, or type-heavy handling beyond actual requirements and callers. Avoid unrelated cleanup and subjective style advice.

Add focused reviewers when the work calls for them, such as security/privacy, test adequacy, UX/accessibility, performance, migration/rollback behavior, concurrency, or domain calculations. This list is illustrative: invent a specialized lane when the task has another important failure mode. Avoid multiple vague reviewers covering the same surface.

### Data Validation

Use a data-validation reviewer when a change writes, migrates, backfills, aggregates, or precomputes consequential database values. It should:

- Work read-only against an explicitly permitted local or staging database; never production without explicit authorization.
- Derive expected results independently from the task's business or mathematical contract. Do not call, import, copy, or mechanically translate the implementation being reviewed.
- Use a reproducible random or stratified sample covering relevant groups and boundary cases, retrieve the source facts, and compare independently calculated results with stored values using the contract's precision, rounding, null, tie, and grouping rules.
- Pair sampling with practical full-table invariant checks such as coverage, ranges, uniqueness, population counts, missing values, and stale or ineligible rows.
- Report the sample method, identifiers, reasoning, expected and actual values, mismatches, invariant results, and validation limits. Missing access or an unclear contract is a validation gap, not evidence that the data is correct.

## Workflow

1. **Establish the surface.** Summarize the task and changed behavior; inspect `git diff`, `git status`, and existing validation; identify affected contracts and choose distinct reviewer lanes.
2. **Launch read-only reviewers.** Prefer one native `spawn_agent` task per lane. Give each reviewer the goal, changed surface, constraints, established assumptions, and exact focus. Explicitly forbid edits and state-changing operations because native agents share the workspace. Respect available agent slots.
3. **Collect every result.** Use `wait_agent` until each reviewer finishes, or explicitly `interrupt_agent` when one is no longer relevant. When native agents are unavailable, run equivalent focused reviewers through separate `codex exec` invocations.
4. **Triage findings.** Treat each finding as a hypothesis. Verify it against current code and task intent, then classify it as fix, reject, defer, or needs-user-clarification. Reconcile duplicates and conflicts; do not change code for false positives or reviewer preferences.
5. **Iterate as the main agent.** Make minimal fixes for validated findings and rerun relevant tests, type checks, linters, builds, or focused checks. After substantive fixes, use `followup_task` for continuity or spawn a fresh independent reviewer when a new perspective is more valuable.
6. **Report.** State the reviewer mechanism and lanes, accepted fixes, rejected or deferred findings, residual risks, and validation results.

Ask reviewers for severity-ranked actionable findings with exact file and line evidence, reasoning or reproduction, and the smallest fix. They should distinguish confirmed defects from questions or unstated assumptions and say clearly when they find nothing.

## Stop Conditions

Stop and ask the user before editing when a finding requires choosing an unstated behavior or changing an unapproved product contract, public API, persistence shape, permissions, security posture, generated data, dependency set, migration strategy, or user-visible workflow. Also stop when reviewers conflict and evidence cannot resolve the decision safely.

End only after every reviewer has completed or been interrupted, every finding is fixed, rejected with evidence, deferred with rationale, or escalated, and relevant validation has passed or unrelated failures have been reported. Run another round only when fixes were substantive or meaningful uncertainty remains.
