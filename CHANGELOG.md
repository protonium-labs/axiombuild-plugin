# Changelog

All notable changes to the AxiomBuild plugin. Newest on top.

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
