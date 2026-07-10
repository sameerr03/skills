---
name: atlas
description: Research and plan a large, long-running task as a verified visual GitHub issue map. Use when a task is too large for one reviewable PR or agent session and needs an epic, nested sub-issues, exact PR-sized implementation leaves, and native blocking relationships before execution. Resolve research and important decisions during planning, obtain approval before GitHub writes, create and verify the complete issue graph, then stop without implementing it.
---

# Atlas

## Purpose

Turn a large objective into a decision-complete GitHub execution map. Atlas charts the route; it does not implement it.

The map lives in GitHub's native issue hierarchy and blocked-by relationships, not a duplicated checklist or prose graph. Use names and links in human-facing summaries rather than walls of issue numbers.

## Planning

1. Establish the destination, success criteria, scope, non-goals, constraints, repository, and relevant existing issues.
2. Inspect the codebase, architecture, history, documentation, and external contracts needed to plan accurately. Conduct the user discussion, research, and important product or technical decisions now.
3. Do not publish speculative downstream work. If research is incomplete or a consequential decision remains unresolved, continue planning or stop for the user rather than turning uncertainty into an implementation issue.
4. Break the destination into coherent workstreams, then recursively split them until every implementation leaf represents one independently understandable, testable, reviewable PR. A grouping issue may contain children but is not itself a PR unit.
5. Give each implementation leaf a clear outcome, context, in-scope and out-of-scope boundaries, established technical direction, likely affected surfaces, acceptance criteria, validation requirements, assumptions, and dependencies.
6. Define native blocked-by relationships for every dependency. Explicitly identify leaves with no blockers, reject cycles, and ensure no leaf is orphaned from the epic hierarchy.

## Publish the Map

Present the complete proposed hierarchy and dependency edges to the user before creating or changing GitHub issues. After explicit approval:

1. Create the epic and all grouping and implementation issues.
2. Attach native parent/sub-issue relationships.
3. Attach native blocked-by relationships. These relationships are the execution graph; do not maintain a second manual ordering that can drift.
4. Read the graph back from GitHub and verify titles, bodies, hierarchy, dependency edges, cycles, and orphaned leaves.
5. Report the epic link and a concise named overview of the verified map.

## Boundaries

- Atlas may research, inspect, discuss, and write the approved issue map. It must not edit product code, create branches or PRs, launch implementation workers, or begin executing the map.
- The epic is a low-resolution overview and scope anchor. Detailed implementation contracts live in exactly one issue rather than being copied across the epic and children.
- Do not create separate research tickets as a substitute for planning. Research required to define the implementation graph belongs in Atlas.
- When the destination is small enough for one reviewable PR, say that an epic map is unnecessary and ask how the user wants to proceed.

