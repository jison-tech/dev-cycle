---
description: Run a task through the full flow -- brainstorm, analyze, plan, implement, webperf check, review -- looping until approved, then report + commit messages + PR description
argument-hint: <task description>
---

You are executing a task for the user through this standard flow. The task is: $ARGUMENTS

Follow these steps **in order**. Stop and wait for the user's explicit go-ahead before moving to the next step each time -- do not chain steps automatically.

Some steps below reference optional skills (from the `superpowers` and `agent-skills` plugins, if installed). If a named skill isn't available in this session, don't fail -- just do the equivalent work directly and say so ("skill X isn't installed, doing this step manually").

## Step 1 — Brainstorm
Use the `superpowers:brainstorming` skill if available to explore intent, requirements, and design options for the task. If not available, do this as plain conversation: ask what the user wants, surface options and trade-offs. Do not write any plan or code yet.

## Step 2 — Chat & analyze
Discuss the brainstorm output with the user in plain conversation: trade-offs, feasibility, what fits the existing codebase (check for `CLAUDE.md` / `AGENTS.md` conventions in the target repo, if present). This is a dialogue, not a deliverable -- keep going until the user confirms direction.

## Step 3 — Write plan
Only after the user confirms the direction from Step 2, use the `superpowers:writing-plans` skill if available to produce a concrete implementation plan; otherwise write a short plain plan yourself (files to touch, order of steps). Present it and get explicit confirmation before implementing.

## Steps 4-6 — Implement -> webperf -> test/review loop
These three steps are a **loop**, not a one-shot sequence. Repeat the cycle until the user approves -- do not treat Step 6 as the end unless the user says the changes are approved.

### Step 4 — Implement, one by one
Execute the confirmed plan (or the latest round of requested changes) incrementally -- one step/task at a time, verifying as you go. Use `superpowers:executing-plans` or `agent-skills:build` if available; otherwise just implement directly, step by step. Do not batch unrelated steps together.

### Step 5 — Web performance check
If this task touches a web frontend, run a performance check using the `agent-skills:webperf` skill if available (web-performance-auditor persona) against the changed pages/components. If the skill isn't available, do a basic manual check instead (bundle size impact, unnecessary re-renders, image optimization, lazy loading). Skip this step entirely for non-web changes (e.g. backend-only, CLI tools, docs).

### Step 6 — Review / test
Use the `agent-skills:review` skill if available (five-axis: correctness, readability, architecture, security, performance) on the current diff; otherwise do a plain self-review covering the same five axes. Also hand the change to the user for manual testing.

### Loop condition
- If the review or the user's testing produces **requested changes**: go back to Step 4 with those changes as the new work item, then re-run Step 5 and Step 6. Repeat.
- If there are **no more requested changes** and the user explicitly approves: the loop ends and the task is complete.
- Each pass through the loop should be scoped to just the requested changes from that round -- don't re-implement things that were already approved.

## Step 7 — Final report, commit message, and PR description
Only after the user gives final approval (loop above is done). Do this once, at the end, not per round.

### 7a. Round-by-round report
Keep a running log through every loop iteration (track it as you go, you'll need it here): for each round, note what was implemented and why, what the webperf check found and why anything was changed because of it, and what testing found and why anything was changed because of it. At the end, print this as a plain chronological log, oldest round first, e.g.:

```
Round 1
- Implemented: <what>
- Why: <reason / what was requested>
- Webperf: <result> -- <changed X because Y / no change needed / skipped, not a web change>
- Testing: <result> -- <changed X because Y / no change needed>

Round 2
- Implemented: <what>
- Why: <what changes were requested after round 1 and by whom/what feedback>
- Webperf: <result>
- Testing: <result>
...
```
Keep each line short and factual -- no fluff, this is a log not a narrative.

### 7b. Commit messages
Give one commit message per round/logical chunk of work (not one giant commit message), in the target repo's normal commit style (short imperative subject line, optional body only if needed). Do NOT run any git commands yourself unless the user explicitly asks you to -- just print the message(s) for the user to use.

### 7c. PR description
Look for a PR template in the target repo first (common paths: `.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE.md`, `docs/pull_request_template.md`). If found, fill it out. If the user has pointed you at a specific template file/path, use that instead. If no template exists anywhere, use a simple generic structure: Summary, What changed, Why, Testing done, Notes for reviewer.

Base the PR description on the whole task (all rounds combined, not per-round). Write it in simple, plain, human English -- like a regular developer casually filling out a PR, not corporate or overly polished. Keep every section short -- a few words to one short sentence per bullet, no long paragraphs.

## Notes
- Respect whatever conventions the target repo defines (`CLAUDE.md`, `AGENTS.md`, linting/style rules) at every step.
- If the user redirects or skips a step explicitly, follow their instruction rather than forcing the sequence.
- Never run destructive or shared-state git operations (commit, push, force-push, reset --hard, etc.) unless the user explicitly asks for that specific action.
