---
name: scope
description: Own the scope/ inbox and keep the books - which feature document is current, which are finished, which release each one became. Actions - init (scaffold a project), quick (draft a small feature locally), list, start (open a feature and cut its branch), done (archive it, write the changelog entry and set the release tag), standards, overview. Documents from AxiomCore are read-only and this skill never mints an F### number. It does not plan, implement or verify - superpowers does that.
---

# Task

Execute the requested action: $ARGUMENTS

Calling rule: `/scope <action> [argument]`.

If the requested action is ambiguous or contradictory, do NOT guess: flag it and ask for clarification before executing.

---

## What this skill is, and is not

It keeps the books. Which feature is open, which are finished, what the project's stack rules are.

It does **not** decompose a feature into tasks, write code, or verify anything. That is `superpowers`:

```text
scope/<doc>.md  →  /scope start <ID>
                →  superpowers:writing-plans        (the tasks, with the repo readable; reads docs/mockups/ too)
                →  superpowers:executing-plans      (TDD, commit per step)
                →  superpowers:verification-before-completion
                →  superpowers:requesting-code-review
                →  superpowers:finishing-a-development-branch
                →  /scope done <ID>
```

Entry is at `writing-plans`, **not** `brainstorming` — the product thinking was already done in AxiomCore, and brainstorming's hard gate would repeat it.

**A UI feature arrives with its mockups** (2.1.0). AxiomCore copies each screen the feature touches into `docs/mockups/` as self-contained HTML, and the document names them in its `mockups:` frontmatter. The document owns behaviour, the mockup owns where everything sits on the screen, and both are read-only here. Library documentation is checked via context7, never recalled from memory. There is no automated comparison against the mockup — the owner checks the running screen.

**Every finished feature and fix is a release** (2.2.0), where `scope/registry.md` says `Release tagging: on`. The number is `v<major>.<minor>.<patch>`. For an `F###` document the first two digits arrive in its `release:` frontmatter — the second digit is the feature number, so features may finish in any order — and are transcribed, never computed. For a `Q###` document the number is the highest existing tag with the third digit raised by one. `done` writes the number to `VERSION`, records the release in `CHANGELOG.md` and sets the tag on that commit. The tag labels a point in history; it triggers nothing. With release tagging off, or the line absent, none of this happens and the skill behaves as on 2.1.0.

---

## If action is empty

Describe this skill: its purpose and the actions table from "overview". Do not execute anything.

## If action is "overview" (or unknown)

| Argument | Action |
| :--- | :--- |
| `init` | Scaffold this project: `scope/`, `CHANGELOG.md`, the current-feature record, the stack profile |
| `quick <description>` | Draft a small feature document locally, subject to the three gates |
| `list` | What is waiting in `scope/`, and what is currently open |
| `start <ID>` | Open a feature: record it as current and cut its branch |
| `done <ID>` | Close it: move the document to `scope/done/`, write the changelog entry, set the release tag |
| `standards [show \| add <rule>]` | Read or extend the project's stack profile |
| `overview` | This table |

If the action was unknown, say so first, then show the table.

---

## If action is "init"

Scaffolds the project. **Non-destructive and idempotent**: anything that already exists is read, never overwritten. Running it twice is safe; running it in a project that predates this plugin adopts what is already there.

1. **Survey first.** List which target files already exist. Report them as "found" and leave their contents alone.
2. **Detect the stack.** Read `package.json`, `pyproject.toml` or the equivalent manifest for the framework, the test command, and the build command. Do not ask about anything already discoverable.
3. **Ask for the project acronym** — two or three uppercase letters, fixed for the project's life. Warn the user to check it against their AxiomCore project acronyms: the `F###` and `Q###` namespaces share this prefix, so a clash makes IDs ambiguous across the two systems.
4. **Ask about releases** — two yes/no questions, both answered `off` when skipped. *Release tagging*: should every finished feature and fix get a version number and a git tag? *Push releases*: should `done` push the bookkeeping commit and the tag itself, or report the command and leave the push to the owner? In a project that already has `scope/registry.md` without these lines, ask and add them; never change a value that is there.
5. **Interview for the stack profile** — language and version, test command, build command, lint command, framework conventions, anything the agent must never do (e.g. "never `prisma db push`"). Offer to skip; a skipped interview leaves a stub with the detected values only.
6. **Verify git.** If the folder is not a git repository, say so plainly — `start` cuts a branch and `superpowers:finishing-a-development-branch` merges one, and neither works without it. Do not run `git init` unasked.
7. **Check for superpowers.** If the `superpowers` plugin is not available in this project, say so: without it there is nothing here that plans or builds. Point at `/plugin install superpowers@claude-plugins-official`. Do not install it unasked.
8. **Create what is missing**, from the templates in this skill folder:

   | Path | Template |
   | :--- | :--- |
   | `scope/README.md` | `scope-readme-template.md` |
   | `scope/registry.md` | `registry-template.md` (acronym, `Next quick feature: Q001`, the two release settings) |
   | `scope/done/` | empty folder |
   | `CHANGELOG.md` | `changelog-template.md`, example rows stripped; `Format: **v2**` with release tagging on, `v1` with it off |
   | `VERSION` | only with release tagging on and no tag yet: one line, `v0.0.0` |
   | `context/current-feature.md` | `current-feature-template.md`, idle state |
   | `context/coding-standards.md` | interview answers, or a stub |

9. **Report** two short lists: created, and found-and-left-alone.

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
2. If no gate trips: draft the document from `quick-feature-template.md`. **Requirements, not tasks** — what the finished thing does, from the outside. The task breakdown is `superpowers:writing-plans`' job, after `start`. Fill `Out of Scope` with what a reasonable reader might assume is included but is not. With release tagging on, fill `release_note:` — one sentence in the customer's words, in the language of the application's user interface, saying what now works ("The column order in the table is kept after a reload"); leave it empty when a customer sees nothing of the change. With release tagging off, delete the line. The document carries no `release:` — the number is not known before `done`.
3. **Show the draft and wait for approval.** Nothing is written to `scope/` before the user agrees. Lint the draft before showing it (see *Markdown discipline*) — once it lands in `scope/` it is read-only, and a malformed one can only be fixed by redrafting the whole thing.
4. On approval: read `scope/registry.md`, allocate the next `Q###`, write `scope/<acr>-q###-<slug>.md`, increment the counter. Numbers are monotonic and never reused.
5. Offer to `start` it.

## If action is "list"

1. Read every document directly in `scope/` (not `done/`). For each, show: ID, title, origin, and version where the frontmatter carries one.
2. Read `context/current-feature.md` and show what is open, if anything.
3. If `scope/` is empty and nothing is open, say so and point at `/scope quick`. AxiomCore writes an `F###` document into `scope/` itself, so an empty inbox means nothing has been issued yet — not that a copy is waiting to be carried.

## If action is "start"

Opens a feature. This is bookkeeping plus one git command — it does not read the document's content beyond the frontmatter, and it plans nothing.

1. Read `context/current-feature.md`. If `status` is anything but `idle`, refuse: a feature is already open. Name it and point at `/scope done`.
2. Resolve `<ID>` to a document directly in `scope/`. If it is not there, say which documents are, and stop.
3. Confirm the ID is not already spent — not in `CHANGELOG.md`, not in `scope/done/`. A repeat means the feature was already delivered; ask before going further.
4. **Check the mockups.** Read `mockups:` from the frontmatter. Every file it names must exist in `docs/mockups/`. If one is missing, name it and stop — no branch is cut, because building a screen without its mockup is exactly the gap mockups exist to close. An absent or empty `mockups:` means the feature has no screen to draw and passes.
5. **Check the release**, with release tagging on. An `F###` document must carry `release:` in the form `v<number>.<number>.0`; if it is absent or malformed, say so and stop — the number is AxiomCore's to give, so the answer is a re-issued document, never a number invented here. If a tag of that name already exists, stop: the release was already made. A `Q###` document carries none and passes.
6. Cut the branch `feature/<ID>` from the default branch, working tree clean. If it is not clean, stop and say what is uncommitted.
7. Write `context/current-feature.md` from `current-feature-template.md`: `feature`, `title`, `origin`, `type`, `version`, `release`, `source`, `mockups`, `branch`, `status: open`, `started`. `release` is empty for a quick document and with release tagging off. Transcribe from the document's frontmatter — **an absent `origin` means `axiomcore`**, and the document's own `version` is carried across as it stands. No document pins another document's version.
8. Report the ID, the branch, and point at `superpowers:writing-plans` with the document path and the mockup paths. Do not plan the work yourself.

## If action is "done"

Closes a feature. Run it **after** `superpowers:finishing-a-development-branch` has merged the work — this action records what happened, it does not merge, test, or verify.

1. Read `context/current-feature.md`. If nothing is open, or the open feature is not `<ID>`, say so and stop.
2. Confirm the work is actually merged: the branch's commits are on the default branch and the working tree is clean. If not, say what is outstanding and stop — the books are not written before the work lands.
3. Move `scope/<file>` to `scope/done/<file>`. The filename does not change: `CHANGELOG.md` links to it.
4. **Determine the release**, with release tagging on. `F###`: transcribe `release:` from the document. `Q###`: take the highest existing tag in version order (`git tag --list "v*" --sort=-v:refname`, after `git fetch --tags` where there is a remote) and raise its third digit by one; with no tag yet the fix is `v0.0.1`. If a tag of the resulting name already exists, stop and say so.
5. Write the release to `VERSION` at the repository root — the number and a line end, nothing else. A build reads this file; it needs neither git history nor tags.
6. Prepend the changelog entry, format per `changelog-template.md`. The title is **transcribed verbatim** from the document's `title` frontmatter, the customer line verbatim from `release_note:`; an empty or absent `release_note:` means no customer line. Do not compose prose here — the description lives in the archived document.
7. Reset `context/current-feature.md` to its idle state.
8. Commit the bookkeeping: the moved document, `CHANGELOG.md`, `VERSION`, `context/current-feature.md`. Message `chore(<ID>): archive and record <release>`.
9. Set an annotated tag on that commit: `git tag -a <release> -m "<ID> <title>"`.
10. **Push**, where there is a remote. `Push releases: on`: `git push --follow-tags` on the default branch. Off: report that command and leave it to the owner.
11. Report what moved, the release, what the changelog entry says, and whether it was pushed.

With release tagging off, steps 4, 5, 9 and 10 are skipped, the changelog line is format v1 and the commit message carries no release.

**Abandoning a feature** rather than finishing it: say so explicitly, confirm with the user, move the document to `scope/done/` without a changelog entry, a `VERSION` change or a tag, and note in the report that none was written. The release number of an abandoned `F###` stays unused.

## If action is "standards"

Reads and extends `context/coding-standards.md` — the project's stack profile. `/scope init`
creates it; this keeps it current, so a correction is given once instead of every few weeks.

**`show`** (or no argument) — print the profile. Name any section still holding placeholder values,
and say plainly what each unfilled section costs: no `Commands` means the test and build commands
are rediscovered every session; no `Never` list means the agent works from general practice rather
than this project's rules.

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

---

## Markdown discipline

Every markdown file you write or edit is lint-clean before you report it done. This skill owns four of them — `scope/README.md`, `scope/registry.md`, `context/coding-standards.md`, `context/current-feature.md` — plus each quick document at drafting time.

```bash
npx --yes markdownlint-cli2 --fix [--config <path>] <the file you just wrote>
```

Config resolution, in order: the repository's own config if it has one (pass no `--config`, and never edit or replace it), else `$CLAUDE_PLUGIN_ROOT/skills/scope/markdownlint.json`, else `{ "default": true, "MD013": false }` written to a temp file outside the repository. Nothing is ever added to the user's project.

Two exceptions, both for the same reason — a document in `scope/` is read-only:

- **Never lint a document already in `scope/` or `scope/done/`**, with or without `--fix`. Not to tidy it, not to fix a table.
- A **quick draft** is linted *before* it is shown for approval, while it is still yours.

If `npx` is unavailable, say so once and carry on. The rule is that the check is not skipped silently, not that work stops without a linter.

---

## After any action

Report as two short lists: what changed on disk, and what needs the user's decision. Never pad.

## Notes

- **A mockup in `docs/mockups/` is read-only**, like the document that names it. A screen that cannot be built as drawn is a question for AxiomCore, which corrects the mockup and re-issues it.
- **A document in `scope/` is read-only.** This skill creates quick documents; it never edits any document, from either origin, after it is written. A change to an issued feature is a new document, never an edit to the old one.
- **An issued feature that cannot be built is a question, not an edit.** Stop and ask the user. The answer is authored in AxiomCore, the version bumped, and the corrected copy re-issued into `scope/`. An implementing agent must never be able to move the target it is measured against.
- **`F###` documents arrive on their own.** AxiomCore writes them into `scope/`, and re-writes the file when it re-issues a corrected version. Nothing here fetches, and nothing here is copied by hand. A document appearing without warning is normal; a document changing under you means AxiomCore revised it, and the version in the frontmatter says which one you now hold.
- **Never mint an `F###`.** That counter belongs to AxiomCore. Locally authored documents are always `Q###`.
- **Never choose the first two digits of a release.** They arrive in `release:`. The only digit counted here is the third, and only for a quick document.
- **A tag is never moved or deleted** to repair a mistake. A wrong release is followed by a corrected one.
- **A feature document states requirements, not tasks.** If you want a task list before `writing-plans` has run, you are doing that skill's job.
- `list` and `standards show` are read-only and safe to run at any time.
- The three gates are not advisory. When one trips, the answer is a draft for AxiomCore, not a smaller version of the change.
