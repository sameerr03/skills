---
name: handoff
description: Create a concise, copy-pasteable prompt that starts a fresh Codex chat with the right discussion context for the next task. Use when the user asks for a handoff, continuation prompt, fresh-context prompt, context reset summary, new-thread prompt, or wants to move an ongoing feature discussion, investigation, or task framing into a new chat before planning or implementation.
---

# Handoff

## Goal

Produce one polished prompt the user can paste into a new chat so the next agent can start a focused discussion about the next task with the right context.

The handoff is not a transcript summary and not an implementation plan. It should tee up the next conversation: what the task is, why it matters, what context is already known, and what should be discussed first.

## Workflow

1. Identify the next task:
   - What the user wants to discuss next
   - What decision, design direction, or investigation the next chat should help with
   - Whether the next chat should stay discussion-only before implementation

2. Gather only the context needed to continue:
   - User goal and the reason for the work
   - Decisions already made
   - Constraints and explicit user preferences
   - Important discoveries, evidence, and rejected approaches
   - The recommended next task, described briefly
   - Open questions or trade-offs the next chat should discuss first

3. Include repo, branch, path, or local state only when it is essential:
   - Omit branch names, worktree paths, and current filesystem state by default.
   - Assume the user may paste the prompt into a new branch or new worktree where old branch/path details are stale.
   - Mention files, commands, or code areas only if they are important context for the discussion.

4. Write the handoff as a single prompt addressed to the next agent.

## Output Contract

Return only the prompt by default, in a fenced code block for easy copying. Add a one-line preface only if needed to explain uncertainty.

Use this structure by default:

```text
You are helping me continue from a previous discussion. Start by discussing the next task with me; do not jump straight into implementation.

Next task:
...

Context:
...

What we already decided or learned:
...

Constraints and preferences:
...

What I want from this chat:
...

Good starting questions:
...

Please begin by restating the task in your own words, then ask or answer the highest-leverage discussion questions before proposing any implementation plan.
```

Use this implementation-focused structure only when the user explicitly asks for a coding handoff:

```text
You are helping me continue implementation from a previous Codex chat.

Task:
...

Relevant context:
...

Known constraints:
...

Suggested next implementation step:
...

Please verify the current local state before editing.
```

## Writing Rules

- Be specific enough that the next agent can act without reading the old chat.
- Preserve user wording for hard constraints, preferences, and product decisions.
- Separate confirmed facts from inferences or guesses.
- Focus on the next task and the context needed to discuss it well.
- Omit branch names, worktree paths, and repo status unless the user specifically needs them.
- Include exact file paths, commands, issue numbers, PR numbers, and test results only when they matter for the next discussion.
- Mention uncertainty explicitly instead of smoothing over gaps.
- Keep it concise: usually 150-500 words, unless the discussion has many important constraints.
- Do not include private chain-of-thought, irrelevant conversational texture, or raw transcript excerpts.
- Do not invent completed work, validation, or decisions.
- Do not turn the handoff into a step-by-step implementation plan unless the user asks.

## Handoff Variants

For discussion handoffs, emphasize problem framing, options considered, trade-offs, current recommendation, and what decision remains. This is the default.

For implementation handoffs, emphasize local state, files touched, behavioral intent, test status, next edits, and known risks only when the user explicitly asks to continue coding.

For review handoffs, emphasize the diff or PR under review, review stance, already-inspected files, candidate findings, and what still needs verification.

For debugging handoffs, emphasize symptoms, reproduction steps, logs/errors, hypotheses tested, ruled-out causes, and the next highest-signal diagnostic.
