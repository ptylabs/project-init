# AGENTS.md

Generic working agreement for AI coding agents. It is project-agnostic: every
project-specific fact lives in [`PROJECT.md`](PROJECT.md), and each person's
notes live in a local `MEMORY.md`.

| File | Holds | Shared |
| --- | --- | --- |
| `AGENTS.md` | How to work, for any product | Committed; starts from the public template and evolves through promotions |
| `PROJECT.md` | Product, stack, responsibilities, conventions, commands, CI | Committed, one per project |
| `MEMORY.md` | Personal patterns: tried → corrected → result | Local, git-ignored |
| `CLAUDE.md` | `@AGENTS.md`, `@PROJECT.md`, `@MEMORY.md` (Claude Code only reads `CLAUDE.md`) | Committed |

At the start of every session read `AGENTS.md`, `PROJECT.md` and `MEMORY.md`.
If `PROJECT.md` does not exist, run [Bootstrap](#bootstrap) before anything else.

## Bootstrap

The first interaction sets up the project for this agreement. The structure
depends on the product: a website, a mobile app, a desktop app, a game or a
hardware+software system do not share languages, layout or tooling.

1. **Detect.** If the repo has code, infer product type, languages, frameworks,
   package managers, test/lint tools, existing CI and i18n from the files.
2. **Ask only what changes the structure** and cannot be detected: product
   type, target platforms, product languages (i18n), deployment target,
   whether there is a production environment with data to preserve.
3. **Propose** in one message:
   - the responsibility map (e.g. `frontend`, `api`, `db`, `firmware`,
     `game-logic`) with each folder, stack and verification command;
   - the single sources of truth to centralise (theme/design tokens,
     configuration, i18n catalogues, shared constants, API contracts);
   - the CI pipeline (lint, type-check, tests per responsibility), by default
     GitHub Actions in `.github/workflows/`, or whatever fits the host
     (GitLab CI, pre-commit hooks, platform-specific build checks);
   - `CLAUDE.md` if missing, and `MEMORY.md` added to `.gitignore`.
4. **Generate only after confirmation**: `PROJECT.md` (template below), CI,
   `CLAUDE.md` if missing, `.gitignore` entry, and a first `MEMORY.md`.

`PROJECT.md` template:

```markdown
# PROJECT.md
## Product           <!-- what it is, for whom, environment (production or not) -->
## Responsibilities  <!-- table: scope | folder | stack | verify command -->
## Sources of truth  <!-- where tokens, config, i18n, contracts live -->
## Conventions       <!-- only rules that differ per project -->
## Data policy       <!-- migrations or init-only schema, retention -->
```

## Workflow

1. **Understand the product goal.** The requester often does not know exactly
   what they want yet, and a solution that only fits today's example will need
   rework tomorrow. Ask when the answer changes the design, the scope or the
   product behaviour. Do not ask trivial questions: decide those with the
   project conventions and state the assumption.
2. **Slice.** When a request contains several features, split them, agree on
   the order, handle one at a time, and record the rest in `MEMORY.md` → Future.
3. **Read and search before writing.**
   - Trace the real flow the change touches: callers, tests, config, exports,
     i18n, other services consuming the same data.
   - Go through the [reuse order](#reuse-order).
   - Use the internet as an instant second opinion: state of the art, how a
     specific problem is usually solved, known pitfalls, and well-established
     libraries or components (shadcn/ui, Base UI, Radix, MUI…). Proven
     components usually bring theming, responsive sizing and accessibility
     that a hand-made one would have to rebuild.
4. **Compare approaches** for anything non-trivial. Do not accept the first
   idea; weigh size of the change, readability, runtime cost and fit with the
   product goal. Give a recommendation with its reason.
5. **Implement** following the principles below.
6. **Verify** with the commands in `PROJECT.md`.
7. **Report** (see [Reporting](#reporting)).
8. **Update memory** (see [Memory](#memory)) and propose promotions.

## Principles

### Reuse order

Before writing code, check these in order and stop at the first that solves it:

1. Does this need to exist at all? Skip unrequested features.
2. Is it already in the repo? Reuse or extend it.
3. Does the standard library do it?
4. Is there a native platform feature? (HTML element, browser/OS API, SQL…)
5. Does an installed dependency do it?
6. Is there a well-established library or component for it?
7. Can it be one line?
8. Only then: the minimum new code that fully works.

### Never cut

Less code comes from needing less, not from compressing it. Never remove or
skip validation at trust boundaries, error handling where data can be lost,
security, or accessibility. Keep the solution minimal, but finish every part
the task needs.

### Simplicity

- Prefer what is easiest to understand, explain, locate and maintain. If
  50–100 lines solve it, 1000 lines is the wrong answer.
- Abstract only when the design needs it (a real second use, a stable
  boundary). No speculative options, layers or flags.
- Every line is a maintenance cost; removing code is an improvement.

### Single source of truth

Centralise what defines the product so one change propagates everywhere:
design tokens and theme, configuration, i18n strings, shared constants, API
contracts. Never hard-code a value that already has a central home.

### No legacy

No exceptions. A change replaces the old design; delete what it makes
obsolete (code, files, keys, config) in the same change. Code whose purpose is
unknown or obsolete must not stay: verify nothing uses it, remove it, and
mention it in the report. No compatibility shims, deprecated paths or
"just in case" code.

### Performance

- For non-trivial logic, think about algorithmic complexity before coding
  (e.g. an O(n³) nested loop where sorting/indexing gives O(n log n)).
- Avoid chatty communication: N+1 queries, one request per item, downloading
  whole files to use a fragment, polling where an event exists.
- Pick the simplest option whose complexity is acceptable.

### Bugs and tests

- For a bug, find every caller of the code involved and fix the root cause in
  the shared place.
- Logic with branches, loops, parsing, time windows, money or security gets
  one small test (or an assert-based self-check where no test setup exists).
- When a deliberate limit is accepted, state it next to the code:
  `Limit: <what it does not handle>; <when it would need more>`.

### Comments

Explain what the code solves, what goes in and out, and the approach. No
change-log comments ("fixed X", "now works"); history belongs in git. Match the
comment density and language of the surrounding code.

### Languages

Follow the product's i18n setup: every user-facing string goes through it, in
every supported language.

## Reporting

End every reply with one or two lines on what was not checked and any risk.

When an implementation or fix is finished, list it as commit lines:

```text
type(scope): description
```

- **type:** `feat`, `fix`, `refactor`, `docs`, `chore`, `test`, `perf`.
- **scope:** a responsibility from `PROJECT.md`; combine only when
  inseparable (`frontend&api`).
- **description:** in the user's language, very short: "adds X", "fixes Y".
- Split unrelated changes into separate commits, ordered so each one works
  on top of the previous (e.g. db → backend → frontend).

## Reviews and audits

Rank findings by priority. For each: what the code does, what goes wrong
(concrete input or state → wrong result), how to fix it, and what happens if it
is skipped. Look for bugs, security, behaviour under load, slow paths, missing
tests, legacy and code that can be deleted.

## Memory

`MEMORY.md` is the personal, local thinking space between sessions. It records
patterns, not transcripts: "date — what was tried → what the user corrected →
what was done → did it work". Example: "user disliked one bordered card per
field → `<dl>` grid with collapsible technical section; accepted."

| Section | Content | Limit |
| --- | --- | --- |
| Past | Condensed lessons from earlier sessions | 60 lines, ≤ 4 lines per entry |
| Present | Current session: goal, decisions, corrections | 20 lines |
| Future | Pending slices, open questions, risks | 20 lines, 1–2 lines per item |

- At the start of a session, condense the previous Present into Past.
- When a section exceeds its limit, merge similar entries; drop entries that
  no longer change behaviour or refer to things that no longer exist.
- Verify file names, functions and paths from memory before relying on them.
- **Promotion:** a lesson that would help anyone on any project goes to
  `AGENTS.md`; one that applies to this project goes to `PROJECT.md`; only
  personal preferences stay in `MEMORY.md`. Propose promotions to the user.
