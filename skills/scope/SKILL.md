---
name: scope
description: Own the scope/ inbox - feature documents waiting to be implemented. Actions - init (scaffold a project), quick (draft a small feature locally), list, check (pre-flight validation), load (parse into the task ledger), archive, overview. Documents from AxiomCore are read-only and this skill never mints an F### number.
---

# Task

Execute the requested action: $ARGUMENTS

Calling rule: `/scope <action> [argument]`.

If the requested action is ambiguous or contradictory, do NOT guess: flag it and ask for clarification before executing.

---

## If action is empty

Describe this skill: its purpose and the actions table from "overview". Do not execute anything.

## If action is "overview" (or unknown)

| Argument | Action |
| :--- | :--- |
| `init` | Scaffold this project: `scope/`, `CHANGELOG.md`, the task ledger, the stack profile |
| `quick <description>` | Draft a small feature document locally, subject to the three gates |
| `list` | What is waiting in `scope/`, and what is currently loaded |
| `check <ID>` | Pre-flight: is this document implementable without guessing? Reports, never repairs |
| `load <ID>` | Parse a document into the task ledger. Runs `check` first |
| `archive <ID>` | Move a document to `scope/done/`. Normally called by `/implement complete` |
| `standards [show \| add <rule>]` | Read or extend the project's stack profile |
| `overview` | This table |

If the action was unknown, say so first, then show the table.

---

## If action is "init"

Scaffolds the project. **Non-destructive and idempotent**: anything that already exists is read, never overwritten. Running it twice is safe; running it in a project that predates this plugin adopts what is already there.

1. **Survey first.** List which target files already exist. Report them as "found" and leave their contents alone.
2. **Detect the stack.** Read `package.json` (or the equivalent manifest) for the framework, the test command, and the build command. Do not ask about anything already discoverable.
3. **Ask for the project acronym** — two or three uppercase letters, fixed for the project's life. Warn the user to check it against their AxiomCore project acronyms: the `F###` and `Q###` namespaces share this prefix, so a clash makes IDs ambiguous across the two systems.
4. **Interview for the stack profile** — language and version, test command, build command, lint command, framework conventions, anything the agent must never do (e.g. "never `prisma db push`"). Offer to skip; a skipped interview leaves a stub with the detected values only.
5. **Verify git.** If the folder is not a git repository, say so plainly — `/implement` branches and squash-merges, and will not work without it. Do not run `git init` unasked.
6. **Create what is missing**, from the templates in this skill folder:

   | Path | Template |
   | :--- | :--- |
   | `scope/README.md` | `scope-readme-template.md` |
   | `scope/registry.md` | `registry-template.md` (acronym and `Next quick feature: Q001`) |
   | `scope/done/` | empty folder |
   | `CHANGELOG.md` | `changelog-template.md`, example rows stripped |
   | `context/current-feature.md` | `current-feature-template.md`, idle state |
   | `context/coding-standards.md` | interview answers, or a stub |

7. **Report** two short lists: created, and found-and-left-alone.

## If action is "quick"

Drafts a small feature document locally, so a minor change does not need a planning round in AxiomCore. This is the **only** place the agent authors specification prose; everywhere else it transcribes.

**Run the gates before drafting anything.**

| Gate | Trips when the change requires... |
| :--- | :--- |
| New dependency | A package that is not already in the project manifest |
| Schema change | A database migration, or an edit to the data model |
| Public interface | A new or changed endpoint, exported function signature, or server-action contract |

Assess the gates **against the actual repository, not against the description**. Read the relevant code first. "Change the card border colour" is a style change until you open the file and find the colour comes from a theme token that does not exist yet.

1. If a gate trips: **refuse to create the document.** Name which gate and what specifically triggered it. Then hand the user a drafted description — goal, the constraint that tripped the gate, and what you would propose — ready to paste into AxiomCore's `/dev feature`. Stop there. Do not offer to proceed anyway.
2. If no gate trips: draft the document from `quick-feature-template.md`. Decompose into numbered tasks with observable acceptance criteria. Fill `Out of Scope` with what a reasonable reader might assume is included but is not.
3. **Show the draft and wait for approval.** Nothing is written to `scope/` before the user agrees. Lint the draft before showing it (see *Markdown discipline*) — once it lands in `scope/` it is read-only, and a malformed one can only be fixed by redrafting the whole thing.
4. On approval: read `scope/registry.md`, allocate the next `Q###`, write `scope/<acr>-q###-<slug>.md`, increment the counter. Numbers are monotonic and never reused.
5. Offer to `load` it.

## If action is "list"

1. Read every document directly in `scope/` (not `done/`). For each, show: ID, title, origin, task count, and whether `check` has passed this session.
2. Read `context/current-feature.md` and show what is loaded, if anything.
3. If `scope/` is empty and nothing is loaded, say so and point at `/scope quick`. AxiomCore writes an `F###` document into `scope/` itself once it passes its pre-issue check, so an empty inbox means nothing has been issued yet — not that a copy is waiting to be carried.

## If action is "check"

Pre-flight validation: **is this document implementable without guessing?** Delegate to the **spec-checker** subagent, which owns the check list. Relay its report unchanged.

The checks:

0. **Structurally parseable** — balanced code fences, well-formed tables, one H1, tasks at `###`, delimited frontmatter. Runs first and stops the rest: everything below reads structure out of this document, and so does `load`. A malformed document yields a plausible wrong ledger rather than an error.
1. **Frontmatter** — `id` and `title` present; `id` matches the `<ACR>-F###` or `<ACR>-Q###` shape.
2. **ID is free** — not already in `CHANGELOG.md` and not already in `scope/done/`.
3. **Tasks exist** — at least one `### T<n>`, each with at least one acceptance criterion.
4. **No placeholders** — no `TBD`, no `TODO`, no "etc.", no "and similar", no "add error handling", no "as needed". A placeholder is an instruction to guess.
5. **Criteria are observable** — checkable without judgement. "Renders without a console error" passes; "looks good" and "works well" do not.
6. **Paths resolve** — any file path the document states as existing, exists.
7. **Binding sections match reality** — where the document names an interface, signature, or data shape that the code already defines differently, that is a conflict, not a detail.
8. **Out of Scope present.**
9. **Format version readable.**

**This action reports and stops there.** It never repairs a document. A failing AxiomCore document is `/dev revise` input for the user to carry back; a failing quick document is redrafted with `/scope quick`.

## If action is "load"

1. Read `context/current-feature.md`. If `status` is anything but `idle`, refuse: a feature is already active. Name it and point at `/implement complete` or `/implement blocked`.
2. Run **check**. On any failure, refuse to load and report the gaps. Do not offer to load anyway.
3. Parse the document into the ledger:
   - Frontmatter → ledger frontmatter. **An absent `origin` field means `axiomcore`** — documents from AxiomCore do not carry one and must not be required to. Carry the document's own `version` across; there is no `spec_version` on either side, and a document pins no other document's version.
   - Each `### T<n>` → one ledger row, status `pending`, criteria counted.
   - Resolve **binding standing** per design section: Data & Error Flow and any public interface are always binding; every other section is binding unless it carries the "Proposed reference design" banner, in which case it is proposed.
   - Copy `Out of Scope` verbatim.
4. Write the ledger, `status: loaded`. **Do not branch and do not touch the working tree** — that is `/implement start`.
5. Report: N tasks, which sections are binding, what is out of scope. Then point at `/implement start` — which branches **and runs the tasks**, all of them unless given a number.

## If action is "standards"

Reads and extends `context/coding-standards.md` — the project's stack profile. `/scope init`
creates it; this keeps it current, so a correction is given once instead of every few weeks.

**`show`** (or no argument) — print the profile. Name any section still holding placeholder values,
and say plainly what each unfilled section costs: no `Commands` means `/verify` cannot run the
tests or build; no `Never` list means the agent and `code-scanner` both work from general practice
rather than this project's rules.

**`add <rule>`** — record a new entry.

1. Work out which section it belongs to — `Commands`, `Stack`, `Conventions`, `Patterns`, `Never`,
   `Testing`, `Notes`. Ask only if it is genuinely ambiguous.
2. **A `Never` entry requires its reason.** If the user gave one, use it. If not, ask — a bare
   prohibition tells the agent what not to type, while the reason tells it what the rule is
   protecting, which is what lets it judge a case the rule never anticipated. This is the one
   thing worth a follow-up question.
3. Write it in the existing voice of that section. Do not restructure the file, and do not
   reword entries you were not asked to touch.
4. Show the added line and where it landed.

Most entries arrive as corrections during implementation — "no, not like that". That is the moment
worth capturing, while the reason is still obvious.

## If action is "archive"

1. Move the document from `scope/` to `scope/done/`. The filename does not change — `CHANGELOG.md` links to it.
2. If the document is still loaded in the ledger, refuse unless the ledger's status is `idle`.
3. Normally invoked by `/implement complete`. Called directly, it is for abandoning a feature — say so and confirm before moving.

---

## Markdown discipline

Every markdown file you write or edit is lint-clean before you report it done. This skill owns four of them — `scope/README.md`, `scope/registry.md`, `context/coding-standards.md`, `context/current-feature.md` — plus each quick document at drafting time.

```bash
npx --yes markdownlint-cli2 --fix [--config <path>] <the file you just wrote>
```

Config resolution is the same three-step order `/verify` uses: the repository's own config if it has one (pass no `--config`, and never edit or replace it), else `$CLAUDE_PLUGIN_ROOT/skills/verify/markdownlint.json`, else `{ "default": true, "MD013": false }` written to a temp file outside the repository. Nothing is ever added to the user's project.

Two exceptions, both for the same reason — a document in `scope/` is read-only:

- **Never lint a document already in `scope/` or `scope/done/`**, with or without `--fix`. Not to tidy it, not to fix a table. `spec-checker` reports on their structure at `check`; nobody repairs them here.
- A **quick draft** is linted *before* it is shown for approval, while it is still yours.

If `npx` is unavailable, say so once and carry on. The rule is that the check is not skipped silently, not that work stops without a linter.

---

## After any action

Report as two short lists: what changed on disk, and what needs the user's decision. Never pad.

## Notes

- **A document in `scope/` is read-only.** This skill creates quick documents; it never edits any document, from either origin, after it is written. A change to an issued feature is a new document, never an edit to the old one.
- **`F###` documents arrive on their own.** AxiomCore writes them into `scope/` after its own pre-issue check passes, and re-writes the file when it re-issues a corrected version. Nothing here fetches, and nothing here is copied by hand. A document appearing without warning is normal; a document changing under you means AxiomCore revised it, and the version in the frontmatter says which one you now hold.
- **Never mint an `F###`.** That counter belongs to AxiomCore. Locally authored documents are always `Q###`.
- `check` and `list` are read-only and safe to run at any time.
- The three gates are not advisory. When one trips, the answer is a draft for AxiomCore, not a smaller version of the change.
