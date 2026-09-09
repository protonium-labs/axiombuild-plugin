# AxiomBuild

Bookkeeping for spec-driven coding projects. It owns the inbox of feature documents waiting to be
built, records which one is open, and archives each one with a changelog line when it lands.

It does not plan, implement or verify. From **2.0.0** that work belongs to
[superpowers](https://github.com/obra/superpowers), which plans the tasks with the repository
readable and runnable — where they can actually be got right.

Pairs with [AxiomCore](https://github.com/protonium-labs/axiomcore-plugin), which produces the
feature documents. AxiomCore's documents are plain markdown, so neither strictly requires the other,
but `superpowers` is a real dependency: without it nothing here builds anything.

## Install

```text
/plugin marketplace add protonium-labs/protonium-marketplace
/plugin install axiombuild@protonium
/plugin install superpowers@claude-plugins-official
```

Then, in the coding project:

```text
/scope init
```

Skills are namespaced by the plugin, so `/scope` is invoked as `/axiombuild:scope`.

## The loop

```text
        AxiomCore ──writes──┐
                            ▼
        /scope quick ──►  scope/  ──►  /scope start <ID>
                                              │          (record it, cut feature/<ID>)
                                              ▼
                            superpowers:writing-plans          the tasks
                                              ▼
                            superpowers:executing-plans        TDD, commit per step
                                              ▼
                            verification-before-completion
                                              ▼
                            requesting-code-review             fresh context, reads the diff
                                              ▼
                            finishing-a-development-branch     merge
                                              ▼
                                     /scope done <ID>
                                              │
                            ┌─────────────────┤
                            ▼                 ▼
                     CHANGELOG.md        scope/done/
```

Entry is at `writing-plans`, **not** `brainstorming` — the product thinking already happened in
AxiomCore, and repeating it is the ceremony 2.0.0 exists to remove.

One changelog line per feature. The feature ID links it to the commits.

## Why 2.0.0 dropped implementation

Measured on a real project, ten features in: **6,282 lines** of feature document produced **5,821
lines** of shipped code, and those ten documents carried **28 versions** between them.

The cause was structural. A feature document written by a planner that has never run the code was
asserting how the code should be arranged — file trees, task breakdowns, per-task acceptance
criteria, interface signatures. Those assertions were wrong often. Each wrong one became a
clarification, a version bump, a re-check and a re-issue.

So the document now states **requirements**, and the task plan is written where the code is. Three
checking agents went with the change: they existed to make thousand-line documents safe, and a
forty-line requirements document does not need them.

## Two on-ramps

**From AxiomCore** — for anything that deserves planning. AxiomCore writes the feature document into
`scope/` itself and rewrites it when it re-issues a corrected version. Nothing is copied by hand.
The document is authoritative and read-only here.

**From `/scope quick`** — for a change too small to justify a planning round. You describe it, the
agent drafts a minimal requirements document for your approval. Three gates send it back to
AxiomCore instead:

| Gate | Trips when the change requires... |
| :--- | :--- |
| New dependency | A package not already in the manifest |
| Schema change | A migration, or an edit to the data model |
| Public interface | A new or changed endpoint, exported signature, or action contract |

Gates are assessed against the code, not against your description. When one trips the agent
refuses and hands you a draft ready for AxiomCore's `/dev feature` — it does not offer to proceed
anyway, and it does not build a smaller version.

## Skill

| Skill | Actions |
| :--- | :--- |
| `/scope` | `init` · `quick` · `list` · `start` · `done` · `standards` · `overview` |

## Protocols

### Identifiers

| Form | Origin | Counter |
| :--- | :--- | :--- |
| `<ACR>-F###` | AxiomCore | AxiomCore's `features.md`. This agent never mints one |
| `<ACR>-Q###` | `/scope quick` | `scope/registry.md` |

Monotonic, never reused. A document without an `origin` field is treated as AxiomCore's, so
existing feature documents need no change.

### Changelog

```markdown
- **2026-08-04** · `SP-F012` · Favorites page · [`sp-01-f012-favorites.md`](scope/done/sp-01-f012-favorites.md)
```

Latest on top. The line **references** the feature — it never describes it. The title is copied
verbatim from frontmatter; the agent does not write prose here, because the description already
exists in the linked document. No SHA: the ID is in the commit messages, so
`git log --grep="(SP-F012)"` finds them, and an ID survives rebases where a SHA does not.

## Hard rules

1. **The document in `scope/` is read-only and authoritative.** Never edited — not to fix a typo,
   not to record what was built. A change to an issued feature is a new document.
2. **Blocked beats guessing.** A stopped feature with a precise question is worth more than a
   finished feature built on an assumption. The answer is authored in AxiomCore and comes back as
   a bumped version; an implementing agent must not be able to move the target it is measured
   against.
3. **Requirements, not tasks.** A feature document says what the finished thing does. Wanting a
   task list before `writing-plans` has run means doing that skill's job.
4. **Out of Scope is a prohibition**, not a hint.

## What lives where

The plugin ships logic. The project owns state. Updating the plugin never touches your work.

| Plugin — updates with the marketplace | Project — yours, untouched by an update |
| :--- | :--- |
| `skills/`, templates | `scope/`, `scope/done/`, `scope/registry.md` |
| Workflow, gates, protocols | `CHANGELOG.md`, `context/current-feature.md` |
| | `context/coding-standards.md` |

The plugin does not write to your project's `CLAUDE.md`. Rules copied into a project file freeze
at install time; rules that live in the skills travel with every update.

## Upgrading from 1.x

`/implement` and `/verify` are gone, with the `spec-checker`, `criteria-verifier` and
`code-scanner` agents. `/scope check` and `/scope load` are gone too. Replacements:

| 1.x | 2.0.0 |
| :--- | :--- |
| `/scope check` | nothing — a requirements document is read by a person in under a minute |
| `/scope load` | `/scope start` — records the feature and cuts the branch, no task ledger |
| `/implement start` / `next` | `superpowers:writing-plans` then `executing-plans` |
| `/verify all` | `superpowers:verification-before-completion` and `requesting-code-review` |
| `/implement complete` | `superpowers:finishing-a-development-branch` then `/scope done` |

Your project's `scope/`, `CHANGELOG.md` and `context/` are untouched. `context/current-feature.md`
keeps its shape minus the task ledger and binding tables; an in-flight feature should be finished
on 1.x before updating.

## License

MIT
