# Changelog

All notable changes to the AxiomBuild plugin. Newest on top.

## 1.0.0 — 2026-08-04

First release.

**Skills**

- `/scope` — `init`, `quick`, `list`, `check`, `load`, `archive`, `standards`. Owns the `scope/`
  inbox and the three gates that send an oversized quick change back to AxiomCore.
- `/implement` — `start`, `next`, `status`, `explain`, `blocked`, `commit`, `complete`. One task
  per `next`; one squashed commit per feature on `main`.
- `/verify` — `all`, `T<n>`, `criteria`. Per-criterion pass / fail / unverifiable matrix, plus
  tests, build, and a scope-drift check. Gates `/implement complete`.

**Agents**

- `spec-checker` — pre-flight validation of a feature document. Reports, never repairs.
- `criteria-verifier` — judges the diff against the criteria in a context that did not write the
  code. Never infers a pass.
- `code-scanner` — audits recent changes for security, performance, quality, componentization.
  Reads the project's stack profile rather than carrying stack knowledge itself.

**Protocols**

- Feature IDs: `<ACR>-F###` from AxiomCore, `<ACR>-Q###` minted locally. A document with no
  `origin` field is treated as AxiomCore's, so existing feature documents need no change.
- Commits: `<type>(<ID>): <subject>`, subject transcribed from the document title. No attribution.
- Changelog: latest on top, one line per feature, referencing the archived document. No prose,
  no SHA — the ID in the commit message is the link.
- Four stop triggers halt implementation and produce a question instead of a decision.
- Corrections compound: when the user corrects *how* something is done, `/implement` offers once
  to record it in the stack profile, so the same guidance is given once rather than repeatedly.
  `code-scanner` reports when that profile is still a stub, since its findings depend on it.
