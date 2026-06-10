# fable-skill

**By [Muhammad Zakarya](https://github.com/zakarya526)** · MIT · works in [70+ AI coding agents](https://github.com/vercel-labs/skills)

> **The goal:** spend your best model's judgment *once*. Fable 5 understands a codebase deeply and writes plans precise enough that a capable executor (Opus 4.8) — or any other agent, or a human — can build them safely. The skill itself never edits your code.

An agent skill that points **Fable 5** at any codebase, audits it, and writes implementation plans for other agents to execute.

The idea: use Fable 5 — the high-ceiling planner — for the part where intelligence compounds (understanding the codebase, judging what's worth doing, writing the spec) and hand execution to a capable executor model that just follows the spec. The skill never implements anything itself. The plan is the product.

```
you          →  /fable-skill             (Fable 5 advises)
plans/       →  001-fix-n-plus-one.md       (self-contained specs)
other agent  →  implements, tests, ships    (Opus 4.8 executes)
```

## Install

```bash
npx skills add zakarya526/fable-skill
```

The installer (from [`vercel-labs/skills`](https://github.com/vercel-labs/skills)) lets you choose which of 70+ agents to install into — move with ↑↓, **press space to select**, enter to confirm. To install straight into **Claude Code** and skip the picker:

```bash
npx skills add zakarya526/fable-skill -a claude-code
```

Works in any agent that supports the [Agent Skills](https://agentskills.io) format. The plans it writes are plain markdown, so any agent (or human) can pick them up.

## Usage

```
/fable-skill                        full audit → prioritized findings → plans
/fable-skill quick                  cheap pass: hotspots, top findings only
/fable-skill deep                   exhaustive: every package, every category
/fable-skill security               focused audit (also: perf, tests, bugs, ...)
/fable-skill branch                 audit only what the current branch changes
/fable-skill next                   feature suggestions — where to take the project
/fable-skill plan <description>     skip the audit, spec one thing
/fable-skill review-plan <file>     critique and tighten an existing plan
/fable-skill execute <plan>         dispatch a capable executor, review its work
/fable-skill reconcile              refresh the backlog: verify, unblock, retire
/fable-skill ... --issues           also publish plans as GitHub issues
```

## How to use

A typical first run, start to finish:

1. Open your agent in the repo and run `/fable-skill` (or `/fable-skill quick` to keep it cheap).
2. It maps the repo, audits it, and comes back with a findings table. Reply with the ones you want planned — "plan 1, 3 and 5".
3. Plans land in `plans/` — one file each, plus an index with the recommended order. Read them; they're meant to be reviewed.
4. Hand a plan to any agent ("implement plans/001-*.md"), or let the skill run it: `/fable-skill execute 001`. It dispatches a capable executor (Opus 4.8) in an isolated worktree, reviews the diff against the plan, and reports back with a verdict. Merging stays up to you.
5. Next session, run `/fable-skill reconcile` to clean up the backlog: verify what landed, refresh what drifted, unblock what got stuck.

Before a PR, `/fable-skill branch` does the same thing scoped to just what your branch changes.

## Why Fable 5

The skill is built on a tiered-model bet: spend the frontier model where judgment compounds, and delegate the well-specified implementation to a capable executor.

- **Fable 5 plans.** Understanding a large codebase, vetting findings, and writing specs precise enough for another model to follow without context is exactly the work that rewards a frontier model.
- **A capable model executes.** `execute` dispatches an **Opus 4.8** subagent (Sonnet 4.6 for lighter, mechanical plans) in an isolated worktree; Fable 5 stays in the advisor/reviewer seat and never spends its frontier judgment typing out the diff.

You get frontier-quality judgment on the part that matters, and a capable, dedicated executor on the part that's already fully specified.

## Example

A run against a typical TypeScript repo comes back with findings like:

```
| # | Finding                                        | Category  | Effort | Confidence |
|---|------------------------------------------------|-----------|--------|------------|
| 1 | shadow-config duplicated in search.ts/view.ts, | tech-debt | M      | HIGH       |
|   | copies already drifted (TODO at search.ts:31)  |           |        |            |
| 2 | O(n²) icon migration (migrate-icons.ts:168)    | perf      | S      | HIGH       |
```

…and rejects a few, with reasons recorded so they don't come back next run:

```
- [SEC-01] https_proxy env var "SSRF": by-design — standard proxy convention,
  every CLI honors it. Not a finding.
```

Picking #1 produces a plan like [this one](./examples/001-extract-shadow-config-resolution.md) — current code excerpted, exact steps, the repo's own test/lint commands as verification gates, and STOP conditions for when reality doesn't match.

## How it works

**Recon.** Maps the repo: stack, conventions, and the exact build/test/lint commands — these become verification gates in every plan.

**Audit.** Fans out parallel subagents across nine categories: correctness, security, performance, test coverage, tech debt, dependencies & migrations, DX, docs, and direction (feature suggestions — every one must cite evidence from the repo itself, no generic idea-slop). Every finding carries `file:line` evidence, impact, effort, and confidence.

**Vet.** Subagents over-report, so the advisor re-reads every cited location itself before showing you anything — false positives get dropped, wrong attributions get corrected, rejections get recorded.

**Prioritize.** Findings land in a table ordered by leverage (impact ÷ effort, weighted by confidence). You pick what becomes plans.

**Plan.** One file per selected finding, written into `plans/` with an index, priority order, and dependency graph.

## What makes the plans executable

Plans are written for the weakest plausible executor — a model that has never seen the advisor session and may be much smaller. Three properties carry that:

- **Self-contained.** All context is inlined: exact file paths, current-state code excerpts, repo conventions with an exemplar file, verified commands. No "as discussed above."
- **Verification gates.** Every step ends with a command and its expected output. Done criteria are machine-checkable. The executor never has to judge whether it succeeded.
- **Hard boundaries.** Explicit out-of-scope lists, and STOP conditions — "if X, stop and report" — instead of letting a small model improvise when reality doesn't match the plan.

Each plan also stamps the git commit it was written against, so executors run a mechanical drift check before touching anything.

## Closing the loop

Plans aren't fire-and-forget:

- **`execute <plan>`** spawns a capable executor subagent (Opus 4.8) in an isolated git worktree, hands it the plan, then reviews the result like a tech lead — re-runs every done criterion, checks scope compliance, reads the diff against intent. Verdict: approve (merging stays your call), send back for revision (max 2 rounds), or block and refine the plan.
- **`reconcile`** processes what happened since: verifies DONE plans still hold, investigates BLOCKED ones and rewrites around the obstacle, refreshes drifted plans, retires findings that got fixed independently.
- **`--issues`** publishes plans as GitHub issues — same self-contained body, so any agent or human can pick them up where work already lives.

## Hard rules

- Never modifies source code itself. The only writes go to `plans/`; executors edit only in disposable worktrees, and merging is always yours.
- Never runs commands that mutate your working tree — read, search, and read-only analysis only.
- Never reproduces secret values. Locations and credential types only, rotation always recommended.
- Asked to implement? It declines and points at the plan (or offers `execute`).

## Credits

`fable-skill` is a Fable 5 retargeting and rebrand of the excellent [`improve`](https://github.com/shadcn/improve) skill by [shadcn](https://github.com/shadcn), used under the MIT License. Original concept and architecture are shadcn's; this fork retargets the model economics around Fable 5 and rebrands the packaging.

## License

MIT © Muhammad Zakarya. Original work © shadcn — see [LICENSE.md](./LICENSE.md).
