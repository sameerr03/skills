---
name: discuss
description: >
  Discuss and explore a feature or technical decision with the user before writing
  any code. Use this skill whenever the user says "let's discuss", "let's think about",
  "how should we build", "what's the best approach", "I want to talk through", or any
  variation of wanting to explore ideas before implementation. Also trigger when the
  user types /discuss followed by a topic. This is the pre-planning phase — explore
  the codebase, evaluate trade-offs, and help the user arrive at a clear technical
  direction before any code gets written.
---

# Discuss

Explore a feature or technical decision with the user. This is the thinking phase that comes before planning and implementation. No code gets written here — the goal is to arrive at a clear, shared understanding of what to build and how.

## Approach

1. **Understand the context** — read the relevant parts of the codebase. Understand the existing architecture, patterns, and constraints before forming opinions. You can't have a useful discussion about how to build something if you don't know what already exists.

2. **Explore the problem space** — ask clarifying questions using AskUserQuestion. What are the requirements? What are the constraints? Who are the users? What does success look like? Don't assume — interview the user to fill in the gaps.

3. **Evaluate approaches** — if the user has a specific idea, examine it honestly. What are the trade-offs? What could go wrong? What would need to change in the existing code? If they don't have a direction yet, suggest 2-3 concrete approaches with clear pros/cons for each. Be opinionated — "it depends" without context isn't helpful.

4. **Go deep on the chosen direction** — once the user leans toward an approach, dig into the specifics. What files need to change? What new abstractions are needed? What existing patterns should be followed? What are the edge cases? This is where the discussion becomes actionable.

5. **Summarize the decision** — before ending the discussion, confirm the agreed direction so there's a clear handoff to the planning/implementation phase.

## What this is NOT

- Not a planning session — don't create task lists or implementation plans
- Not a code review — don't write or critique existing code
- Not a lecture — this is a dialogue, ask questions and respond to the user's thinking
- Not a rubber stamp — push back if you see problems with an approach, even if the user seems committed to it

## Output

Keep everything conversational. No code blocks, no file writes. The output of this phase is alignment between you and the user on what to build and roughly how — the plan and implementation come next.
