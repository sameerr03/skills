---
name: auto-pr
description: Use when the user asks to monitor, or get a PR ready to merge.
---

# Automated PR Review Loop

## Purpose

Carry an existing pull request through automated review. All the repos have PR Review bots. They are helpful but they may not always be right. In these repos, review triggering depends on the PR's base branch: (Current bots setup: Coderabbit)

- When the base branch is `main`, CodeRabbit automatically reviews the PR when it is created and automatically run subsequent reviews after every new commit. Do not post a manual review-trigger comment except when recovering from a confirmed rate limit.
- When the base branch is not `main`, CodeRabbit does not automatically run reviews. Post the exact comment (`@coderabbitai review`) to trigger for the current head, including after every new commit.

Treat every reviewer comment as a hypothesis. Independently verify and adjudicate it against the current code before editing. The goal is to resolve valid problems, not blindly satisfy bots. Do not implement false positives, duplicates, obsolete findings, subjective preferences, or suggestions that contradict established requirements. Do not let review feedback expand the PR beyond the user's original goal. Address real shortcomings but avoid scope creep. Do not pause for user approval between adjudication and implementation: automatically fix findings supported by the evidence and leave the rest unchanged with a concise reason.

Once fixes have been made, commit and push the changes and trigger the next round of review based on the base being `main` or not. Track the current PR head and trigger at most once per head. Run the loop (trigger -> wait for CodeRabbit and other checks to finish -> review and adjudicate new or still-unresolved comments relevant to the current head -> fix valid ones -> commit and push) until all bots have found no actionable issues and all PR checks are green. If resolving a finding requires a new or conflicting product, UX, database, security, compatibility, or public-contract decision, or cannot be verified safely, escalate to the user and pause auto PR.

Treat CodeRabbit's **Prompt for all review comments with AI agents** as the authoritative source for its findings. Do not spend tokens re-fetching and correlating its individual threads unless the aggregate prompt is unavailable or lacks context required to adjudicate a specific finding. Consider nitpicks too, but validate them before fixing. Do not mark comments as resolved that you are addressing; CodeRabbit will do this as they are addressed. For bot comments that are not being addressed, comment under the bot's comment of why not, mention that you are an agent, and mark them resolved - do not make a general comment to address an entire batch of new comments.

How to wait for review bots: A large part of time in the loop will be spent waiting for the review bots to finish their reviews. Use terminal sleep methods and polling in order to wait for the checks to run in a token efficient manner. However, when CodeRabbit hits a rate limit, use the Codex automation scheduler to set up a wake point for when the reported limit resets. This way you don't have to poll and keep a terminal process active and burn tokens for waiting for long durations. Make sure once the rate limit is resolved to clean up the automation that you created.

Finish by reporting the final head, fixes, rejected findings, checks, and residual blockers.
