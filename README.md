# fable-skill

**By [Muhammad Zakarya](https://github.com/zakarya526)** · MIT · works in [70+ AI coding agents](https://github.com/vercel-labs/skills)

> **The goal:** spend your best model's judgment *once*. Fable 5 reads a codebase deeply and writes plans precise enough that a capable executor (Opus 4.8) — or any agent, or a human — can build them safely. The skill never edits your code itself.

```
you          →  /fable-skill                (Fable 5 audits & advises)
plans/       →  001-fix-n-plus-one.md        (self-contained specs)
other agent  →  implements, tests, ships     (Opus 4.8 executes)
```

## Install

```bash
npx skills add zakarya526/fable-skill                  # pick your agents (space to select)
npx skills add zakarya526/fable-skill -a claude-code   # straight into Claude Code
```

## Usage

```
/fable-skill                  full audit → ranked findings → plans
/fable-skill quick | deep     cheaper / more exhaustive pass
/fable-skill security         focus one category (also: perf, tests, bugs…)
/fable-skill branch           audit only what your branch changes
/fable-skill next             feature ideas grounded in the repo
/fable-skill plan <desc>      spec one thing, skip the audit
/fable-skill execute <plan>   dispatch an executor, review its diff
/fable-skill reconcile        refresh the backlog
/fable-skill … --issues       also publish plans as GitHub issues
```

## How it works

1. **Recon** — maps the stack and the exact build/test/lint commands (they become verification gates).
2. **Audit** — parallel subagents across 9 categories; every finding cites `file:line`.
3. **Vet** — re-reads each finding to drop false positives before you see them.
4. **Prioritize** — ranks by leverage (impact ÷ effort, weighted by confidence); you pick what to plan.
5. **Plan** — one self-contained file per finding in `plans/`, with steps, commands, and STOP conditions ([example](./examples/001-extract-shadow-config-resolution.md)).

## Why the model split

Fable 5 — the frontier planner — does the judgment that compounds (understanding, vetting, specifying). A capable executor (Opus 4.8; Sonnet 4.6 for lighter work) implements the spec in an isolated worktree, and Fable 5 reviews the diff. You get frontier judgment where it matters and a dedicated builder where the work is already specified.

## Guarantees

- **Never edits your source** — the only writes go to `plans/`; executors work in disposable worktrees, and merging is always yours.
- **Never mutates your working tree** — read-only analysis only.
- **Never prints secrets** — locations and rotation advice only.

## Credits & License

A Fable 5 rebrand of [`improve`](https://github.com/shadcn/improve) by [shadcn](https://github.com/shadcn) — original concept and architecture are shadcn's, used under MIT.

MIT © Muhammad Zakarya. Original work © shadcn — see [LICENSE.md](./LICENSE.md).
