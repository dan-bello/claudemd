# CLAUDE.md

Standing methodology for Claude Code sessions on my projects. These are defaults; deviations need explicit direction. If a request conflicts with this file, flag the conflict before proceeding — never silently comply.

## Project-specific context

`PROJECT.md` (imported at the end) holds project specifics: summary, build and test commands, env vars, deployment notes, and code-level invariants (component quirks, API gotchas, deliberate trade-offs). It wins any conflict with this file — it's closer to ground truth.

If its contents aren't in context, check whether it exists. If not, proceed without it and offer to create one, once per session — immediately on an established repo, or once context worth keeping emerges on a new one. An empty `PROJECT.md` means I've opted out.

## 1. Branch and workflow

- The working branch is `preview`, unless I've set up another for the task or you've created one with my confirmation. All changes land there. `main` is updated from `preview` via PR on my cadence — never push to it, and never merge the working branch anywhere yourself.
- Don't create branches without confirming with me first.
- Never force-push or rewrite pushed history. Destructive local commands (`reset --hard`, `clean`, `branch -D`, discarding uncommitted work) need explicit approval.
- Build and test commands come from `PROJECT.md`. If one isn't defined, note it and skip that step — don't invent commands.
- Baseline: run build and tests before the session's first change. Start them while preparing the first proposal and include the result in it (or in your report, if the change needs no proposal). If red, stop for direction — don't fix unrelated breakage or commit on top of it.
- While iterating, use the narrowest relevant test command `PROJECT.md` offers. Before committing, you need a full green build-and-test run that postdates your last edit; don't rerun if you already have one.
- Fix failures your change caused within approved scope; if the fix needs more, stop and report. Never skip, weaken, or delete a test to get green.
- Docs-only changes skip build and test runs.
- Commits: one-line subject; short body for the *why* when non-obvious.
- Push with `git push -u origin <working-branch>`. If rejected, `git pull --rebase origin <working-branch>`. If the rebase conflicts, stop and surface it — don't resolve conflicts yourself. Otherwise, if the incoming commits touch more than docs, rerun build and tests (stop and report on failure), then push.

## 2. Accuracy standard

Report what you observed — commands run, output seen, behavior reproduced — and label anything inferred or unconfirmed. Reproduce anything you'd call CRITICAL before reporting it; if you can't, report it as suspected. No guesses shipped as facts.

## 3. Implementation standard

Every change clears all five bars:

- *Clean* — idiomatic, minimal diff, nothing the change doesn't require (no dead code, debug residue, or scaffolding).
- *Efficient* — when behavior is identical, pick the cheaper option; don't make the runtime do work the build could do.
- *Sensible* — match the file's existing patterns; prefer obvious over clever. The codebase should read as one author's.
- *Secure* — never widen the attack surface. When touching a trust boundary (user input, external APIs, headers, secrets), name the threat in the commit body.
- *Secrets discipline* — never commit, echo, or log secrets, including in commit messages or PR descriptions. If a secret appears where it shouldn't, stop and surface it before anything else.

## 4. Propose first, implement on explicit approval

Analysis, review, and research are always in scope; edits need approval unless they qualify below. Minimize round trips: any message that needs my input should contain everything I need to decide.

- *Ambiguity:* resolve it before starting, never midway. If one reading is clearly likeliest, include its proposal with the question so one reply settles both. When proceeding on an assumption, state it at the top.
- *Proposal shape:* (1) one-line summary, (2) files touched, including tests added, changed, or removed and why, (3) behavior change visible to users or other code, (4) blast radius and most plausible failure mode. Scale it to the change — a small fix can be four short lines.
- *Batching:* number independent proposals so I can approve any subset in one reply. Multi-phase work gets a phased plan; approval covers only the phases I name, and you don't run ahead.
- *Explicit approval* is an unambiguous affirmative ("okay," "go ahead," "ship it") after the proposal, in the same session and thread. It covers the change as proposed, its commit, and the push to the working branch, and extends to subagents (state the approved scope in the delegation). It doesn't survive a new session, a different PR thread, or context compaction; when in doubt, re-confirm.
- *Skip the proposal* when I've specified the exact change — what and where, with no design choices left to you — or when it's low-risk: touches a single function or self-contained block, crosses no external interface or shared-state boundary, and changes no behavior visible outside that scope. These count as approved, commit and push included; note what you did and why it qualified. If an exact-spec request carries a risk I likely missed, flag it and wait for my reply before making the change. When unsure whether a request is exact, propose. Dependency changes (add, remove, major bump) never qualify as low-risk; they skip the proposal only when I've specified them exactly.
- *Reverts you'd initiate* follow the full gate: propose, say why, wait.
- *Mid-task surprises* (a related bug, an unexpected dependency, a conflict): if it affects the approved change's correctness, safety, or blast radius — or you can't rule that out — pause and surface it. Otherwise finish the approved scope and report it at the end. Never silently expand scope, even for a conservative fix.
- *Recommendations:* when a fix is confirmed real, analyzed for blast radius and failure modes, and a net improvement, recommend it plainly. Don't hedge once the analysis is done, and don't hide behind the approval rule to avoid making a call.
- *Scope:* no features, refactors, or "best practice" changes beyond the request. A bug fix fixes only the bug, plus a regression test in the same change when the project has a test suite.

## 5. Health check

When I ask for a health check, audit and report only — don't modify files, commit, or install tools. Default scope: what's on the working branch but not yet in the branch it merges into (`main`, or for a task branch, usually the branch it was cut from), plus code it touches. Audit the whole repo when I ask for a full check or there's nothing to diff. Run independent checks in parallel, and reuse this session's build and test results if nothing has changed since.

Cover:

- build and tests; drift between code and docs
- flawed logic: races, off-by-ones, wrong assumptions, silent failures
- cleanup: dead code, unused deps, redundant branches, commented-out blocks
- security: input validation, injection, secret handling, auth boundaries; the package manager's audit command if a manifest exists
- performance: hot paths, redundant re-renders or re-fetches, bundle weight, caching headers
- freshness: the package manager's outdated command
- license: WARN if `LICENSE` is missing or unreferenced in `README.md`, unless `PROJECT.md` marks the project proprietary or records my decision (§6)

If an audit or outdated tool isn't installed, say so. Report one flat list, most severe first, each labeled CRITICAL, WARN, or INFO, with file and line where applicable; mark unconfirmed findings as suspected. End with numbered recommended fixes I can approve by number under §4.

## 6. Documentation hygiene

- When a change makes `README.md` or `PROJECT.md` inaccurate, update them in the same change; the doc update inherits its approval. If code and docs contradict each other, code is authoritative — update the docs to match.
- If the project isn't proprietary and has no `LICENSE`, raise it once, ask how I want to handle it, and record my answer in `PROJECT.md` so it isn't raised again. Don't pick a license. If a `LICENSE` exists but `README.md` doesn't reference it, propose adding the reference.
- Don't churn accurate docs. If nothing drifted, say so and move on.

## 7. Tone

No need to overthink or psych yourself out. Be direct, be accurate, be brief, and don't manufacture uncertainty.

## Imports

@PROJECT.md
