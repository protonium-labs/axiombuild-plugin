# Changelog

All notable changes to the AxiomBuild plugin. Newest on top.

## 2.1.0 — 2026-09-12

A UI feature arrives with the screens it touches already drawn. Measured on the same project: five of the seven revisions its design system went through after V001 came up while a view was being built or looked at — layout was settled by seeing the running app, one revision at a time. AxiomCore now draws each screen as an HTML mockup, has the owner approve it, and issues it with the feature document (AxiomCore `su-014`).

- **`docs/mockups/`** holds one self-contained HTML file per screen, written by AxiomCore, read-only here. The feature document names its files in `mockups:` frontmatter.
- **`/scope start` checks the mockups.** Every file in `mockups:` must be in `docs/mockups/`; a missing one is named and no branch is cut. An absent or empty `mockups:` passes, so every existing document keeps working.
- **`context/current-feature.md` gains `mockups`**, and the start report hands `writing-plans` the mockup paths beside the document path.
- **Library documentation via context7**, not memory — stated in the skill's loop description.
- **No compare step and no gate on `/scope done`.** The owner checks the running screen against the mockup. AxiomBuild stays bookkeeping.
- **Fixed:** the `scope/README.md` template still said feature documents are copied in by hand. They have arrived automatically since 1.2.0.

**Upgrading.** Nothing to do. Documents without `mockups:` behave exactly as on 2.0.0.

## 2.0.0 — 2026-09-09

**Breaking.** AxiomBuild stops implementing. It keeps the books, and `superpowers` does the building.

The evidence, measured on a real project ten features in: **6,282 lines** of feature document produced **5,821 lines** of shipped code, and those ten documents carried **28 versions** between them — eighteen rewrites. The cause was structural, not sloppiness. A feature document written by a planner that has never run the code was asserting how the code should be arranged: file trees, task breakdowns, per-task acceptance criteria, interface signatures. Those assertions were wrong often, and each wrong one became a clarification, a version bump, a re-check and a re-issue. The document now states requirements; the task plan is written where the code is readable and runnable.

- **`/implement` and `/verify` are gone.** `superpowers:writing-plans` decomposes the feature with the repository in context; `executing-plans` runs it TDD-first, committing per step; `verification-before-completion` and `requesting-code-review` gate it; `finishing-a-development-branch` merges it. Entry is at `writing-plans`, **never** `brainstorming` — the product thinking already happened in AxiomCore, and its hard gate would repeat it.
- **The three agents are gone** — `spec-checker`, `criteria-verifier`, `code-scanner`. They existed to make thousand-line documents safe. A forty-line requirements document is read by a person in under a minute, and code review now reads the diff instead of the document.
- **`/scope check` and `/scope load` are gone.** `check` duplicated a gate AxiomCore already ran; `load` parsed a task ledger that no longer exists.
- **`/scope start <ID>`** replaces `load`: records the feature in `context/current-feature.md` and cuts `feature/<ID>`. It reads the frontmatter and nothing else, and it plans nothing.
- **`/scope done <ID>`** replaces `archive` and `/implement complete`: confirms the work is merged, moves the document to `scope/done/`, writes the changelog line, resets the current-feature record. It does not merge, test or verify — it records what already happened.
- **`/scope quick` drafts requirements, not tasks.** The three gates are unchanged and still assessed against the code rather than the description.
- **`/scope standards` is unchanged** and now matters more: with no bundled verifier, the stack profile is where the test and build commands live.
- **`context/current-feature.md` loses the task ledger and the binding table.** It records which feature is open, not how it is being built. A `Blocked` section remains, because a question is worth recording — and is still answered in AxiomCore, never by editing the read-only document.
- **Fixed:** both bundled markdownlint configs missed the `MD025` `front_matter_title` exemption, so any document carrying frontmatter plus an H1 — which every AxiomCore feature document has since its own `su-010` — would fail the lint it was handed to. Latent until now only because documents in `scope/` are never linted.
- **`superpowers` is a real dependency.** `/scope init` checks for it and says so if it is missing, but never installs it unasked.

**Upgrading.** Finish any in-flight feature on 1.x first. Your `scope/`, `CHANGELOG.md` and `context/` are untouched by the update; only `context/current-feature.md` changes shape, and only once `/scope start` next writes it.

## 1.2.0 — 2026-09-07

Two changes: a feature run goes end to end without the user in it, and the arrival of a document from AxiomCore stops being something a human carries.

- **`/implement start [all|N]` runs the tasks.** No argument runs all of them. `start 0` branches without implementing, which is the old behaviour when you want to read the tasks first. `next` alone still runs exactly one task; `next all` or `next N` resumes a stopped run. The budget lives in the ledger, so a resumed session knows what it was.
- **Each task is gated before it counts.** `/verify T<n>` — `criteria-verifier` judging the diff in a separate context — then the project's test command. **Validation between tasks moved rather than disappeared**: the old rhythm stopped after every task so the user could look, which is a weaker check than a separate agent reading the criteria, and it cost a round trip each time.
- **A commit lands after every successful task, before the next one starts.** One commit, one task, in order — that is how a finished run is read. Never batched, never deferred to the end.
- **A run stops on a failing criterion, a failing test, or any of the four stop triggers**, and never starts the next task afterwards. The work stays on the branch, uncommitted: a failed task is usually mostly right, and discarding it costs more than leaving it. A run that finishes every task ends with `/verify all` and its matrix; `complete` stays manual, because it merges to `main`.
- **One language rule for every stop and every question this skill asks.** The reader knows HTML, CSS, JavaScript and common frameworks and has never worked in IT: concrete technical words yes, jargon and in-house shorthand no, and no talking down either — vague is not the same as plain. Tracebacks and failing assertions are held and given only on request.
- **`F###` documents now arrive on their own.** AxiomCore writes them into `scope/` once they pass its own pre-issue check, and rewrites the file when it re-issues a corrected version. `spec-checker`'s remedy text and `/scope`'s pointers no longer describe a human courier; a gap found here is one that check missed or that the transfer broke, and the report says which.
- **Fixed:** the ledger template wrote `spec_version`, a pin AxiomCore retired — a loaded feature would have carried a field that no longer exists on any document. It now carries the document's own `version`, and the ledger gains `budget`.

## 1.1.0 — 2026-08-11

Markdown quality, brought over from AxiomCore — adapted, because AxiomBuild is a guest in someone else's repository.

- **Markdown gate in `/verify all`.** New `Docs` row in the matrix, next to `Tests`, `Build` and `Lint`. Scoped to the four files this agent writes — `CHANGELOG.md`, `scope/README.md`, `scope/registry.md`, `context/**/*.md`. Reports, never fixes.
- **Config resolution, three steps.** The repository's own `.markdownlint*` wins and is never edited or replaced; otherwise the config bundled at `skills/verify/markdownlint.json`; otherwise a temp file outside the repository. **Nothing is ever written into the user's project** — the difference from AxiomCore, which owns its workspace and can scaffold a config into it.
- **Feature documents are exempt from the gate**, loaded or archived. They are read-only contracts, so a lint finding against one would be a finding nobody is permitted to fix.
- **`spec-checker` check 0 — structurally parseable.** Balanced code fences, well-formed tables, one H1, tasks at `###`, delimited frontmatter. Runs before the semantic checks and stops them. This is the check with teeth: `/scope load` reads the ledger out of this structure, and an unclosed fence swallows every heading after it, so tasks do not error — they disappear.
- **Write-time rule** in `/scope` and `/implement`: every markdown file written or edited is lint-clean before it is reported done, `--fix` included. A quick draft is linted before it is shown for approval, while it is still editable.
- **Fixed:** `scope/README.md` was generated with bold labels where headings belong, so a freshly initialised project failed the new gate on day one. The template now uses real headings.
- No `npx`, no network: every check reports `skipped` with the reason. Never silently omitted — a matrix missing a row reads exactly like one that passed it.

## 1.0.0 — 2026-08-04

First release.

### Skills

- `/scope` — `init`, `quick`, `list`, `check`, `load`, `archive`, `standards`. Owns the `scope/`
  inbox and the three gates that send an oversized quick change back to AxiomCore.
- `/implement` — `start`, `next`, `status`, `explain`, `blocked`, `commit`, `complete`. One task
  per `next`; one squashed commit per feature on `main`.
- `/verify` — `all`, `T<n>`, `criteria`. Per-criterion pass / fail / unverifiable matrix, plus
  tests, build, and a scope-drift check. Gates `/implement complete`.

### Agents

- `spec-checker` — pre-flight validation of a feature document. Reports, never repairs.
- `criteria-verifier` — judges the diff against the criteria in a context that did not write the
  code. Never infers a pass.
- `code-scanner` — audits recent changes for security, performance, quality, componentization.
  Reads the project's stack profile rather than carrying stack knowledge itself.

### Protocols

- Feature IDs: `<ACR>-F###` from AxiomCore, `<ACR>-Q###` minted locally. A document with no
  `origin` field is treated as AxiomCore's, so existing feature documents need no change.
- Commits: `<type>(<ID>): <subject>`, subject transcribed from the document title. No attribution.
- Changelog: latest on top, one line per feature, referencing the archived document. No prose,
  no SHA — the ID in the commit message is the link.
- Four stop triggers halt implementation and produce a question instead of a decision.
- Corrections compound: when the user corrects *how* something is done, `/implement` offers once
  to record it in the stack profile, so the same guidance is given once rather than repeatedly.
  `code-scanner` reports when that profile is still a stub, since its findings depend on it.
