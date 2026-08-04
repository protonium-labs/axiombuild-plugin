# Coding Standards

Format: **v1**

<!--
The project's stack profile. This is the one file that makes the agent portable: the skills and
agents carry no stack knowledge, they read it from here.

Who reads what:
  /implement next   — Conventions, Patterns, Never (before writing any code)
  /verify all       — Commands
  code-scanner      — Stack, Patterns, Never (a finding must not contradict them)

Written by /scope init from a short interview plus whatever it can detect from the manifest.
Grows over time: every "no, not like that" is a line that belongs here, so it does not have to
be said twice.
-->

## Commands

<!-- Machine-consumed by /verify. Exact commands, as typed. -->

| Purpose | Command |
| :--- | :--- |
| Test | `<npm test>` |
| Build | `<npm run build>` |
| Lint | `<npm run lint>` |

## Stack

<!-- Language, framework, database, styling, and the versions that change how code is written.
     A major version matters when it changes the idiom (Tailwind v3 vs v4, React 18 vs 19). -->

| Layer | Choice |
| :--- | :--- |
| Language | `<TypeScript, strict mode>` |
| Framework | `<Next.js 16, App Router>` |
| Database | `<Prisma 7 + PostgreSQL>` |
| Styling | `<Tailwind CSS v4>` |

## Conventions

<!-- Where things go and what they are called. Enough that a new file lands in the right place
     with the right name without asking. -->

- Components: `<src/components/[feature]/ComponentName.tsx>`
- Server actions: `<src/actions/[feature].ts>`
- Utilities: `<src/lib/[utility].ts>`
- Naming: `<components PascalCase, functions camelCase, constants SCREAMING_SNAKE_CASE>`

## Patterns

<!-- Positive rules: the shapes already used here, which new code should match rather than
     improve on. Consistency beats individually-better choices. -->

- `<Server components fetch data directly; client components go through server actions>`
- `<Server actions return { success, data, error } and are validated with Zod>`
- `<Errors surface to the user as a toast, never as a raw exception>`

## Never

<!--
The highest-value section. Explicit prohibitions, each with its reason — the reason is what lets
the agent judge an edge case instead of following the rule off a cliff.

Every one of these usually starts life as a correction. Add to it rather than repeating yourself.
code-scanner also reads this list and will not recommend against anything on it.
-->

- **Never `<create tailwind.config.js>`** — `<v4 is CSS-first; theme config belongs in @theme in globals.css>`
- **Never `<use prisma db push>`** — `<schema changes go through migrate dev so migrations stay in sync>`
- **Never `<use the any type>`** — `<use unknown and narrow it>`
- **Never `<add manual useMemo / useCallback / React.memo>`** — `<the React Compiler handles memoization>`

## Testing

<!-- What is covered, what deliberately is not, and where tests live. The deliberate exclusions
     matter as much as the inclusions — without them the agent writes tests nobody wanted. -->

- Runner: `<Vitest>`
- Scope: `<server actions and utilities only — never components; components are verified in the browser>`
- Location: `<beside the code they cover: src/lib/format.ts -> src/lib/format.test.ts>`
- Always covered: `<security boundaries — ownership scoping, auth checks, token namespacing>`

## Notes

<!-- Anything else the agent should know before touching this codebase. Delete if empty. -->
