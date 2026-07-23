# Target Architecture — File Templates

Templates for every file the migration can produce. Placeholders are in `{braces}`. These templates define **shape only** — every piece of content that fills them comes from the target project's own CLAUDE.md and repo. Never carry a technology, application architecture, or folder convention into a migrated file from anywhere else.

## Root `CLAUDE.md`

The root file is always in context, so it must stay small: a one-paragraph identity, the numbered critical rules, and an index. Detail lives in the rule files.

```markdown
# Project: {Name}

{One paragraph: what the system is, its domain, its stack in one line, and any
global shape statement (e.g., "JSON API only — every response is JSON").}

## Critical Rules

1. **{Short imperative title}**: {one- or two-line rule}. {Pointer to the rule
   file holding the details, e.g., "Full flow in `.claude/rules/workflow.md`."}
2. ...

## Rules

Detailed rules live in `.claude/rules/` — follow all of them:

- `.claude/rules/architecture.md` — {one-line summary of contents}.
- `.claude/rules/coding-style.md` — {...}.
- `.claude/rules/testing.md` — {...}.
- `.claude/rules/workflow.md` — {...}.
- `.claude/rules/decisions.md` — {...}.
{- `.claude/rules/{domain}.md` — {...}, if any.}

{## Knowledge Base — only if step 5 produced one:
one paragraph pointing at `.claude/{kb-name}/INDEX.md`, instructing to consult
the index and load only the files matching the task, and stating that project
rules take precedence over the knowledge base.}
```

Guidance:

- **≤ ~8 critical rules.** A critical rule earns its slot when violating it in *any* task would be serious (data isolation, forbidden write paths, the spec-first workflow, language policy, how tests must be run). Everything else belongs in a rule file.
- Each critical rule is a summary with a pointer, not the full text — the full text lives in the rule file so it's stated exactly once.
- Keep any harness-specific mechanics the source had (imports, tool notes) if still valid.

## `rules/architecture.md`

Holds the target project's **own, pre-existing** architecture statements — never an architecture the migration proposes: its domain model (entities and their relationships, in prose or a short list), module/layer boundaries and who may call whom, data-isolation patterns, directory ↔ concept mapping conventions, and canonical response/error formats if the project is an API.

Structure by concern with `##` headings. For each pattern, state the rule **and the why**, so future readers don't "fix" a deliberate choice — a data-isolation rule shouldn't just say "use the explicit scope"; if the project chose explicit scoping over implicit global state on purpose, say so.

If the source project enforces a structural rule mechanically (an arch test, a lint rule), name the enforcing file and state the maintenance obligation ("new module ⇒ add it to X").

## `rules/coding-style.md`

Holds: naming conventions, the established file patterns to imitate ("follow sibling files before inventing a pattern"), language policy for anything written to disk, and one-line restatements of cross-cutting rules that live in other files (each with a link — restate, don't duplicate detail). End by noting which concerns are covered elsewhere (formatter config, framework guidelines) so nobody re-adds them here.

## `rules/testing.md`

Two sections:

- **Running tests** — the project's *canonical* invocations, discovered from its manifests in step 0: full suite, single file, single test filter. If commands must run through a wrapper (container, build-tool target, script), say so and say **why** bare invocations fail or mislead (e.g., services only reachable inside the container network). Include env kill-switches relevant to tests.
- **Writing tests** — framework conventions, which skills to activate for test work, and the two standing prohibitions the architecture carries: **no test-only backdoors in the app** (no debug endpoints, query flags, or "when testing" conditionals — test state comes from seeders/fixtures) and **no deleting tests without approval**. Link to workflow.md for the test-first ordering.

## `rules/workflow.md`

Written from `workflow-template.md` (see that file). Also hosts the **Skills** section: every kept skill with a one-line "when to activate" description, plus any sync obligations the source project had (API collection sync, docs regeneration).

## `rules/decisions.md`

A log of product/architecture decisions **not derivable from the code**. Rules:

```markdown
# Product & Architecture Decisions

Decisions not derivable from the code alone. Add new entries at the top with a
date; never rewrite history — supersede with a new entry instead.

## Amending a decision

A decision here is load-bearing — code and rules depend on it holding. Don't
edit or delete an existing entry to change direction; add a new entry above
it that:
1. States what's changing and **why** (the concrete trigger — an incident, a
   requirement that no longer holds, a tradeoff that stopped paying off).
2. Names what it supersedes, so the old entry stays as historical record
   instead of getting overwritten.
3. Notes what it breaks, if anything (a rule in another file that must be
   updated to match, a boundary in `workflow.md` that changes).

## {YYYY-MM} — {Decision title}

{What was decided, why, and what NOT to do because of it ("don't scaffold X").}

## Open — {Question title}

{Known-undecided items, so nobody resolves them unilaterally.}
```

During migration, seed it with decisions *extracted* from the source ("we deliberately have no refresh tokens", "feature X was considered and rejected"). Recover dates from git history where possible; otherwise mark `undated, pre-migration`. Never invent decisions.

The **Amending a decision** section is fixed content the skill supplies (not extracted from the source) — always include it, the same way `workflow.md`'s "Staying on the Rails" section is always included. It's what keeps `decisions.md` from becoming a place people quietly rewrite instead of a dated log.

## `rules/{domain}.md` (optional)

One file per systemic concern with several rules of its own — API documentation conventions, i18n, accessibility, a compliance regime. Create it only when the concern has real substance (roughly: 3+ rules that would otherwise bloat another file). Structure: how it's wired up in this project → known gaps (dated) → rules going forward.

## Knowledge base `.claude/{kb-name}/` (optional — produced by skill merges)

When overlapping best-practice skills merge (see `skills-to-rules.md`), they become an indexed knowledge base, **not** one giant file:

```markdown
# {Stack} Dev — Indexed Knowledge Base

{One line: what this is.} Load **only the files matching the task at hand**.

Origin: migrated from the {skill names} skills.

## Precedence
1. **Project rules win.** CLAUDE.md and .claude/rules/*.md override anything
   here. {List the concrete places this project diverges from generic guidance.}
2. **Consistency first.** Before applying any rule, check what sibling files
   already do — these are defaults for when no pattern exists, not overrides.
3. **Verify syntax.** Confirm exact APIs against current docs for the installed
   versions ({versions}).

## Index by Development Context
{Tables: "Context | Load", grouped by area (data layer, HTTP layer, async,
cross-cutting, testing). Each row maps a task context to the file(s) to load.}

## Typical Task → Files Map
{Bullets: "New endpoint: A + B + C".}

## Quick Reference
{One line per rule file with its 3–6 headline rules, so simple tasks never
need to open the full file.}

## Validation Checkpoints
{Table "Stage | Command | Expected" using the project's real commands.}
```

The KB files themselves keep the source skills' content, deduplicated, with project-divergent advice annotated ("here: {project's way}").

## `docs/specs/`

Created (empty or with a README line in the workflow file, not a separate doc) as the destination for confirmed specs: `docs/specs/YYYY-MM-DD-short-slug.md`, saved **only after** the developer confirms a spec. This directory is the project's spec history; the workflow file documents it.
