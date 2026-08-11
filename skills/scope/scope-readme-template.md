# Scope

Feature documents waiting to be implemented.

## How documents arrive here

- **From AxiomCore** — you copy the feature document (`<acr>-<address>-f###-<slug>.md`) into this
  folder by hand. Planning happens there; implementation happens here.
- **From `/scope quick`** — for a small change that does not warrant a planning round, the agent
  drafts a minimal document (`<acr>-q###-<slug>.md`) for your approval and writes it here.

## The contract

A document in this folder is **read-only and authoritative**. The agent implements what it says,
never edits it, and never invents work beyond it. If something is unclear or conflicts with the
code, the agent stops and asks rather than guessing.

## Lifecycle

`scope/` → `/scope check` → `/scope load` → `/implement` → `scope/done/`

Completed documents move to `scope/done/` and are linked from `CHANGELOG.md`.
