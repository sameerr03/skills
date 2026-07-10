---
name: expedition
description: Execute an Atlas GitHub epic through persistent Codex worker tasks and a validated stacked-PR workflow. Use when the user wants an orchestrator to traverse a decision-complete issue dependency map, launch one worktree task per implementation leaf, coordinate parallel and sequential work, review exact PR heads, restack descendants after upstream changes, perform integrated and Computer Use design acceptance, and continue until the epic is implemented without merging PRs. Automated Codex and CodeRabbit review is opt-in, not part of the default loop.
---

# Expedition

## Purpose

Execute a decision-complete Atlas map. The orchestrator owns the epic, dependency graph, stack, shared environments, cross-PR coherence, and acceptance. Each persistent worker owns exactly one implementation issue and PR.

If an issue requires new research or an unstated product, architecture, data, security, compatibility, or UX decision, stop and return to the user or Atlas. Expedition executes the route; it does not silently redesign it.

## Execution Loop

1. **Load the map.** Read the epic, descendants, parent/sub-issue relationships, blocked-by edges, linked PRs, repository instructions, and current branch state. Verify the graph is acyclic and each executable leaf is a one-PR contract.
2. **Choose work.** Native dependencies define ordering. A leaf is merge-ready when its blockers are closed; it may be stack-ready earlier when every blocking implementation leaf has an accepted exact PR head that can serve as its base. Use this distinction to build stacked PRs without misrepresenting GitHub issue state.
3. **Launch one persistent worker.** Create a separate Codex task in a new worktree for the leaf. Give it the issue contract, exact starting base, branch and PR base, exclusions, repository instructions, required skills, validation, and ownership boundaries. Reuse that same task for every fix round.
4. **Worker delivery.** The worker implements only its issue, runs relevant validation and the self-review-loop skill, commits, pushes, and creates or updates a ready PR whose body uses `Closes #N` for that leaf. It returns the exact head SHA, changed surface, validation evidence, assumptions, and residual gaps.
5. **Orchestrator review.** Independently inspect the complete diff, issue-contract compliance, test evidence, security/data implications, integration with accepted ancestors, and stack ancestry. Batch validated findings back to the same worker, then re-review its new exact head until clean.
6. **UI acceptance.** For user-visible changes, the orchestrator owns the running environment and performs design and interaction review through Computer Use against the actual app. Check relevant desktop and narrow widths, themes, keyboard/focus behavior, overflow, states, workflows, visual hierarchy, and coherence with the integrated product. Return findings to the same worker.
7. **Accept and advance.** Record the accepted exact head and linked PR. Advance any combined integration branch only through accepted canonical heads. Do not close the implementation issue merely because its PR is accepted; its closing keyword handles closure when merged.
8. **Propagate upstream changes.** If an accepted ancestor changes, mark every affected descendant stale, restack it onto the new canonical head, resolve conflicts deliberately, and rerun affected worker, orchestrator, and integrated validation before accepting the suffix again.
9. **Repeat.** Run independent stack-ready leaves in parallel only when their code surfaces and provisional bases are safe. Continue until every leaf has an accepted PR and the combined stack passes final integrated acceptance.

## Authority and Safety

- Workers do not modify other leaves, manage shared deployments, perform destructive data actions, merge PRs, or replace orchestrator acceptance.
- The orchestrator owns shared dev servers, deployments, migrations, data-safety gates, combined-branch testing, and cross-PR decisions. Require explicit user approval for destructive or production actions.
- Do not create replacement workers for review fixes. Preserve task context by steering the original persistent worker.
- Do not automatically run the automated-pr-review-loop skill or poll Codex/CodeRabbit. The user may invoke that skill separately when desired.
- Never merge PRs unless the user explicitly authorizes merging. Finish with the epic, PR stack, exact accepted heads, validation dispositions, unresolved blockers, and residual risks.
