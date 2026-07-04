---
name: handoff
description: Create one or more new Codex threads with discussion-first starting prompts for continuing a task in fresh context. Use when the user asks for a handoff, continuation thread, fresh-context thread, new-thread prompt, parallel follow-up threads, or wants to move an ongoing feature discussion, investigation, review, or task framing into separate Codex threads before planning or implementation.
---

# Handoff

## Goal

Create new Codex thread(s) that continue the work with fresh context. The default
output is not a copy-paste prompt. Use Codex thread tools to create the thread(s),
send each initial prompt, and title them so the user can open the right one.

The new thread must start in discussion and investigation mode. It should reset
the goal of that thread around understanding the problem fully, inspect the
relevant codebase or artifacts, and discuss trade-offs with the user before
implementation. It must not begin by editing files or carrying out a plan.

## Thread Creation Workflow

1. Check and announce thread capability:
   - First determine whether thread tools such as `list_projects`,
     `create_thread`, and `set_thread_title` are available.
   - Tell the user briefly whether you can create the thread(s) directly from
     your tools before doing so.
   - If the tools are unavailable, fall back to fenced prompts and say the user
     must create the thread(s) manually.

2. Identify the thread set:
   - Determine whether the user needs one thread or several parallel threads.
   - Give each thread a concrete purpose tied to the eventual implementation or
     investigation outcome.
   - If multiple threads are useful, keep their scopes independent enough that
     each thread can reason cleanly.

3. Resolve where each thread should run:
   - Use `list_projects` before `create_thread` for repo-scoped work.
   - Select the project whose label/path matches the codebase or project for the
     thread. The user's projects are expected to have one environment each, so
     choose the matching project environment rather than a generic or projectless
     target.
   - Do not create repo-scoped work in a projectless thread unless no matching
     project exists and the user explicitly accepts that fallback.
   - If the user's request explicitly says local/main checkout or worktree, use
     that.
   - If there is exactly one thread for one codebase and no parallel coding risk,
     default to the local/main project environment.
   - If two or more threads will work in parallel in the same codebase, prefer
     worktrees for all but any explicitly local coordination thread.
   - If threads target different codebases, local/main is acceptable for a single
     thread in each codebase unless the user asks for isolation.
   - If the number and purpose of threads are known but local vs worktree is not
     clear, stop and ask the user to choose local or worktree for each thread.
     This is a blocking question; do not create the threads until answered.

4. Build each starting prompt:
   - Address the new agent directly.
   - State the task and why it matters.
   - Include only the context needed to begin well: relevant repo, product goal,
     decisions already made, hard constraints, important evidence, rejected
     approaches, and open questions.
   - Explicitly instruct the new thread to begin by understanding and discussing,
     not implementing.
   - Tell it to inspect the current repo/codebase state before forming strong
     conclusions.
   - Tell it to ask or answer the highest-leverage questions with the user before
     proposing a plan.
   - Include known file paths, commands, issues, PRs, or branches only when they
     are important and likely current.

5. Create the thread(s):
   - Use `create_thread` with the chosen project target and environment.
   - Do not override the model or reasoning effort unless the user explicitly
     asks.
   - After creating each thread, use `set_thread_title` when available.
   - In the final response, emit `::created-thread{threadId="..."}` for each
     successfully created thread.

6. Fallback only when tooling is unavailable:
   - If thread tools are unavailable, return the prompt(s) in fenced code blocks
     and say that the user must start the thread manually.

## Starting Prompt Shape

Use this shape by default. Adapt it to the task, but preserve the discussion-first
instruction.

```text
You are continuing a task from another Codex thread. Start by resetting this
thread's goal around understanding the problem fully before implementation.

Task:
...

Context:
...

What is already known or decided:
...

Constraints and preferences:
...

Open questions and trade-offs:
...

How to begin:
1. Restate the task in your own words.
2. Inspect the relevant codebase, files, docs, issues, or current local state.
3. Build a clear mental model of the existing system and the change needed.
4. Discuss the problem, risks, and options with me before proposing an
   implementation plan.

Do not start implementing immediately. Do not edit files until we have discussed
and refined the approach together.
```

## Thread Titles

Name threads for the eventual implementation or investigation outcome, not for
the discussion process. Avoid words like "discuss", "discussion", "brainstorm",
or "handoff" in titles unless the user explicitly asks.

Good title patterns:

- `Portfolio scoring cache fallback`
- `Broker connection callback contract`
- `Principles catalog sync cron`
- `Review import pipeline edge cases`

## Writing Rules

- Prefer creating threads over producing pasteable prompts.
- Keep prompts concise enough to orient the new thread, usually 200-700 words.
- Preserve user wording for hard constraints, preferences, and product decisions.
- Separate confirmed facts from inferences.
- Do not include private chain-of-thought, irrelevant transcript excerpts, or
  stale filesystem details.
- Do not invent completed work, validation, issue numbers, or decisions.
- Do not make the new thread implement immediately unless the user explicitly
  asks for an implementation-first handoff.
- If the user asks for multiple threads, be explicit about which codebase,
  target environment, and purpose each thread gets.
