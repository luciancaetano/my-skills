# Skill Changelog

Dated log of changes to this skill's **output shape** — the root `CLAUDE.md`
template, the rule-file templates, and the workflow template. This is what
`--update` mode reads: it walks the entries top-down, checks each one's
"Detect" against the target project's already-migrated files, and applies
"Update action" for every entry that hasn't landed yet.

Newest entries first. Never rewrite a past entry — if a change is later
revised, add a new entry that supersedes it and say so.

## 2026-08-08 — Phase-gated subagent templates (new Step 5.5)

**Applies to:** new `references/agent-templates.md`, new optional output
`.claude/agents/*.md`. Found via a real Laravel project (multi-tenant API)
that runs spec-driven TDD through four dedicated subagents —
`assumption-reviewer`, `spec-reviewer`, an implementer (`coder`), and
`compliance-reviewer` — one per workflow phase, each forbidden from doing
the next phase's job.
**Change:** new Step 5.5 installs these subagents when the target project's
scale/existing conventions warrant it (see "Scaling down" in the reference
for solo-project vs. team-scale judgment); `target architecture (summary)`
tree gained the `.claude/agents/` block; Step 6 checklist gained a
subagent-references-resolve check; Step U1 now also diffs `.claude/agents/`
against the template.
**Detect:** target has `.claude/rules/workflow.md` (prior migration) but no
`.claude/agents/{assumption-reviewer,spec-reviewer,compliance-reviewer}.md`,
or one exists but references stale rule/gate filenames.
**Update action:** propose Step 5.5 as a delta — write the missing agent
files from `references/agent-templates.md`, or patch stale references in
existing ones, scoped per "Scaling down."

## 2026-07-23 — Tool-managed block handling (Step 0 / Step 1 / Step U1)

**Applies to:** the skill's own process. Found via a dry-run migration
against a real Laravel project whose `CLAUDE.md` embedded a 300+ line
`<laravel-boost-guidelines>` block (and a separate `<!-- rtk-instructions
-->` block) — content two other tools auto-sync into the file, not the
project's own statements.
**Change:** Step 0 now identifies tool-managed blocks (marker-wrapped,
auto-regenerated content) up front; Step 1's classification table gained a
"Tool-managed" bucket (left in place, untouched, not counted against the
root file's size budget, verbatim duplication across files is expected
tool behavior not something to dedupe); Step U1 now excludes tool-managed
blocks from its "unclassified hand-added content" scan so `--update` doesn't
mistake a tool's own resync for project content.
**Detect:** n/a — process change, doesn't touch the output templates.
**Update action:** n/a; always apply the current Step 0/1 rules, including
during `--update` runs.

## 2026-07-23 — `decisions.md` gains an "Amending a decision" section

**Applies to:** `rules/decisions.md`, inserted directly under the intro
paragraph, before the first dated entry.
**Change:** fixed, skill-supplied section explaining how to amend a decision
(add a new entry stating what/why/what it supersedes/what it breaks
downstream) instead of editing history in place — the same discipline
`decisions.md` already asked for informally, now spelled out so it survives
`--update` re-syncs.
**Detect:** `rules/decisions.md` lacks a `## Amending a decision` heading.
**Update action:** insert the section verbatim (see `target-architecture.md`)
immediately after the intro paragraph and before the first dated entry.

## 2026-07-23 — Gate check + Complexity Tracking before implementation

**Applies to:** `rules/workflow.md`, step 3 ("Implementation last").
**Change:** step 3 gained a "Gate check" sub-bullet: before writing
implementation code, confirm the change doesn't add an unwarranted
abstraction layer or scope beyond the spec; if a gate genuinely must be
crossed, that gets a **Complexity Tracking** note in the spec instead of a
silent violation.
**Detect:** `rules/workflow.md` step 3 lacks a "Gate check" bullet.
**Update action:** insert the Gate check bullet under step 3, matching
`workflow-template.md`.

## 2026-07-23 — EARS acceptance criteria in the spec step

**Applies to:** `rules/workflow.md`, step 1 ("Spec first"), and step 2
("Tests second").
**Change:** step 1 gained an EARS-notation primer for acceptance criteria
(`SHALL` keyword only; ubiquitous / event-driven / state-driven / unwanted-
behavior forms), and step 2 now notes that each EARS criterion becomes one or
more test cases.
**Detect:** `rules/workflow.md` step 1 lacks an "Acceptance criteria in EARS
syntax" bullet.
**Update action:** insert the bullet into step 1 (after the "Mark unresolved
ambiguity inline" bullet if present, else after "Surface assumptions first"),
and update step 2's wording, matching `workflow-template.md`.

## 2026-07-23 — `[NEEDS CLARIFICATION]` markers in the spec step

**Applies to:** `rules/workflow.md`, step 1 ("Spec first").
**Change:** step 1 gained a "Mark unresolved ambiguity inline" bullet:
ambiguities get a literal `[NEEDS CLARIFICATION: specific question]` marker
at the exact point in the spec text they affect, and the spec isn't
approvable while any marker remains — each must resolve to a requirement or
an explicit Open Questions entry.
**Detect:** `rules/workflow.md` step 1 lacks a "Mark unresolved ambiguity
inline" bullet.
**Update action:** insert the bullet directly after "Surface assumptions
first", matching `workflow-template.md`.

## 2026-07-23 — "Surface assumptions first" bullet in the spec step

**Applies to:** `rules/workflow.md`, step 1 ("Spec first").
**Change:** step 1 gained a "Surface assumptions first" bullet — list what's
being assumed and let the developer correct it before the spec is drafted.
Split into its own entry because it predates, and outlived, the now-removed
`spec-driven-development` pointer below: it's part of the permanently inlined
method, not something the pointer-removal entry should be responsible for
re-adding.
**Detect:** `rules/workflow.md` step 1 lacks a "Surface assumptions first"
bullet.
**Update action:** insert the bullet directly after the "How to present it"
bullet, matching `workflow-template.md`.

## 2026-07-23 — `spec-driven-development` dependency removed; spec method inlined

**Applies to:** `rules/workflow.md` step 1, and the skill's own Step 0 / Step
4 / Step 6.
**Change:** the pointer to an external `spec-driven-development` skill
(added earlier the same day, see the superseded entry below) is removed —
that skill won't be present going forward. The method it stood in for is
written directly into `workflow-template.md` instead of referenced: the
"Surface assumptions first" entry above, plus NEEDS CLARIFICATION markers,
EARS criteria, and the gate check further above. **Supersedes** "Spec step
points to `spec-driven-development`" below — apply the entry above instead
of this one for the assumptions bullet — and narrows "Step 0
skill-availability check" below from a fixed list of expected skills to a
generic rule: check whatever a rule file happens to name, if anything.
**Detect:** `rules/workflow.md` step 1 contains a blockquote referencing
`spec-driven-development` by name.
**Update action:** remove the blockquote. If any of the four entries above
(assumptions bullet, NEEDS CLARIFICATION, EARS, gate check) haven't landed
yet either, apply them in the same pass — the inlined method must be
complete, not a regression from what the pointer used to promise.

## 2026-07-23 — `--update` mode added

**Applies to:** the skill's own process (new mode, no output-file change by
itself).
**Change:** the skill can now re-sync a project it previously migrated
against the current version of its templates, instead of only doing one-shot
migrations from a monolithic source.
**Detect:** n/a — this entry documents the mode's own introduction.
**Update action:** n/a.

## 2026-07-23 — Step 6 became a literal checklist

**Applies to:** the skill's own process (Step 6 — Verify and report).
**Change:** the verify step changed from prose bullets to a checkbox
checklist (coverage, no placeholders, no duplication, tech check, skill
references resolve, ≤~8 critical rules, boundaries defined).
**Detect:** n/a — process change, not an output-file change.
**Update action:** n/a; always use the current checklist when verifying,
including during `--update` runs.

## 2026-07-23 — Step 0 skill-availability check

**Applies to:** the skill's own process (Step 0 — Inventory) and any rule
file that names another skill.
**Change:** before pointing a migrated file at a canonical-source skill
(`spec-driven-development`, `planning-and-task-breakdown`,
`incremental-implementation`, `test-driven-development`,
`context-engineering`), confirm it's actually available in the current
environment's skill catalog. Unavailable skills get inlined instead of
referenced.
**Detect:** any migrated file contains a bare skill-name reference (e.g.
`` `spec-driven-development` ``, `` `planning-and-task-breakdown` ``) that
does not currently resolve in the skill catalog.
**Update action:** re-check availability against the *current* catalog (it
may have changed since the original migration, in either direction). If a
previously-inlined section could now be replaced by a pointer to a newly
available skill, propose that simplification in the update plan. If a
previously-referenced skill is no longer available, propose inlining its
content instead of leaving a dangling reference.

## 2026-07-23 — "Staying on the Rails" section in `workflow.md`

**Applies to:** `rules/workflow.md`, inserted before the `## Skills` section.
**Change:** added a fixed (skill-supplied, not project-sourced)
rationalizations table and red-flags list, so the installed spec-driven-TDD
flow resists eroding under deadline pressure instead of just being documented
once.
**Detect:** `rules/workflow.md` lacks a `## Staying on the Rails` heading.
**Update action:** append the section verbatim (see
`workflow-template.md`) directly before `## Skills`. If the project already
has its own rationalizations/red-flags content elsewhere in the file, merge
rows instead of duplicating and note the merge in the update report.

## 2026-07-23 — Optional `## Boundaries` section in `workflow.md`

**Applies to:** `rules/workflow.md`, between the numbered flow and the
sync-obligations section.
**Change:** added an optional Always/Ask-first/Never boundaries block,
shaped after `spec-driven-development`'s three-tier system, filled with the
target project's own boundaries.
**Detect:** `rules/workflow.md` lacks a `## Boundaries` heading, **and** the
project's CLAUDE.md, rule files, or repo conventions contain enough
Always/Ask-first/Never-shaped material to extract one (e.g. scattered "always
run X before committing" / "ask before changing Y" / "never do Z"
statements).
**Update action:** if material exists to extract, propose the consolidated
Boundaries section in the update plan citing where each line came from; if no
such material exists, leave it out (do not invent boundaries) and say so in
the report rather than silently skipping it.

## 2026-07-23 — Spec step points to `spec-driven-development`

**Applies to:** `rules/workflow.md`, step 1 ("Spec first") of the numbered
flow.
**Change:** step 1 gained a "Surface assumptions first" bullet and a
canonical-source pointer to `spec-driven-development` (the six-area spec,
assumption-surfacing, boundaries, success-criteria reframing), matching how
steps 2–3 already point at `planning-and-task-breakdown`. Step 1 previously
reimplemented a thinner version of the same method inline with no pointer.
**Detect:** `rules/workflow.md` step 1 lacks both the "Surface assumptions
first" bullet and the `> Follow \`spec-driven-development\`...` blockquote.
**Update action:** insert the "Surface assumptions first" bullet after the
existing "How to present it" bullet, and the canonical-source blockquote
after "Save on confirmation" — same placement as in `workflow-template.md`.
Skip the blockquote (inline the method summary instead) if
`spec-driven-development` fails the Step 0 availability check.
