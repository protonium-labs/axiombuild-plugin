---
format: v1
status: idle
---

# Current Feature

No feature open. Run `/scope list` to see what is waiting, then `/scope start <ID>`.

<!--
ACTIVE SHAPE — what /scope start writes. Reset to the idle state above by /scope done.

This file records WHICH feature is open. It does not hold the task plan: that is
superpowers' business and lives in docs/superpowers/plans/, beside the code.

---
format: v1
feature: SP-F012
title: Favorites page
origin: axiomcore          # axiomcore | local  (absent in a document => axiomcore)
type: new                  # new | change | quick
version: V003              # the feature document's own version; no other document's version is pinned
source: scope/sp-01-f012-favorites.md
mockups: [docs/mockups/sp-01-006-mockup-favorites.html]   # from the document's frontmatter; [] when it has no UI
branch: feature/SP-F012
status: open               # idle | open | blocked
started: 2026-09-09
---

# Current Feature — SP-F012 Favorites page

Requirements: `scope/sp-01-f012-favorites.md` (read-only).
Mockups: `docs/mockups/sp-01-006-mockup-favorites.html` (read-only).
Task plan: `docs/superpowers/plans/2026-09-09-favorites-page.md`.

## Blocked

Questions raised while building. Empty when clear.

A question here is taken to AxiomCore as `/dev revise` input — it is never resolved by editing
the source document. The answer comes back as a bumped version of that document, re-issued into
scope/. An implementing agent must not be able to move the target it is measured against.

- **2026-09-09** — The requirements say favorites are per-user, but the existing session carries
  no user id on the public route. Which is intended?
-->
