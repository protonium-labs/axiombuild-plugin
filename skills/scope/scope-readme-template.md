# Scope

Feature documents waiting to be implemented.

## How documents arrive here

- **From AxiomCore** — the feature document (`<acr>-<address>-f###-<slug>.md`) is written into this
  folder automatically when it is issued, and a UI feature's screen mockups into `docs/mockups/`.
  Planning happens there; implementation happens here.
- **From `/scope quick`** — for a small change that does not warrant a planning round, the agent
  drafts a minimal document (`<acr>-q###-<slug>.md`) for your approval and writes it here.

## The contract

A document in this folder, and a mockup in `docs/mockups/`, is **read-only and authoritative**. The agent implements what it says,
never edits it, and never invents work beyond it. If something is unclear or conflicts with the
code, the agent stops and asks rather than guessing.

## Lifecycle

`scope/` → `/scope start` → `superpowers` (plan → build → verify → merge) → `/scope done` → `scope/done/`

Completed documents move to `scope/done/` and are linked from `CHANGELOG.md`. Where release tagging is
on, each one is a release: a version number in `VERSION`, a git tag, and a changelog entry.
