---
name: automated-pr-review-loop
description: Run an autonomous pull-request review loop for automated Codex and CodeRabbit feedback. Use when the user asks to wait for PR checks or automated reviews, address valid bot review comments, keep polling a PR after fix commits, or continue until automated review is complete. Triage every inline, outside-diff, nitpick, summary, and top-level finding; fix only findings supported by the code and established task assumptions; validate, commit, and push fixes; then process CodeRabbit incrementally until the latest head has no valid unaddressed feedback.
---

# Automated PR Review Loop

## Purpose

Carry an existing pull request through automated review. On the initial PR head, adjudicate both Codex and CodeRabbit feedback. After fix commits, process only CodeRabbit because Codex does not incrementally re-review later heads.

Treat every reviewer comment as a hypothesis. The goal is to resolve valid problems, not blindly satisfy bots. Do not implement false positives, duplicates, obsolete findings, subjective preferences, or suggestions that contradict established requirements.

## Workflow

1. **Resolve the PR and head.** Identify the repository, PR, base branch, current head SHA, and whether the local branch matches it. Record the head for each review round.
2. **Wait for the initial pass.** Poll checks and review activity for the initial head until required checks and the expected Codex and CodeRabbit reviews are terminal. A documented skip or non-applicable state is terminal; do not manually request another review.
3. **Read every feedback surface.** Use CodeRabbit's top-level **Prompt for all review comments with AI agents** section as its authoritative feedback source; it aggregates inline, outside-diff, summary, and nitpick findings efficiently. For Codex, inspect review threads, review bodies, inline and outside-diff comments, and issue-level comments as needed.
4. **Adjudicate each finding.** Inspect the current diff, surrounding code, callers, tests, conventions, and task assumptions. Classify findings as fix, reject, already addressed/obsolete, defer, or needs-user-clarification. Keep evidence for rejected findings so later polls do not reopen them.
5. **Fix and validate.** Make the smallest changes for valid findings and run validation appropriate to the affected code. Diagnose failed required checks and fix them when the branch caused the failure; report unrelated failures rather than changing unrelated code.
6. **Commit and push.** Group the accepted fixes coherently, commit them, push the PR branch, and record the new head SHA. Pushing is part of this loop because it triggers incremental review.
7. **Run subsequent passes.** For every later head, wait for required checks and CodeRabbit's incremental review, then read its updated aggregate AI-agent prompt. Do not wait for or reprocess Codex after the initial pass. Repeat adjudication, fixes, validation, commit, and push while valid new findings remain.
8. **Confirm quiescence.** Refresh once more after CodeRabbit has reached a terminal state for the latest head. Finish only when no valid unaddressed finding or branch-caused check failure remains.

## GitHub Review Rules

- Treat CodeRabbit's **Prompt for all review comments with AI agents** as the authoritative source for its findings. Do not spend tokens re-fetching and correlating its individual threads unless the aggregate prompt is unavailable or lacks context required to adjudicate a specific finding.
- Use thread-aware GitHub reads for Codex or exceptional cases where inline context, outdated status, or resolution state is necessary.
- Track findings by reviewer, stable comment or thread identity, and the head on which they appeared. Do not mistake old comments, bot edits, or automatically resolved threads for new findings.
- Do not manually resolve threads or reply merely to mark work complete. CodeRabbit should update resolution state during its later review.
- Do not comment `@coderabbitai review` or otherwise trigger a manual review unless the user explicitly asks.
- Poll with bounded waits and provide concise progress updates during long-running checks. Never claim a pending check or review has passed.

## Stop Conditions

Stop and ask the user before changing code when a finding requires a new or conflicting product, UX, data, security, compatibility, or public-contract assumption; introduces an unapproved dependency, migration, or broad refactor; or cannot be adjudicated safely from available evidence.

Complete only when:

- Required checks for the latest head are terminal, with branch-caused failures fixed and unrelated failures reported.
- Initial Codex findings and all CodeRabbit findings through the latest head have been adjudicated.
- CodeRabbit has completed or terminally skipped its review of the latest head.
- A final refresh finds no new valid feedback.

Report the final head, review rounds, accepted fixes, rejected or deferred findings, validation results, check states, and residual risks.
