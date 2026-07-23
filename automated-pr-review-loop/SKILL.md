---
name: automated-pr-review-loop
description: Run an autonomous pull-request review loop for Codex and CodeRabbit feedback, including base-aware review triggering, token-efficient polling, and rate-limit recovery. Use when the user asks to wait for PR checks or automated reviews, address valid bot review comments, keep polling a PR after fix commits, or continue until automated review is complete. Independently verify and adjudicate every inline, outside-diff, nitpick, summary, and top-level finding before editing; automatically fix only findings supported by the code and established task assumptions; validate, commit, and push fixes; then process reviews until CodeRabbit has no valid unaddressed feedback.
---

# Automated PR Review Loop

## Purpose

Carry an existing pull request through automated review. Review triggering depends on the PR's base branch:

- When the base branch is `main`, Codex and CodeRabbit automatically review the PR when it is created and automatically run subsequent reviews after every new commit. Do not post a manual review-trigger comment except when recovering from a confirmed rate limit.
- When the base branch is not `main`, Codex and CodeRabbit do not automatically run reviews. Post the exact comment `@coderabbitai review` to trigger CodeRabbit for the current head, including after every new commit. The comment does not trigger Codex, so do not wait for Codex unless a Codex review is actually present.

Treat every reviewer comment as a hypothesis. Independently verify and adjudicate it against the current code before editing. The goal is to resolve valid problems, not blindly satisfy bots. Do not implement false positives, duplicates, obsolete findings, subjective preferences, or suggestions that contradict established requirements. Do not pause for user approval between adjudication and implementation: automatically fix findings supported by the evidence and leave the rest unchanged with a concise reason.

## Workflow

1. **Resolve the PR and head.** Identify the repository, PR, base branch, current head SHA, and whether the local branch matches it. Record the head for each review round. Treat the exact base branch as the review-trigger policy switch.
2. **Start the initial pass.** If the base is `main`, wait for the Codex and CodeRabbit reviews that start automatically when the PR is created. If the base is not `main`, post `@coderabbitai review` once for the current head and wait for CodeRabbit; do not expect Codex unless it appears. Use the token-efficient polling policy below until expected review activity is terminal. A documented skip or non-applicable state is terminal.
3. **Read every feedback surface.** Use CodeRabbit's top-level **Prompt for all review comments with AI agents** section as its authoritative feedback source; it aggregates inline, outside-diff, summary, and nitpick findings efficiently. For Codex, inspect review threads, review bodies, inline and outside-diff comments, and issue-level comments as needed.
4. **Independently adjudicate before editing.** Complete a read-only evidence pass over every finding before making any change. Inspect the current diff, surrounding code, callers, tests, conventions, and task assumptions rather than trusting the review comment itself. Classify each finding as fix, reject, already addressed/obsolete, defer, or needs clarification. Keep evidence for every non-fix classification so later polls do not reopen it.
5. **Fix and validate without an approval pause.** Automatically make the smallest correct changes for every finding classified as fix, then run validation appropriate to the affected code. Do not implement findings that could not be independently verified. Diagnose failed required checks and fix them when the branch caused the failure; report unrelated failures rather than changing unrelated code.
6. **Commit and push.** Group the accepted fixes coherently, commit them, push the PR branch, and record the new head SHA. On a `main`-based PR, the push starts automatic reviews; on any other base, continue to the manual trigger in the next step.
7. **Run subsequent passes.** For every later head, follow the base-branch policy again. If the base is `main`, wait for both Codex and CodeRabbit to automatically review the new commit. If the base is not `main`, post `@coderabbitai review` once for that head and wait for CodeRabbit; process Codex only if it appears. Use the same token-efficient polling policy, read updated feedback surfaces only after a state change, and repeat adjudication, fixes, validation, commit, and push while valid new findings remain.
8. **Confirm quiescence.** Refresh once more after all reviews expected for the latest head have reached a terminal state. Finish only when no valid unaddressed finding or branch-caused check failure remains.

## GitHub Review Rules

- Treat CodeRabbit's **Prompt for all review comments with AI agents** as the authoritative source for its findings. Do not spend tokens re-fetching and correlating its individual threads unless the aggregate prompt is unavailable or lacks context required to adjudicate a specific finding.
- Use thread-aware GitHub reads for Codex or exceptional cases where inline context, outdated status, or resolution state is necessary.
- Track findings by reviewer, stable comment or thread identity, and the head on which they appeared. Do not mistake old comments, bot edits, or automatically resolved threads for new findings.
- Do not manually resolve threads or reply merely to mark work complete. CodeRabbit should update resolution state during its later review.
- On PRs based on `main`, rely on the automatic Codex and CodeRabbit reviews after PR creation and after every new commit; never add a redundant manual trigger except for the rate-limit recovery procedure below.
- On PRs not based on `main`, post `@coderabbitai review` once per head, including after every new commit, because reviews do not run automatically. Track the head for each trigger so polling never posts duplicates.
- Follow the polling policy below. Never claim a pending check or review has passed.

## Token-Efficient Polling

- After a review starts or is requested, perform one lightweight status read to record the head, expected reviewers, current states, and latest review identifiers or timestamps.
- While CodeRabbit remains pending, wait ten minutes between status polls by default. Do not run rapid, continuous, or one-minute polling loops. Check Codex state in the same lightweight request rather than creating a separate high-frequency loop.
- Treat every ten-minute wake as read-only polling. If CodeRabbit is still pending or running, record the unchanged state and schedule the next wake; never post another `@coderabbitai review` merely because ten minutes elapsed. Trigger only once per head, except for the documented rate-limit recovery after its reset time.
- In the Codex app, use `automation_update` to create or reuse exactly one ten-minute heartbeat attached to the current task. Immediately yield and end the active turn after scheduling it. Each heartbeat wake performs exactly one lightweight status poll; if CodeRabbit remains pending, leave or update that same heartbeat and yield again. Never create duplicate polling automations.
- Never use `wait_agent` or `wait_threads` as a clock. Those tools wait for agent or task events, not elapsed time, and may return before the polling interval. Never use terminal `sleep`, prose, elapsed-time narration, or repeated commentary to simulate a wait.
- If `automation_update` or a current-task heartbeat is unavailable, report the missing scheduling capability once and pause the loop rather than improvising another wait mechanism.
- During pending polls, fetch only the minimum status metadata needed to detect a changed, skipped, failed, or completed review. Do not repeatedly fetch the diff, full comment threads, or CodeRabbit's aggregate prompt while its review state and identifiers are unchanged.
- Fetch and inspect full feedback only when a reviewer becomes terminal or its review identifier, update timestamp, or comment count changes. Reset the ten-minute cadence after every new commit or review trigger.
- Treat a scheduled wait as suspended work, not active work requiring progress updates. Do not claim to be halfway through a wait or send "still waiting" messages. The scheduling action is the only update until the task wakes.
- When the review becomes terminal or the loop stops for any reason, disable or delete the polling heartbeat immediately so it cannot wake the task again.
- A known rate-limit reset time overrides the ten-minute cadence: follow the recovery policy below and do not poll during the limited window.

## Rate-Limit Recovery

- Do not treat a rate-limited review as completed or terminally skipped. Inspect the bot response, check output, or available rate-limit metadata to identify the exact reset time. If the service reports a relative duration, convert it to an absolute timestamp and record it. Do not guess a reset time that the available evidence does not support.
- Do not poll or post repeated review comments during the limited period. In the Codex app, use `automation_update` to update the same current-task heartbeat for the reset time, then immediately yield and emit no interim progress messages. If that scheduling capability is unavailable, report it once and pause the loop. Remove the heartbeat when recovery or the loop completes.
- At or just after the reset time, refresh the PR and confirm its current head. If the head is unchanged, post `@coderabbitai review` once to request the CodeRabbit review; this is the sole manual-trigger exception for a `main`-based PR. If the head changed, apply the normal base-branch trigger policy to the new head instead.
- Record the rate-limit event, reset time, affected head, and retry comment so the loop neither retries early nor submits duplicate requests.

## Stop Conditions

Do not stop for adjudication approval. When a finding depends on a new or conflicting product, UX, data, security, compatibility, or public-contract assumption, or cannot be verified safely from available evidence, leave it unchanged and record why it was not valid to fix in this loop.

Complete only when:

- Required checks for the latest head are terminal, with branch-caused failures fixed and unrelated failures reported.
- Every Codex and CodeRabbit finding received under the base-branch policy has been independently verified and adjudicated through the latest head. For a non-`main` base, also adjudicate any Codex review that happens to be present.
- For a `main` base, Codex and CodeRabbit have completed or terminally skipped their automatic reviews of the latest head. For a non-`main` base, the manually triggered CodeRabbit review has completed or terminally skipped.
- No unresolved rate limit remains; any rate-limited CodeRabbit review was retried only after its identified reset time.
- CodeRabbit either has no comments on the latest head or has only comments adjudicated as not valid to fix. A final refresh finds no new valid feedback.

Report the final head, review rounds, accepted fixes, rejected or deferred findings, validation results, check states, and residual risks.
