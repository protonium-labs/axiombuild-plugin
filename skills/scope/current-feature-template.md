---
format: v1
status: idle
---

# Current Feature

No feature loaded. Run `/scope list` to see what is waiting, then `/scope load <ID>`.

<!--
ACTIVE SHAPE — what /scope load writes. Reset to the idle state above by /implement complete.

---
format: v1
feature: SP-F012
title: Favorites page
origin: axiomcore          # axiomcore | local  (absent in a document => axiomcore)
type: new                  # new | change | quick
spec_version: V003         # omitted for quick features
source: scope/sp-01-f012-favorites.md
branch: feature/SP-F012
status: in-progress        # idle | loaded | in-progress | blocked | verified
started: 2026-08-04
---

# Current Feature — SP-F012 Favorites page

## Tasks

| # | Task | Status | Criteria |
| :- | :--- | :--- | :--- |
| T1 | Add owner-scoped getFavorites query | done | 3/3 |
| T2 | Build the /favorites route | in-progress | 0/2 |
| T3 | Star button in the top bar | pending | 0/1 |

Task status: pending | in-progress | done | blocked

## Binding

Which design sections of the source document are binding, resolved at load time.

| Section | Standing |
| :--- | :--- |
| Data & Error Flow | binding (always) |
| Interfaces — public | binding (always) |
| Interfaces — internal | binding |
| Affected Structure | proposed |
| Logic | not present |

## Out of Scope

Copied verbatim from the source document. These are prohibitions, not hints.

- Collection favorites — covered by SP-F013

## Blocked

Questions raised during implementation. Empty when clear. Each entry names the trigger that
fired, so it can be taken back to AxiomCore as /dev revise input.

- **T2 · binding conflict** — the Interfaces section specifies `getFavorites(userId)`, but the
  existing query layer takes a session object. Which wins?
-->
