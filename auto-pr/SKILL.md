---
name: auto-pr
description: Run an autonomous pull-request review loop for Codex and CodeRabbit feedback, including base-aware review triggering, token-efficient polling, and rate-limit recovery. Use when the user asks to wait for PR checks or automated reviews, address valid bot review comments, keep polling a PR after fix commits, or continue until automated review is complete. Independently verify and adjudicate every inline, outside-diff, nitpick, summary, and top-level finding before editing; automatically fix only findings supported by the code and established task assumptions; validate, commit, and push fixes; then process reviews until CodeRabbit has no valid unaddressed feedback.
---

# Automated PR Review Loop

## Purpose

Carry an existing pull request through automated review. Review triggering depends on the PR's base branch:

- When the base branch is `main`, Codex and CodeRabbit automatically review the PR when it is created and automatically run subsequent reviews after every new commit. Do not post a manual review-trigger comment except when recovering from a confirmed rate limit.
- When the base branch is not `main`, Codex and CodeRabbit do not automatically run reviews. Post the exact comment `@coderabbitai review` to trigger CodeRabbit for the current head, including after every new commit. The comment does not trigger Codex, so do not wait for Codex unless a Codex review is actually present.

Treat every reviewer comment as a hypothesis. Independently verify and adjudicate it against the current code before editing. The goal is to resolve valid problems, not blindly satisfy bots. Do not implement false positives, duplicates, obsolete findings, subjective preferences, or suggestions that contradict established requirements. Do not pause for user approval between adjudication and implementation: automatically fix findings supported by the evidence and leave the rest unchanged with a concise reason. Once fixes have been made, commit and push the changes and trigger the next round of review based on the base being `main` or not. Track the current PR head and trigger at most once per head. Run the loop (trigger -> wait for CodeRabbit and other checks to finish -> review and adjudicate comments -> fix valid ones -> commit and push) until CodeRabbit has found no actionable issues and all PR checks are green. If resolving a finding requires a new or conflicting product, UX, database, security, compatibility, or public-contract decision, or cannot be verified safely, escalate to the user and pause auto PR.

Treat CodeRabbit's **Prompt for all review comments with AI agents** as the authoritative source for its findings. Do not spend tokens re-fetching and correlating its individual threads unless the aggregate prompt is unavailable or lacks context required to adjudicate a specific finding. Consider nitpicks too, but validate them before fixing. Do not mark comments as resolved; CodeRabbit will do this as they are addressed.

How to wait for review bots: A large part of time in the loop will be spent waiting for the review bots to finish their reviews. Use terminal sleep methods and polling in order to wait for the checks to run in a token efficient manner. However, when CodeRabbit hits a rate limit, use the Codex automation scheduler to set up a wake point for when the reported limit resets. This way you don't have to poll and keep a terminal process active and burn tokens for waiting for long durations. Make sure once the rate limit is resolved to clean up the automation that you created.

Finish by reporting the final head, fixes, rejected findings, checks, and residual blockers.
