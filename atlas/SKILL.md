---
name: atlas
description: Research and plan a large, long-running task through an evolving ATLAS.md draft and a verified visual GitHub issue map. Use when a task is too large for one reviewable PR or agent session and needs an epic, nested sub-issues, exact PR-sized implementation leaves, and native blocking relationships before execution. Resolve research and important decisions during planning, publish the approved draft to GitHub, verify the complete issue graph, freeze ATLAS.md as a pointer, then stop without implementing it.
---

# Atlas

## Purpose

Turn a large objective into a decision-complete GitHub execution map. Atlas charts the route; it does not implement it.

During planning, `ATLAS.md` is the canonical draft and durable discussion tracker. After approved publication and read-back verification, GitHub's native issue hierarchy and blocked-by relationships become the execution source of truth. Never maintain two authoritative maps.

## ATLAS.md Lifecycle

Create `ATLAS.md` at the repository root unless the user chooses another location. Start minimally with the destination, Expedition goal, scope, non-goals, open questions, and an empty issue-map section. Update it after substantive research, decisions, or map changes rather than transcribing every conversation turn.

Grow the document in two stages:

1. **Design the map.** Decide grouping issues, implementation leaves, one-PR boundaries, exclusions, parent/sub-issue hierarchy, and blocking relationships.
2. **Plan each PR deeply.** Expand every implementation leaf with its outcome, context, resolved decisions, affected surfaces, acceptance criteria, validation, assumptions, dependencies, subtraction expectations, and conditions requiring a return to Atlas.

Keep uncertainty explicit. Never record an open question as a decision or publish speculative downstream work.

Define one **Expedition Goal** as a concrete outcome statement that the future orchestrator can pursue without reinterpreting the destination. Include the completion conditions: every implementation leaf has an accepted exact PR head, required integrated acceptance passes, blockers and residual risks are reported, and no PR is merged without explicit user authorization.

## Planning

1. Establish the destination, Expedition goal and completion conditions, scope, non-goals, constraints, repository, and relevant existing issues in `ATLAS.md`.
2. Inspect the codebase, architecture, history, documentation, and external contracts needed to plan accurately. Conduct the user discussion, research, and important product or technical decisions now, recording their durable outcomes.
3. If research is incomplete or a consequential decision remains unresolved, continue planning or stop for the user rather than turning uncertainty into an implementation issue.
4. Break the destination into coherent workstreams, then recursively split them until every implementation leaf represents one independently understandable, testable, reviewable PR. A grouping issue may contain children but is not itself a PR unit.
5. Complete the deep plan for every implementation leaf and define native blocked-by relationships for every dependency. Explicitly identify leaves with no blockers, reject cycles, and ensure no leaf is orphaned from the epic hierarchy.

## Implementation Constraints

Make each implementation leaf constrain code growth as well as behavior:

- Reuse existing modules, state, schemas, services, and dependencies before adding abstractions.
- Do not introduce a new schema concept, state layer, service boundary, or dependency unless Atlas has researched the need and the user has approved that architectural direction.
- Estimate the likely files touched and production-code line impact before publishing the leaf. Treat estimates as scope alarms, not quotas or implementation targets.
- Define a stop condition requiring execution to return to Atlas when the discovered work materially exceeds the estimated surface or requires an unapproved architectural concept.
- State which existing paths, abstractions, or files the change should delete, replace, or make obsolete. Say explicitly when the leaf is intentionally additive.
- Perform a subtraction review of the proposed map: ask whether the same product outcome can use fewer concepts, fields, layers, dependencies, compatibility paths, or PRs. Revise the map when the answer is yes.

## Publish the Map

Present the complete `ATLAS.md` to the user before creating or changing GitHub issues. Treat the approved version as the publication snapshot. After explicit approval:

1. Create the epic with an explicit `## Expedition Goal` section, then create all grouping and implementation issues.
2. Convert each implementation section into its issue body, then attach native parent/sub-issue and blocked-by relationships.
3. Read the graph back from GitHub and compare the Expedition goal, titles, bodies, hierarchy, dependencies, cycles, and orphaned leaves against the approved snapshot. Correct publication discrepancies before declaring the map ready.
4. Make GitHub canonical. Replace the full `ATLAS.md` with a small frozen pointer containing the Atlas title, published status and date, epic link, and a statement that GitHub is the execution source of truth.
5. Report the epic link and a concise named overview of the verified map.

## Boundaries

- Atlas may research, inspect, discuss, maintain `ATLAS.md`, and write the approved issue map. It must not edit product code, create branches or PRs, launch implementation workers, or begin executing the map.
- The epic is a low-resolution overview and scope anchor. Detailed implementation contracts live in exactly one issue rather than being copied across the epic and children.
- Do not create separate research tickets as a substitute for planning. Research required to define the implementation graph belongs in Atlas.
- Do not commit or push the planning tracker unless the user explicitly asks. After publication, Expedition reads GitHub rather than the frozen pointer.
- When the destination is small enough for one reviewable PR, say that an epic map is unnecessary and ask how the user wants to proceed.
