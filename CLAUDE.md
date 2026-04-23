# CLAUDE.md

Standing methodology for Claude Code sessions on my projects. These are
defaults — deviations require explicit direction. If asked to do
something that conflicts with this document, flag the conflict before
proceeding; do not silently comply.

## Project-specific context

Before acting in this repo, also read `PROJECT.md`. It holds the
project summary, env vars, deployment notes, and the code-specific
invariants (component quirks, API gotchas, deliberate trade-offs)
that this methodology file intentionally does not cover. If
`PROJECT.md` does not yet exist, that's expected on new projects —
proceed without it, and propose creating one when project-specific
context surfaces that's worth preserving.

Where `PROJECT.md` and this file conflict, `PROJECT.md` wins — it is
more specific and closer to the project's ground truth. Where
`PROJECT.md` exists but is silent on a specific topic, fall back to
this file. If neither covers it, surface the gap and ask before
acting.

## 1. Branch and workflow

- The working/development branch is `preview`. All day-to-day changes
  land there.
- Never push directly to `main`. `main` is updated from `preview` via
  PR, on my cadence.
- Do not create new branches without confirming with me first.
- On session start, if the build is already broken or tests already
  fail on a clean checkout, report it before doing any work and wait
  for direction. Do not begin fixing unrelated breakage to satisfy
  the pre-commit verification rule below, and do not treat a red
  baseline as license to commit on top of it.
- Before committing, verify the build succeeds and tests pass —
  project-specific commands live in `PROJECT.md`. If no test command
  is defined in `PROJECT.md`, note the absence and skip this step;
  do not invent or assume a test command. If tests fail, stop and
  report; do not attempt fixes without explicit approval.
- Commit messages: one-line subject, short body explaining the *why*
  when non-obvious.
- Push with `git push -u origin preview`. If the push is rejected,
  `git pull --rebase origin preview` and retry. If the rebase itself
  produces conflicts, stop and surface them — do not resolve conflicts
  unilaterally.
- Do not commit without approval as defined in §4.

## 2. Accuracy standard

Before relaying any finding, re-verify it: reproduce flagged issues,
confirm commands actually ran and produced the output quoted, and
sanity-check that each recommendation survives a second pass. If
something cannot be confirmed, say so explicitly. No guesses shipped
as facts.

## 3. Implementation standard

Every change must clear all four bars before it ships:

- *Clean* — idiomatic, minimal diff, no dead code, no debug residue,
  no leftover scaffolding. The diff contains only what the change
  requires.
- *Efficient* — pick the cheaper equivalent when behavior is identical;
  don't make the runtime do work the build could do.
- *Sensible* — decisions match the existing patterns and conventions of
  the file. Prefer the obvious approach over the clever one; the
  codebase should read as if one person wrote it.
- *Secure* — never widen the attack surface. When touching code that
  crosses a trust boundary (user input, external APIs, headers,
  secrets), name the threat in the commit body.
- *Secrets discipline* — never commit secrets, never echo them in tool
  output, never include them in commit messages, PR descriptions, or
  logs. If a secret appears where it shouldn't, stop and surface it
  before doing anything else.

## 4. Propose first, implement on explicit approval

- If the initial request is ambiguous, clarify before starting — not
  midway through.
- When proceeding under an assumption rather than certainty, state the
  assumption explicitly at the top of your response.
- Lay out the proposed change before touching files. **Exception:
  clearly-scoped, low-risk changes may proceed directly — note what
  was done and why it qualified. To qualify, a change must meet all
  three: (1) touches a single function or self-contained block, (2)
  crosses no external interface or shared state boundary, and (3)
  produces no behavior change visible outside that scope.**
  For large or multi-phase tasks, propose a phased
  plan and get approval before beginning each phase; do not run ahead.
  Analysis, review, and research are always in-scope; edits are not,
  until approved.
- Proposals should follow this shape: (1) one-line summary of the
  change, (2) files touched, (3) behavior change visible to the user
  or to other code, (4) blast radius and the most plausible failure
  mode. Keep it tight; this is a decision aid, not a design doc.
- Explicit approval means "okay," "go ahead," "ship it," or any
  unambiguous affirmative — delivered in the current session, in the
  same thread where the change was proposed. Approval from a prior
  session, a separate PR comment, or any other context does not carry
  forward. Context compaction expires approval: if the conversation
  has been compacted since approval was given, treat it as no longer
  in force and re-confirm. When in doubt, re-confirm. This definition
  applies to both implementation and commits.
- Reverts follow the same gate: propose the revert, state why, and
  wait for approval before executing.
- If something unexpected surfaces mid-task — a related bug, a
  surprising dependency, a conflict — pause, surface it, and wait for
  direction before proceeding. Do not silently expand scope, even to
  make a conservative fix.
- Surface genuinely-worthy fixes as clear recommendations — verified
  real, analyzed across blast radius and failure modes, judged a net
  improvement. Don't hedge when the analysis is done; don't hide behind
  the approval rule to avoid making a call.
- Do not add features, refactors, or "best practice" changes beyond
  the scope of the request. When fixing a bug, fix only the bug —
  with one exception: a regression test that locks in the fix is
  in-scope and should be added in the same change when the project
  has a test suite.

## 5. Health-check routine

When I ask for a health check (or equivalent), run through:

- **Overall health report** — build status, obvious breakage, drift
  from docs.
- **Flawed-logic scan** — race conditions, off-by-ones, wrong
  assumptions, silent failure modes.
- **Cleanup scan** — dead code, unused deps, redundant branches,
  commented-out blocks.
- **Security test** — input validation, injection vectors, secret
  handling, auth boundaries; run the package manager's audit command
  (e.g. `npm audit`, `cargo audit`, `pip-audit`) if a manifest is
  present.
- **Performance test** — hot paths, unnecessary re-renders/re-fetches,
  bundle weight, caching headers.
- **Test suite** — if the project has tests, run them and report
  results; the command lives in `PROJECT.md`.
- **Dependency freshness** — run the package manager's outdated command
  (e.g. `npm outdated`, `cargo outdated`, `pip list --outdated`); flag
  anything behind so updates don't pile into a single painful bump
  later. Keep this in steady cadence, not only when asked.
- **Verification** — re-verify all findings before reporting (see §2).

Report findings as a flat list, each prefixed with a severity label:
**CRITICAL**, **WARN**, or **INFO**. Do not implement fixes without
explicit approval (see §4).

## 6. Documentation hygiene

- When a change makes README.md or PROJECT.md inaccurate, update them
  as part of the same change — the doc update inherits the approval
  already granted for the code change and does not require a separate
  gate. If code and docs contradict each other, the code is
  authoritative — update the docs to match, not the reverse.
- Don't churn docs that are already accurate. If nothing drifted, say
  so and move on.

## 7. Tone

No need to overthink or psych yourself out. Be direct, be accurate,
don't manufacture uncertainty.
