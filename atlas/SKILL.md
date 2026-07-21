---
name: atlas
description: Research and shape a large, long-running task through open-ended alignment, lightweight ATLAS.md notes, and a visual GitHub issue map. Use when work is too large for one reviewable PR or agent session and needs outcome-focused implementation leaves, native dependencies, GitHub-first review, and an execution-ready handoff to Expedition. Publish the draft map for review, refine it on GitHub, verify it, then stop without implementing it.
---

# Atlas

## Purpose

Turn a large objective into an outcome-complete, dependency-complete, reviewable GitHub execution map. Atlas charts the route; it does not implement it.

Atlas must establish the end-to-end outcome, operating model, important constraints, consequential technical decisions, review points, and sensible PR boundaries. It must not prescribe ordinary implementation details that the future Expedition orchestrator can decide from the live code and the user's intent.

`ATLAS.md` is a lightweight research and discussion notebook. Once a draft issue map is published, GitHub becomes the review surface and source of truth. Never maintain two authoritative maps.

## Lifecycle

1. **Align.** Inspect the existing system and discuss the objective, current behavior, desired end-to-end workflow, trust boundaries, realistic failure cases, scope, and non-goals.
2. **Shape the map.** Propose independently reviewable implementation leaves, their observable outcomes, dependencies, and integrated acceptance path.
3. **Publish for review.** With explicit permission, publish the epic and leaves to GitHub as an under-review map, including native relationships and a Mermaid dependency diagram in the epic.
4. **Refine and accept.** Apply the user's issue-level feedback, verify the complete graph, and only after explicit acceptance mark it ready for Expedition and freeze `ATLAS.md` to a pointer.

Create `ATLAS.md` at the repository root unless the user chooses another location. Start it minimally with the objective, current understanding, scope, non-goals, accepted assumptions, open questions, and a preliminary issue map. Update it after substantive research or decisions so the discussion survives compaction; do not turn it into a polished duplicate of the GitHub issues.

Define one **Expedition Goal** as a concrete outcome statement. Include completion conditions: every implementation leaf has an accepted exact PR head, required integrated acceptance passes, blockers and residual risks are reported, and no PR is merged without explicit user authorization.

## Discussion and Research

- Inspect the codebase, architecture, history, documentation, existing issues, and external contracts before asking the user questions the repository can answer.
- Use an open-ended conversation. Periodically reflect the complete understanding and invite correction; do not run an exhaustive decision-tree interview or turn ordinary technical suggestions into a sequence of recommended yes/no approvals.
- Establish the plain-language operator or user journey before proposing mechanisms. Identify where data or behavior becomes trusted and which realistic failures must be prevented.
- Push back with concrete evidence and the smallest necessary safeguard. Do not manufacture requirements from generic edge cases, hostile threat models, or speculative future uses that contradict the agreed operating model.
- Resolve product decisions and cross-leaf contracts during Atlas. Keep consequential uncertainty explicit rather than publishing it as a settled implementation requirement.

Each leaf must explicitly define consequential technical changes it owns. These include schema and type changes, migrations and backfills, authentication or authorization changes, public APIs and external contracts, persistent state, service boundaries, new dependencies or providers, destructive or irreversible data behavior, and any other choice that materially affects data safety or multiple leaves.

Leave ordinary internal implementation choices to the future Expedition orchestrator. The orchestrator owns the worker prompt because it retains the planning context and user intent. A worker executes that bounded plan and its self-review loop; it must not independently redesign the leaf or expand its scope.

## Design the Map

- Break the end-to-end workflow into coherent outcomes, not merely codebase layers. Prefer vertical, demonstrable slices.
- Make every implementation leaf one independently understandable, testable, and reviewable PR with a meaningful capability, artifact, interface, report, or behavior at its branch head.
- Allow dependent leaves when each boundary still has standalone verification and a stable output contract. Do not split backend, frontend, schema, or infrastructure mechanically when the resulting branch has no useful review surface.
- Map one implementation leaf to one persistent Expedition worker task and one PR. Use grouping issues only when they materially improve navigation; grouping issues are not PR units.
- Define native blocked-by relationships, identify unblocked leaves, reject cycles, and ensure no leaf is orphaned from the epic.

Each leaf should state only what execution and review need:

- outcome and why the leaf exists;
- inputs, outputs, and user review surface;
- acceptance criteria and mechanical or human validation;
- dependencies and integrated role;
- consequential technical decisions;
- explicit exclusions and assumptions; and
- conditions requiring a return to Atlas.

Mention affected code surfaces when research makes them useful, but do not predict exact files, abstractions, or line counts merely to make the plan appear complete. Lines of code are not a scope target.

## Control Complexity

- Reuse existing modules, state, schemas, services, and dependencies before adding concepts.
- Introduce a new schema concept, persistent state layer, service boundary, provider, dependency, compatibility path, security mechanism, or versioning system only when an explicit requirement, existing contract, realistic failure, or cross-leaf interface justifies it.
- Treat the user's operating assumptions as design constraints. Do not build machinery to defend against scenarios those assumptions exclude unless codebase evidence shows a concrete correctness risk.
- Perform a subtraction review of the map: seek the same outcome with fewer concepts, fields, layers, dependencies, compatibility paths, validators, and PRs while preserving meaningful verification.
- Return to Atlas when execution discovers a consequential technical change or issue boundary that the accepted map did not authorize.

## Publish and Review on GitHub

After enough alignment to produce a credible map, present a compact overview and obtain explicit permission to publish it for review. Do not require the user to approve a fully expanded `ATLAS.md` first.

1. Create the epic with `## Expedition Goal`, scope, non-goals, operating model, an `Atlas status: Under review` marker, and a Mermaid dependency map that visually keys every implementation leaf.
2. Create the implementation issues with their leaf contracts, then attach native parent/sub-issue and blocked-by relationships. Native relationships remain authoritative; keep the Mermaid diagram synchronized as the visual overview.
3. Let the user review the published GitHub issues directly. Revise issue bodies, boundaries, hierarchy, dependencies, and the epic diagram from that feedback.
4. Read the graph back from GitHub and verify the goal, titles, bodies, hierarchy, dependencies, cycles, orphaned leaves, and diagram against the accepted map.
5. After explicit acceptance, mark the epic `Atlas status: Ready for Expedition`. Replace `ATLAS.md` with a small frozen pointer containing the title, publication date, epic link, and statement that GitHub is the execution source of truth.
6. Report the epic link and a concise named overview of the verified map.

## Boundaries

- Atlas may research, inspect, discuss, maintain `ATLAS.md`, and publish or revise the issue map with authorization. It must not edit product code, create branches or PRs, launch implementation workers, or execute the map.
- Detailed implementation contracts live in exactly one issue rather than being copied across the epic and children.
- Do not create research tickets as a substitute for planning. Research needed to make the graph credible belongs in Atlas.
- Do not commit or push the planning notebook unless the user explicitly asks. Expedition reads GitHub after acceptance.
- When the destination is small enough for one reviewable PR, say that an Atlas map is unnecessary and ask how the user wants to proceed.
