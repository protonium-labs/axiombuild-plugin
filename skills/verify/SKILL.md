---
name: verify
description: Check the loaded feature against its acceptance criteria before it can be completed. Actions - all (criteria matrix + tests + build + markdown gate + scope-drift check; this is the gate for /implement complete), T<n> (one task), criteria (matrix only, skipping tests, build and markdown), overview. Criteria are judged by a separate agent, not by the context that wrote the code.
---

# Task

Execute the requested action: $ARGUMENTS

Calling rule: `/verify <action>`.

If the requested action is ambiguous or contradictory, do NOT guess: flag it and ask for clarification before executing.

---

## If action is empty

Describe this skill: its purpose and the actions table from "overview". Do not execute anything.

## If action is "overview" (or unknown)

| Argument | Action |
| :--- | :--- |
| `all` | Criteria matrix + tests + build + markdown gate + scope-drift check. The gate for `/implement complete` |
| `T<n>` | One task's criteria only |
| `criteria` | The full matrix, skipping tests, build and the markdown gate. Fast pass while still working |
| `overview` | This table |

If the action was unknown, say so first, then show the table.

---

## If action is "all"

1. **Read the ledger and the source document.** Criteria come from the document, not from the ledger summary.
2. **Delegate the criteria matrix to the criteria-verifier subagent.** It receives the diff and the criteria, and nothing about how the code came to be written. The context that wrote the code does not judge whether it meets the spec.
3. **Run the project commands** named in `context/coding-standards.md` — tests, then build, then lint if one is defined. Report failures with the actual output, not a summary of it.
4. **Markdown gate.** Lint the markdown *this agent writes*, and only that:

   ```bash
   npx --yes markdownlint-cli2 [--config <path>] "CHANGELOG.md" "scope/README.md" "scope/registry.md" "context/**/*.md"
   ```

   **Not the rest of the repository.** The project's own documentation belongs to the project, not to this agent, and reformatting it is scope drift by any other name.

   **Not the feature documents either** — neither the loaded one nor anything in `scope/done/`. Those are read-only contracts (see Notes), so a finding against one would be a finding nobody is allowed to fix. Their structural validity is checked earlier and elsewhere, by `spec-checker` at `/scope check`.

   Resolve the configuration in this order, stopping at the first that applies:

   1. **The repository already has one** — any `.markdownlint.{json,jsonc,yaml,yml,cjs,mjs}` or `.markdownlint-cli2.*`. Pass no `--config` at all; `markdownlint-cli2` discovers it and walks up from each linted file. **Never edit it, never replace it, never add a second one.** The project's house style wins over ours, always.
   2. **`$CLAUDE_PLUGIN_ROOT` resolves** — `--config "$CLAUDE_PLUGIN_ROOT/skills/verify/markdownlint.json"`.
   3. **Neither** — write these two settings to a temp file *outside* the repository and point `--config` at it. Nothing is added to the user's project, ever:

      ```json
      { "default": true, "MD013": false }
      ```

   `MD013` (line length) is off deliberately: a changelog line carrying a title and a link has no natural wrap point.

   **No `--fix` here.** This skill reports; it does not repair. A finding goes back to whichever skill wrote the file, which fixes it with `--fix` and re-runs — that is one command, not a negotiation, so a `Docs` finding is never waived and never blocks for long.

   If `npx` is unavailable or the network is closed, report the row as `skipped` with the reason. A missing linter is not a failure — but it is never silently dropped from the matrix either, because a matrix that quietly omits a check reads exactly like one that passed it.
5. **Scope-drift check.** Compare the changed files against the tasks:
   - A file changed that no task accounts for → flag it.
   - Anything touching the ledger's `Out of Scope` list → flag it. That is a prohibition, and crossing it is a finding, not a bonus.
   - A binding section deviated from → flag it. Deviating from a *proposed* section is allowed and only needs to be recorded.
6. **Report the matrix** (see below) and record the result in the ledger frontmatter:
   `verified: <date> · <pass|fail> · <branch HEAD sha>`.

   The sha matters: `/implement complete` compares it against the current HEAD, so a verification goes stale the moment more code lands. Re-run rather than trusting an old pass.

## If action is `T<n>`

The criteria matrix for one task. No tests, no build, no markdown gate, no scope-drift check. Does not record a result — it cannot gate anything on its own.

## If action is "criteria"

The full matrix across every task, skipping tests, build and the markdown gate. Useful mid-feature. Does not record a gating result.

---

## The matrix

Report per task, per criterion, with one of three verdicts:

| Verdict | Meaning |
| :--- | :--- |
| **pass** | Observably met. Name what was checked |
| **fail** | Observably not met. Name what is missing |
| **unverifiable** | The criterion cannot be mechanically checked ("feels responsive", "looks right") |

```text
T1 — Add owner-scoped getFavorites query          3/3 pass
  ✓ Returns only rows owned by the session user   src/lib/db/favorites.test.ts:22
  ✓ Orders by pinned, then updatedAt              src/lib/db/favorites.test.ts:41
  ✓ Returns an empty array, never null            src/lib/db/favorites.test.ts:58

T2 — Build the /favorites route                   1/2 pass · 1 fail
  ✓ Renders items and collections in one list     src/app/(app)/favorites/page.tsx
  ✗ Empty state when nothing is starred           no empty-state branch in the component

Tests   pass (48)
Build   pass
Lint    pass
Docs    1 finding — CHANGELOG.md:14 MD032 list not surrounded by blank lines
Drift   1 finding — src/lib/format.ts changed, no task covers it
```

**A `fail` is never waivable.** It means the code does not do what the document says.

**An `unverifiable` needs your judgement, and only yours.** The agent cannot measure whether an animation feels smooth. It reports the criterion, states plainly that it cannot check it, and asks. If you accept it, that is recorded as an accepted-unverified criterion — never silently upgraded to a pass. Repeated unverifiables are a signal that criteria are being written too softly; `/scope check` is where that gets fixed.

---

## After any action

Report as two short lists: what the matrix found, and what needs the user's decision. Never pad, and never soften a failure into a caveat.

## Notes

- This skill is **read-only on source code**. It never fixes what it finds — a failure goes back to `/implement next`, a `Docs` finding to the skill that wrote the file, and a scope-drift finding to the user.
- It never edits the source document, and never rewrites a criterion to something it can check.
- The markdown gate never touches a file outside the four globs in step 4, and never writes a config file into the user's repository.
- A verification is bound to a commit. If HEAD moved, it is stale and means nothing.
- If the criteria themselves are the problem rather than the code, say so directly. Softly-worded criteria are a `/scope check` failure that got through, and the remedy is a document revision, not a generous reading.
