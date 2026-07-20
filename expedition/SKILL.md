---
name: expedition
description: Execute an Atlas GitHub epic through persistent Codex worker tasks and a validated draft stacked-PR workflow rooted in a dedicated feature branch. Use when the user wants an orchestrator to traverse a decision-complete issue dependency map, launch one worktree task per implementation leaf, require each worker to run self-review-loop after implementation, review exact PR heads, restack descendants after upstream changes, perform integrated and Computer Use design acceptance, and continue until the epic is implemented without merging PRs. Automated Codex and CodeRabbit review is an optional worker-only loop at the user's discretion because repeated bot review can hit GitHub rate limits.
---

# Expedition

## Purpose

Execute a decision-complete Atlas map. The orchestrator owns the epic, dependency graph, dedicated feature branch, stack, shared environments, cross-PR coherence, and acceptance. Each persistent worker owns exactly one implementation issue and draft PR.

If an issue requires new research or an unstated product, architecture, data, security, compatibility, or UX decision, stop and return to the user or Atlas. Expedition executes the route; it does not silently redesign it.

## Stack Topology

Create one dedicated feature branch for the whole Expedition from the exact original target head. Root the implementation stack there rather than directly on the repository's default or release branch:

```text
original target <- expedition feature branch <- first leaf <- next dependent leaf
```

The first leaf PR targets the Expedition feature branch. Each dependent leaf PR targets its accepted parent leaf branch; independent roots may each target the feature branch. The feature branch is the eventual single integration path back to the original target. Do not bypass it by retargeting the first leaf to the original target.

## Execution Loop

1. **Load the map and goal.** Read the epic's `## Expedition Goal`, descendants, parent/sub-issue relationships, blocked-by edges, linked PRs, repository instructions, and current branch state. When the user explicitly invokes Expedition, initialize the thread's active goal from that exact objective and its completion conditions when a goal mechanism is available; do not silently reinterpret or broaden it. Verify the graph is acyclic and each executable leaf is a one-PR contract. Identify the original target branch and exact head, then create and push one clearly named Expedition feature branch from that head before launching workers. If the proposed feature branch already exists, verify its ownership, ancestry, and exact head instead of overwriting or repurposing it.
2. **Choose work.** Native dependencies define ordering. A leaf is merge-ready when its blockers are closed; it may be stack-ready earlier when every blocking implementation leaf has an accepted exact PR head that can serve as its base. Use this distinction to build stacked PRs without misrepresenting GitHub issue state.
3. **Launch one persistent worker.** Create a separate Codex task in a new worktree for the leaf. Give it the issue contract, original target, Expedition feature branch, exact starting base, branch and PR base, exclusions, repository instructions, required skills, validation, and ownership boundaries. For the first leaf, explicitly set the PR base to the Expedition feature branch. Reuse that same task for every fix round.
4. **Worker delivery.** The worker implements only its issue and runs relevant validation. After completing the implementation, it must run the `self-review-loop` skill, fix validated findings, and rerun affected validation before committing, pushing, and creating or updating a draft PR whose body uses `Closes #N` for that leaf. It must verify the PR's base branch and draft state after creation and must not mark it ready for review. If the user opted into automated review, the same worker then runs `automated-pr-review-loop` and owns every resulting fix round. It returns the exact head SHA, changed surface, validation and review evidence, assumptions, and residual gaps.
5. **Orchestrator review.** Independently inspect the complete diff, issue-contract compliance, test evidence, security/data implications, integration with accepted ancestors, and stack ancestry. Batch validated findings back to the same worker, then re-review its new exact head until clean.
6. **UI acceptance.** For user-visible changes, the orchestrator owns the running environment and performs design and interaction review through Computer Use against the actual app. Check relevant desktop and narrow widths, themes, keyboard/focus behavior, overflow, states, workflows, visual hierarchy, and coherence with the integrated product. Return findings to the same worker.
7. **Accept and advance.** Record the accepted exact head and linked draft PR. Build integrated test state only from accepted canonical heads while preserving the published PR bases and dedicated feature-branch boundary. Do not close the implementation issue merely because its PR is accepted; its closing keyword handles closure when merged.
8. **Propagate upstream changes.** If an accepted ancestor changes, mark every affected descendant stale, restack it onto the new canonical head, resolve conflicts deliberately, and rerun affected worker, orchestrator, and integrated validation before accepting the suffix again.
9. **Repeat.** Run independent stack-ready leaves in parallel only when their code surfaces and provisional bases are safe. Continue until every leaf has an accepted draft PR and the combined stack passes final integrated acceptance. Report the eventual promotion order as leaf PRs into the Expedition feature branch, then the feature branch into the original target; do not perform those merges without explicit user authorization.

## Authority and Safety

- Workers do not modify other leaves, manage shared deployments, perform destructive data actions, merge PRs, or replace orchestrator acceptance.
- The orchestrator owns shared dev servers, deployments, migrations, data-safety gates, combined-branch testing, and cross-PR decisions. Require explicit user approval for destructive or production actions.
- Every PR created by Expedition is a draft, including any eventual feature-branch PR to the original target. Never mark one ready unless the user explicitly overrides this rule for that PR or run.
- Never target the first leaf PR at the original target branch. The dedicated Expedition feature branch is the required base and eventual integration boundary.
- Do not create replacement workers for review fixes. Preserve task context by steering the original persistent worker.
- The `automated-pr-review-loop` skill is an optional worker-only post-PR gate. The orchestrator must never run or take over that loop. Tell the existing worker to run it only when the user opts in for the relevant PR or Expedition; do not infer consent or enable it by default because repeated Codex and CodeRabbit review rounds can hit GitHub review-bot rate limits.
- Never merge PRs unless the user explicitly authorizes merging. Mark the active goal complete only after every leaf has an accepted exact head and required integrated acceptance is complete. Finish with the epic, PR stack, exact accepted heads, validation dispositions, unresolved blockers, and residual risks.
