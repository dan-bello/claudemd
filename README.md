# CLAUDE.md

A portable `CLAUDE.md` — the standing methodology file I drop into every repo I use with [Claude Code](https://www.anthropic.com/product/claude-code). Sharing it here so it's easy to grab, and in case anyone else finds it useful.

> This is a living document — I update it as my workflow evolves. Pin to a commit if you want stability.

## What it is

A compact set of defaults covering the parts of working with Claude Code that benefit from being written down once:

- branch and workflow rules
- accuracy and implementation standards
- a propose-first gate for non-trivial changes
- a reusable health-check routine
- documentation hygiene and tone

It's opinionated and terse on purpose — guardrails that help without getting in the way.

## How to use it

1. Copy `CLAUDE.md` into the root of your repo.
2. (Recommended) Create a `PROJECT.md` next to it with project-specific context: build and test commands, env vars, component quirks, deliberate trade-offs.
3. Commit both. Claude Code reads them on every session.

## Why PROJECT.md

`CLAUDE.md` intentionally stays generic so it's portable across every repo. Anything project-specific — the test command, the deploy process, the weird edge case in module X — belongs in `PROJECT.md`. Where the two overlap, `PROJECT.md` wins. It's closer to ground truth.

## Customizing

Fork it, edit it, make it yours. The defaults reflect my preferences (e.g. `preview` as the working branch, propose-first for non-trivial changes). Swap them to fit your own workflow.

## License

MIT. Use it however you like.
