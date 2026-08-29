# dev-cycle

A Claude Code slash command that runs a coding task through a full, repeatable flow:

brainstorm -> analyze -> plan -> **(implement -> webperf check -> review/test loop)** -> report + commit messages + PR description.

It stops for your confirmation between the major steps, and loops the implement/webperf/review part until you say it's approved.

## Install

```
/plugin marketplace add <your-github-username>/dev-cycle
/plugin install dev-cycle
```

## Usage

```
/dev-cycle add a wishlist button to the product page
```

Claude will:
1. Brainstorm the approach with you.
2. Talk it through and confirm direction.
3. Write a plan and get your confirmation.
4. Implement it step by step.
5. Run a web performance check (skipped automatically for non-web changes).
6. Review the diff / hand it to you for testing.
7. If you request changes, it loops back to step 4 and repeats 4-6 until you approve.
8. Once approved, it gives you: a round-by-round report of what changed and why, commit messages for each round, and a filled-in PR description.

## Optional dependencies

This command works standalone, but it's better with these plugins installed -- if they're missing, `dev-cycle` falls back to doing the equivalent work manually instead of failing:

- [`superpowers`](https://github.com/obra/superpowers) -- brainstorming, plan writing, plan execution
- `agent-skills` -- `webperf` (performance audits) and `review` (five-axis code review)

## PR description

It looks for a PR template in your repo (`.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE.md`, `docs/pull_request_template.md`) and fills that in. If none exists, it uses a simple generic structure. The description is written in plain, casual language -- not corporate-sounding.

## Notes

- It never runs git commands (commit/push/etc.) on its own -- it just prints the commit messages for you to use.
- It respects any `CLAUDE.md` / `AGENTS.md` conventions already in your target repo.
