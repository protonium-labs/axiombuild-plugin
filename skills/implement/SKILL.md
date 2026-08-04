---
name: implement
description: Run the loaded feature, one task at a time. Actions - start (branch), next (execute exactly one task, then stop), status, explain, blocked (record a stop trigger and halt), commit (checkpoint on the branch), complete (squash-merge, changelog line, archive), overview. Never invents work beyond the loaded document.
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
| `start` | Create the feature branch and open the ledger for work |
| `next [T<n>]` | Execute exactly one task, then stop for validation |
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
5. Report the task list and point at `/implement next`. Do not start implementing.

## If action is "next"

Executes **exactly one task**, then stops. This is the two-phase rule at task granularity: one step, shown, validated, then the next.

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
7. **Stop.** Show the work for this task only — no preview of the next one, no summary of the whole feature.

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
   Feature: <ID> (spec <version>)
   Task:    T<n>
   Trigger: <ambiguity | binding conflict | beyond tasks | repeated failure>

   Situation: <what the document says vs what the code does>
   Question:  <the specific decision needed>
   Options:   <if there are obvious candidates, list them; otherwise omit>
   ```

   For an `F###` feature this is `/dev revise` input for AxiomCore. For a `Q###` feature, the fix is a redraft with `/scope quick`.
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

   `- **<date>** · \`<ID>\` · <Title verbatim> · [\`<filename>\`](scope/done/<filename>)`

   The title is copied from frontmatter exactly as written. Do not describe the feature — the description is the document being linked.
5. Archive: move the document from `scope/` to `scope/done/`, filename unchanged.
6. Amend the squash commit to include the changelog line and the archived document, so the feature remains one commit.
7. Reset `context/current-feature.md` to the idle state.
8. Ask before deleting the branch, and before pushing `main`.
9. Report: the commit, the changelog line, and the archive path.

---

## After any action

Report as two short lists: what changed on disk, and what needs the user's decision. Never pad, and never summarise work the user has just watched.

## Notes

- **The source document is read-only.** It is a contract, frozen at issuance. Never edit it — not to fix a typo, not to record what was built, not to note a deviation. Deviations live in the ledger.
- **Never invent scope.** Anything not in a `T<n>` is not in this feature, however obvious or cheap it looks.
- Do not refactor unrelated code, and do not add improvements nobody asked for.
- Do not add manual memoization where a compiler handles it, and follow whatever equivalent rules `context/coding-standards.md` sets for this stack.
- One task per `next`. The user validates between tasks; that gap is the point, not friction to optimise away.
