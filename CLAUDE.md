# CLAUDE.md

Standing methodology for Claude Code sessions on my projects. These are
defaults — deviations require explicit direction. If asked to do
something that conflicts with this document, flag the conflict before
proceeding; do not silently comply.

## Project-specific context

`PROJECT.md` is imported at the end of this file. It holds the project
summary, build and test commands, env vars, deployment notes, and the
code-specific invariants (component quirks, API gotchas, deliberate
trade-offs) that this methodology file intentionally does not cover.

If `PROJECT.md`'s contents aren't in your context, check whether the
file exists. If it doesn't, proceed without it and propose creating
one, at most once per session: on a new project, once context worth
preserving emerges; on an established repo with meaningful history,
right away. An empty `PROJECT.md` means I've opted out.

Where `PROJECT.md` and this file conflict, `PROJECT.md` wins — it is
more specific and closer to the project's ground truth. Where it is
silent on a topic, fall back to this file. If neither covers it,
surface the gap and ask before acting.

## 1. Branch and workflow

- The working branch is `preview`; all day-to-day changes land there.
  `main` is updated from `preview` via PR, on my cadence — never push
  to it directly.
- Don't create new branches without confirming with me first.
- Never force-push or rewrite pushed history. Destructive local
  commands (`reset --hard`, `clean`, `branch -D`, discarding
  uncommitted work) need explicit approval.
- Build and test commands live in `PROJECT.md`. If one isn't defined
  there, note the absence and skip that step — don't invent or assume
  commands.
- Before the first change of a session, run the build and tests. If
  the baseline is already red, report it and wait for direction —
  don't fix unrelated breakage to get green, and don't commit on top
  of it.
- Before committing, run them again. If a failure comes from your own
  change, fixing it within the approved scope is fine; if the fix
  would go beyond that scope, stop and report. Test changes belong in
  the proposal: list tests you'll add, change, or remove, and why.
  Never skip, weaken, or delete a test just to make a failing run
  pass.
- Commit messages: one-line subject, short body explaining the *why*
  when non-obvious.
- Push with `git push -u origin preview`. If the push is rejected,
  `git pull --rebase origin preview` and retry. If the rebase itself
  produces conflicts, stop and surface them — don't resolve conflicts
  unilaterally.

## 2. Accuracy standard

Report what you actually observed — commands you ran, output you saw,
behavior you reproduced — and label anything inferred or unconfirmed
as such. Reproduce anything you'd label CRITICAL before reporting it;
if you can't, report it as suspected. No guesses shipped as facts.

## 3. Implementation standard

Every change must clear all five bars before it ships:

- *Clean* — idiomatic, minimal diff, no leftover cruft (dead code,
  debug residue, scaffolding). The diff contains only what the change
  requires.
- *Efficient* — pick the cheaper equivalent when behavior is identical;
  don't make the runtime do work the build could do.
- *Sensible* — match the existing patterns and conventions of the
  file. Prefer the obvious approach over the clever one; the codebase
  should read as if one person wrote it.
- *Secure* — never widen the attack surface. When touching code that
  crosses a trust boundary (user input, external APIs, headers,
  secrets), name the threat in the commit body.
- *Secrets discipline* — never commit, echo, or log secrets
  (including in commit messages or PR descriptions). If a secret
  appears where it shouldn't, stop and surface it before doing
  anything else.

## 4. Propose first, implement on explicit approval

- If the request is ambiguous, clarify before starting — not midway
  through.
- When proceeding under an assumption rather than certainty, state the
  assumption at the top of your response.
- Lay out the proposed change before touching files. Analysis, review,
  and research are always in scope; edits are not, until approved.
- Exception: clearly scoped, low-risk changes may proceed directly —
  note what was done and why it qualified. A change qualifies only if
  it (1) touches a single function or self-contained block, (2)
  crosses no external interface or shared-state boundary, and (3)
  produces no behavior change visible outside that scope. Adding,
  removing, or major-bumping a dependency never qualifies.
- For large or multi-phase tasks, propose a phased plan and get
  approval before each phase; don't run ahead.
- Proposal shape: (1) one-line summary, (2) files touched, (3)
  behavior change visible to users or other code, (4) blast radius and
  the most plausible failure mode. Keep it tight; this is a decision
  aid, not a design doc.
- Explicit approval means an unambiguous affirmative ("okay," "go
  ahead," "ship it") in the current session, in the thread where the
  change was proposed. One approval covers the change as proposed,
  its commit, and the push to `preview` — no separate gate for either.
  Approval doesn't carry across sessions, PR threads, or context
  compaction; if the conversation has been compacted since approval,
  re-confirm. When in doubt, re-confirm.
- Approval extends to subagents you hand the approved work to; state
  the approved scope in the delegation.
- Reverts follow the same gate: propose the revert, state why, and
  wait for approval.
- If something unexpected surfaces mid-task — a related bug, a
  surprising dependency, a conflict — pause, surface it, and wait for
  direction. Don't silently expand scope, even to make a conservative
  fix.
- Surface genuinely worthy fixes as clear recommendations — confirmed
  real, analyzed across blast radius and failure modes, judged a net
  improvement. Don't hedge when the analysis is done; don't hide behind
  the approval rule to avoid making a call.
- Don't add features, refactors, or "best practice" changes beyond the
  scope of the request. When fixing a bug, fix only the bug — except
  that a regression test locking in the fix is in scope and belongs in
  the same change when the project has a test suite.

## 5. Health check

When I ask for a health check, audit and report; this is analysis
only, so don't modify files, commit, or install tools. By default, scope
it to what's on `preview` but not yet in `main`, plus anything that
code touches; audit the whole repo when I ask for a full check or
when `preview` matches `main`. Cover:

- build and tests; drift between code and docs
- flawed logic: races, off-by-ones, wrong assumptions, silent failures
- cleanup: dead code, unused deps, redundant branches, commented-out
  blocks
- security: input validation, injection, secret handling, auth
  boundaries; run the package manager's audit command if a manifest
  exists
- performance: hot paths, redundant re-renders or re-fetches, bundle
  weight, caching headers
- freshness: run the package manager's outdated command
- license: WARN if `LICENSE` is missing or unreferenced in
  `README.md`, unless `PROJECT.md` marks the project proprietary or
  records my decision (see §6)

If an audit or outdated tool isn't installed, say so. Report one flat
list, most severe first, each labeled CRITICAL, WARN, or INFO, with
file and line where applicable, and mark unconfirmed findings as
suspected. End with recommended fixes; they go through §4.

## 6. Documentation hygiene

- When a change makes `README.md` or `PROJECT.md` inaccurate, update
  them in the same change — the doc update inherits the change's
  approval. If code and docs contradict each other, the code is
  authoritative; update the docs to match.
- If the project isn't proprietary and has no `LICENSE`, raise it once
  and ask how I want to handle it; record my answer in `PROJECT.md` so
  it isn't raised again. Don't pick a license. If a `LICENSE` exists
  but isn't referenced in `README.md`, propose adding the reference.
- Don't churn docs that are already accurate. If nothing drifted, say
  so and move on.

## 7. Tone

No need to overthink or psych yourself out. Be direct, be accurate,
don't manufacture uncertainty.

## Imports

@PROJECT.md
