# AxiomBuild

Implementation agent for coding projects. Planning happens elsewhere; this builds what was planned.

AxiomBuild reads a feature document and builds it task by task, gating each one against the
document's own acceptance criteria before committing it and moving on, then records the feature in a
structured changelog matched to its commit. A run goes end to end on its own and stops the moment
something does not pass. It does not decide *what* to build — it refuses to guess, and stops to ask
instead.

Pairs with [AxiomCore](https://github.com/protonium-labs/axiomcore-plugin), which produces the
feature documents. Neither requires the other: AxiomBuild can draft small features itself, and
AxiomCore's documents are plain markdown.

## Install

```text
/plugin marketplace add protonium-labs/protonium-marketplace
/plugin install axiombuild@protonium
```

Then, in the coding project:

```text
/scope init
```

Skills are namespaced by the plugin, so they are invoked as `/axiombuild:scope`,
`/axiombuild:implement` and `/axiombuild:verify`.

## The loop

```text
        AxiomCore ──writes──┐
                            ▼
        /scope quick ──►  scope/  ──►  /scope check  ──►  /scope load
                                                                │
                                                                ▼
                                            /implement start [all|N]
                                                                │
                                            ┌── task ─► verify ─► test ─► commit ──┐
                                            │            │                         │
                                            └────────────┼─────────── next task ◄──┘
                                                         ▼
                                                   anything fails → stop and ask
                                                                │
                                                                ▼
                                             /verify all ──► /implement complete
                                                                │
                                      ┌─────────────────────────┤
                                      ▼                         ▼
                              CHANGELOG.md               scope/done/
```

One squashed commit per feature on `main`. One changelog line per feature. The feature ID links them.

## Two on-ramps

**From AxiomCore** — for anything that deserves planning. AxiomCore writes the feature document into
`scope/` itself, once it has passed its own pre-issue check, and rewrites it when it re-issues a
corrected version. Nothing is copied by hand. The document is authoritative and read-only here.

**From `/scope quick`** — for a change too small to justify a planning round. You describe it, the
agent drafts a minimal document for your approval. Three gates send it back to AxiomCore instead:

| Gate | Trips when the change requires... |
| :--- | :--- |
| New dependency | A package not already in the manifest |
| Schema change | A migration, or an edit to the data model |
| Public interface | A new or changed endpoint, exported signature, or action contract |

Gates are assessed against the code, not against your description. When one trips the agent
refuses and hands you a draft ready for AxiomCore's `/dev feature` — it does not offer to
proceed anyway, and it does not build a smaller version.

## Skills

| Skill | Actions |
| :--- | :--- |
| `/scope` | `init` · `quick` · `list` · `check` · `load` · `archive` · `standards` |
| `/implement` | `start [all\|N]` · `next [T<n>\|all\|N]` · `status` · `explain` · `blocked` · `commit` · `complete` |
| `/verify` | `all` · `T<n>` · `criteria` |

## Agents

| Agent | Role |
| :--- | :--- |
| `spec-checker` | Pre-flight: is this document implementable without guessing? Reports, never repairs |
| `criteria-verifier` | Judges the diff against the criteria, in a context that did not write the code |
| `code-scanner` | Audits recent changes for security, performance, quality, componentization |

## Protocols

### Identifiers

| Form | Origin | Counter |
| :--- | :--- | :--- |
| `<ACR>-F###` | AxiomCore | AxiomCore's `features.md`. This agent never mints one |
| `<ACR>-Q###` | `/scope quick` | `scope/registry.md` |

Monotonic, never reused. A document without an `origin` field is treated as AxiomCore's, so
existing feature documents need no change.

### Commits

```text
<type>(<ID>): <subject>

feat(SP-F012): favorites page
style(SP-Q001): change card border colour
```

Conventional type, feature ID as scope, subject transcribed from the document's `title`.
Branch `feature/<ID>` or `quick/<ID>`. **One commit per task, written as soon as that task passes its
criteria and the tests** (`feat(SP-F012): T2 build the /favorites route`) — that sequence is how a
finished run is read. They are squashed away at completion.
No attribution lines, ever.

### Changelog

```markdown
- **2026-08-04** · `SP-F012` · Favorites page · [`sp-01-f012-favorites.md`](scope/done/sp-01-f012-favorites.md)
```

Latest on top. The line **references** the feature — it never describes it. The title is copied
verbatim from frontmatter; the agent does not write prose here, because the description already
exists in the linked document. No SHA: the ID is in the commit message, so
`git log --grep="(SP-F012)"` finds it, and an ID survives rebases where a SHA does not.

## Hard rules

1. **The document in `scope/` is read-only and authoritative.** Never edited — not to fix a typo,
   not to record what was built. A change to an issued feature is a new document.
2. **Blocked beats guessing.** A stopped feature with a precise question is worth more than a
   finished feature built on an assumption.
3. **Binding is binding.** Data & Error Flow and any public interface always. Other design
   sections too, unless marked as a proposed reference design — deviation there is legal and
   gets recorded.
4. **One task per `next`.** Shown, validated, then the next. The gap is the point.
5. **Out of Scope is a prohibition**, not a hint.

## Stop triggers

Any of these halts implementation and produces a question rather than a decision:

| Trigger | Fires when |
| :--- | :--- |
| Spec ambiguity | A task reads two ways, or a needed detail is absent |
| Binding conflict | A binding section contradicts what the code already defines |
| Work beyond the tasks | Delivery needs something no task covers |
| Repeated failure | The same problem survives two or three real attempts |

## What lives where

The plugin ships logic. The project owns state. Updating the plugin never touches your work.

| Plugin — updates with the marketplace | Project — yours, untouched by an update |
| :--- | :--- |
| `skills/`, `agents/`, templates | `scope/`, `scope/done/`, `scope/registry.md` |
| Workflow, gates, protocols | `CHANGELOG.md`, `context/current-feature.md` |
| | `context/coding-standards.md` |

The plugin does not write to your project's `CLAUDE.md`. Rules copied into a project file freeze
at install time; rules that live in the skills travel with every update.

## License

MIT
