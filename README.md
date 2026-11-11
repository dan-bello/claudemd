# CLAUDE.md

A portable `CLAUDE.md` — the standing methodology file I drop into every repo I use with [Claude Code](https://claude.com/product/claude-code). Sharing it here so it's easy to grab, and in case anyone else finds it useful.

> This is a living document — I update it as my workflow evolves. Pin to a commit if you want stability.

## What it is

A compact set of defaults covering the parts of working with Claude Code that benefit from being written down once:

- branch and workflow rules
- accuracy, implementation, and secrets standards
- a propose-first gate for non-trivial changes
- a reusable health-check routine
- documentation hygiene and tone

It's opinionated and terse on purpose — guardrails that help without slowing you down, over-restricting, or spending tokens unnecessarily.

## How to use it

1. Copy `CLAUDE.md` into the root of your repo.
2. (Recommended) Create a `PROJECT.md` next to it with project-specific context: build and test commands, a CI-status command if your CI runs the same build and tests, env vars, component quirks, deliberate trade-offs, and whether the project is proprietary. If it's missing, Claude will offer to create one; an empty `PROJECT.md` opts out.
3. Commit both. Claude Code reads `CLAUDE.md` on every session, and `CLAUDE.md` pulls in `PROJECT.md` with an `@PROJECT.md` import, so keep the two side by side.
4. (Recommended) Pre-allow your build, test, and read-only git commands in Claude Code's permission settings. The file tells Claude to start the baseline run while it prepares a proposal, which only saves time if those commands don't stop at a permission prompt.

## Why PROJECT.md

`CLAUDE.md` intentionally stays generic so it's portable across every repo. Anything project-specific — the test command, the deploy process, the weird edge case in module X — belongs in `PROJECT.md`. Where the two overlap, `PROJECT.md` wins. It's closer to ground truth. It also loads every session, so keep it lean.

Claude keeps `PROJECT.md` accurate as the code changes, but never edits `CLAUDE.md` unless you explicitly ask.

## How it stays fast

- The full build-and-test run gates the push, not each commit. Claude iterates and commits on the narrowest test command, then runs the full suite once per push.
- The session baseline reuses a green CI result for HEAD when the tree is clean, so small changes don't pay for two full runs. Without a CI-status command in `PROJECT.md`, Claude runs the baseline locally.
- Exact-spec requests, low-risk edits, doc and comment corrections, and test-only additions skip the proposal. Everything else gets one proposal built to be approved in a single reply.

## Customizing

Fork it, edit it, make it yours. The defaults reflect my preferences (e.g. `preview` as the default working branch, propose-first for non-trivial changes, one approval covering commit and push, one full test run per push). Swap them to fit your own workflow. If you use a different default working branch, change `preview` in §1; the rest of the file refers to it as "the working branch." If your integration branch isn't `main`, change it in §1 and §5. If you want every commit green on its own, move the full-run requirement in §1 from push back to commit.

## License

MIT. See [LICENSE](LICENSE) for details.
