---
name: implement
description: Run the loaded feature. Actions - start [all|N] (branch, then run tasks back to back, committing after each), next [all|N] (run one task, or resume a run), status, explain, blocked (record a stop trigger and halt), commit (checkpoint on the branch), complete (squash-merge, changelog line, archive), overview. Stops on a failing criterion, a failing test, or any stop trigger. Never invents work beyond the loaded document.
---

# Task

Execute the requested action: $ARGUMENTS

Calling rule: `/implement <action> [argument]`.

If the requested action is ambiguous or contradictory, do NOT guess: flag it and ask for clarification before executing.

---

## The four stop triggers

These apply to **every** action in this skill, not to one section. When any of them fires, go to **blocked**. Do not improvise, do not pick the most likely reading, do not build a smaller version.

| Trigger | Fires when |
| :--- | :--- |
| Spec ambiguity | A task or criterion can be read two ways, or a detail needed to build it is simply absent |
| Binding conflict | A binding section states an interface, signature, or data shape that the code already defines differently |
| Work beyond the tasks | Delivering a task needs something no `T<n>` covers — a refactor, a migration, a dependency |
| Repeated failure | The same problem survives two or three genuine attempts |

**Blocked is a successful outcome.** A stopped feature with a precise question is worth more than a finished feature built on a guess.

---

## Stopping, and how you say so

A run stops on any of these. The first three are failures; the last two are simply the end.

| Stop | What it means |
| :--- | :--- |
| A criterion fails `/verify T<n>` | The task did not do what the document asked |
| The test command fails | Something that used to work no longer does |
| One of the four stop triggers fires | Go to **blocked**, which owns that path |
| The budget is spent | Not a failure. Report and wait |
| No pending task remains | Run `/verify all`, report the matrix, stop |

**The work stays where it is.** A failed task is not reverted and not committed — it is usually
mostly right, and throwing it away costs more than leaving it for the next instruction. Say plainly
that it is uncommitted, so the state of the branch is never a surprise.

**Never start the next task after a stop.** Not to be helpful, not because the next one looks
unrelated.

### The language a stop is reported in

Applies to every stop, to `blocked`, and to any question this skill puts to the user.

*Who you are writing for.* Someone who knows HTML, CSS, JavaScript and common frameworks — Django,
Next.js — and has **never worked in IT**. Concrete technical words are fine and expected: a test, a
function, a branch, a date range, a default value, a commit. What is not fine is jargon and in-house
shorthand — abbreviations, criterion numbers as the subject, file paths standing in for an
explanation, and any term that means something only inside this codebase. **Do not talk down
either**: no analogies, no explaining what a test is. Vague is not the same as plain.

Three or four lines, not a report:

- Good: *"`T4` stopped. The test that checks the year range now fails — the slider's lowest year
  comes back as text, not a number, and the comparison rejects it. I can convert it where the value
  is read, or fix it in the slider itself. Which?"*
- Bad: *"T4/C3 assertion failure: `TypeError` in `test_year_bounds`, expected `int` got `str`
  (`views/timeseries.py:88`)."*
- Also bad: *"Something went wrong with the years."*

The technical detail — the traceback, the exact line, the failing assertion — is **held and given
only if the user asks for it**. It is never volunteered and never attached to the question.

---

## Capturing corrections

When the user corrects **how** you work — a pattern, a convention, a prohibition — offer once to record it in the stack profile:

> Should I add "queries live in `src/lib/db`, never in a component" to the Never list?

On yes, hand it to `/scope standards add` with the reason the user gave. On no, drop it and never raise that one again.

The test is a single question: **would this apply to the next feature too?**

- *"Use the theme token, never a raw hex colour"* — yes. That is a rule. Offer.
- *"Make this button blue"* — no. That is a decision about this feature. Say nothing.

Rules:

- Offer **once**, immediately after fixing what was corrected. Never re-ask, never batch several offers, never raise it again later in the session.
- One line. Do not explain why recording it is a good idea.
- When unsure which side of the test a correction falls on, stay quiet. A missed rule costs one repetition later; a stream of unnecessary questions costs the user's willingness to answer any of them.
- Never write to the profile yourself without being told to.

This is the mechanism by which the profile grows. Without it, a correction lives only in the session it was given in, and the same mistake returns in three weeks.

---

## If action is empty

Describe this skill: its purpose and the actions table from "overview". Do not execute anything.

## If action is "overview" (or unknown)

| Argument | Action |
| :--- | :--- |
| `start [all\|N]` | Create the feature branch, then run the tasks. No argument runs all of them |
| `next [T<n>\|all\|N]` | Run one task, a named task, or resume a run of N (or all) remaining |
| `status` | Ledger state: tasks, progress, branch, anything blocked |
| `explain` | What changed on this branch, file by file, and how it connects |
| `blocked <question>` | Record a stop trigger, halt, and produce a question for AxiomCore |
| `commit [message]` | Checkpoint the current work on the branch |
| `complete` | Squash-merge, write the changelog line, archive the document, reset |
| `overview` | This table |

If the action was unknown, say so first, then show the table.

---

## If action is "start"

1. Read `context/current-feature.md`. Require `status: loaded`. If `idle`, point at `/scope load`; if already `in-progress`, say so and stop.
2. **Require a clean working tree.** Uncommitted changes plus a new branch is exactly the mess this workflow exists to prevent. Report what is dirty and stop.
3. Create and check out the branch: `feature/<ID>` when `origin` is `axiomcore`, `quick/<ID>` when it is `local`.
4. Set `status: in-progress`, record `branch` and `started`.
5. **Read the run budget from the argument** and record it in the ledger as `budget`:
   - no argument, or `all` — every pending task;
   - a number `N` — the next `N` pending tasks;
   - `0` — branch only, implement nothing. This is the old behaviour, kept for when you want to read
     the tasks before any code is written.
6. Report the task list and the budget, then **run the loop** (below). Do not ask first: the budget
   is the permission.

## If action is "next"

`next` alone executes **exactly one task** and stops — the manual mode, for when you want to look
at the work before more is written. `next all` or `next N` sets a new budget and resumes the loop,
which is how a run that stopped is picked back up.

1. Read the ledger. Take the `T<n>` given as argument, or the first task with status `pending`. If every task is `done`, say so and point at `/verify all`.
2. Read the **source document** for that task's full text and criteria — the ledger row is a summary, not the spec.
3. Read `context/coding-standards.md` before writing any code. Preserve existing patterns in the codebase; match them rather than improving them.
4. Mark the task `in-progress`. Implement it:
   - Make the minimal change that satisfies the task. Nothing beyond it.
   - `Out of Scope` in the ledger is a prohibition, not a hint.
   - Honour binding standing: a binding section is a requirement; a proposed one may be deviated from, and the deviation gets recorded in the ledger.
   - Watch the four stop triggers throughout. Any one fires → **blocked**.
5. Self-check the task's acceptance criteria and record the count (`2/3`). This is a smoke test so obviously incomplete work is not handed back — it is **not** verification. `/verify` owns that, in a separate context.
6. If every criterion passes, mark `done`. If not, report which failed and stay `in-progress`.
7. **Gate the task before it counts as finished**, in this order. Any failure stops the run (see
   *Stopping*):
   - **`/verify T<n>`** — the criteria matrix for this task, judged by `criteria-verifier` in a
     separate context. The context that wrote the code does not decide whether it met the spec.
   - **The project's test command** from `context/coding-standards.md`. A regression introduced at
     `T3` and found at `T13` costs far more than the seconds this takes. If no test command is
     defined, say so once per run and carry on.
8. **Commit.** `<type>(<ID>): T<n> <what was done>`, exactly as the `commit` action defines it.
   **This happens after every successful task, before the next one starts** — the commit boundary is
   how the work is read afterwards, so it is never batched and never deferred to the end of a run.
9. **Decrement the budget and continue** to the next pending task at step 1. When the budget is
   spent, or no pending task remains, stop and report.
10. **At the end of a run that finished every task**, run `/verify all` and report the matrix.
    `complete` is **not** run — it merges to `main` and keeps its own confirmation.

## If action is "status"

Read the ledger and report: feature ID and title, origin, branch, the task table with progress, any blocked entries, and the single next action. Read-only.

## If action is "explain"

1. `git diff <branch-point> --name-only` for the files this feature touched.
2. Per file: path, whether new or modified, one or two sentences on what it does and why it changed.
3. Close with how the pieces connect — the data or control flow between them.

Read-only. Useful before `/verify` or when picking a feature back up after a break.

## If action is "blocked"

1. Append an entry to the ledger's `## Blocked` section: which task, **which of the four triggers fired**, the situation in one or two sentences, and the decision needed.
2. Set the task's status and the ledger's status to `blocked`.
3. Produce a question block for the user to carry back:

   ```text
   Feature: <ID> v<document version>
   Task:    T<n>
   Trigger: <ambiguity | binding conflict | beyond tasks | repeated failure>

   Situation: <what the document says vs what the code does>
   Question:  <the specific decision needed>
   Options:   <if there are obvious candidates, list them; otherwise omit>
   ```

   The version is the feature document's own, from its frontmatter — no spec version is pinned
   anywhere, on either side.

   Say it to the user in the language *Stopping* defines, and keep the block above for the record
   rather than as the thing you show them. For an `F###` feature the answer is authored in AxiomCore
   and the corrected document is re-issued into `scope/` automatically; for a `Q###` feature the fix
   is a redraft with `/scope quick`.
4. **Stop implementing.** Do not work around it, and do not move to the next task while a blocker stands.

## If action is "commit"

A checkpoint **on the feature branch**. It does not merge, and it will be squashed away at `complete` — so it is cheap and can be run often.

1. Require the feature branch to be checked out. Refuse on `main`.
2. `git status` first. Never stage gitignored or machine-local files (`.claude/settings.local.json` and the like).
3. Message: `<type>(<ID>): T<n> <what was done>`. Conventional type, feature ID as scope.
4. No `Co-Authored-By`, no "Generated with Claude", no attribution of any kind.
5. Show the new commit and the resulting `git status`.

A checkpoint is not required to build. Stopping after each task means a mid-feature tree is often legitimately incomplete, and the gate that matters is at `complete`.

## If action is "complete"

The only action that touches `main`. Ask before merging and before pushing.

1. **Require every task `done`.** Otherwise list what is outstanding and stop.
2. **Require `/verify all` to have passed** on the current state. Not run, or run and failed, means stop — this is the gate, and it does not get waived.
3. Squash-merge, so `main` carries exactly one commit for this feature:

   ```bash
   git checkout main
   git merge --squash <branch>
   git commit -m "<type>(<ID>): <subject>"
   ```

   The subject is the document's `title`, lowercased. **Transcribed, never composed.**
4. Write the `CHANGELOG.md` line at the **top** of the list:

   ```text
   - **<date>** · `<ID>` · <Title verbatim> · [`<filename>`](scope/done/<filename>)
   ```

   The title is copied from frontmatter exactly as written. Do not describe the feature — the description is the document being linked.

   Then lint it (see *Markdown discipline*). This file is appended to on every completed feature and is the most-edited markdown in the repository — it is where formatting drift accumulates if nothing checks it.
5. Archive: move the document from `scope/` to `scope/done/`, filename unchanged.
6. Amend the squash commit to include the changelog line and the archived document, so the feature remains one commit.
7. Reset `context/current-feature.md` to the idle state.
8. Ask before deleting the branch, and before pushing `main`.
9. Report: the commit, the changelog line, and the archive path.

---

## Markdown discipline

Every markdown file you write or edit is lint-clean before you report it done — `CHANGELOG.md` and `context/current-feature.md` here, and any documentation a task explicitly asks for.

```bash
npx --yes markdownlint-cli2 --fix [--config <path>] <the file you just wrote>
```

Config resolution is the same three-step order `/verify` uses: the repository's own config if it has one (pass no `--config`, and never edit or replace it), else `$CLAUDE_PLUGIN_ROOT/skills/verify/markdownlint.json`, else `{ "default": true, "MD013": false }` written to a temp file outside the repository.

Two things this rule is not:

- **Not a licence to touch other markdown.** A README you were not asked to edit is out of scope like any other file. Linting it is scope drift, and `/verify` will flag the diff.
- **Not applicable to the source document.** It is read-only. That holds for the linter exactly as it holds for you.

If `npx` is unavailable, say so once and carry on.

---

## After any action

Report as two short lists: what changed on disk, and what needs the user's decision. Never pad, and never summarise work the user has just watched.

## Notes

- **The source document is read-only.** It is a contract, frozen at issuance. Never edit it — not to fix a typo, not to record what was built, not to note a deviation. Deviations live in the ledger.
- **Never invent scope.** Anything not in a `T<n>` is not in this feature, however obvious or cheap it looks.
- Do not refactor unrelated code, and do not add improvements nobody asked for.
- Do not add manual memoization where a compiler handles it, and follow whatever equivalent rules `context/coding-standards.md` sets for this stack.
- **Validation between tasks moved; it was not removed.** `next` used to stop after every task so the user could look. A run now gates each task with `/verify T<n>` — `criteria-verifier`, in a separate context, judging the diff against the criteria — and with the project's tests, then commits. That is a stronger check than a glance, and it costs the user nothing. `next` with no argument still runs a single task for when you want to look anyway.
- **A commit per successful task is not optional.** It is how the run is read afterwards: one commit, one task, in order. Never batch them, never defer them to the end of a run.
