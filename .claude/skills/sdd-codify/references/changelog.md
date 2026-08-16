# Skill Changelog

Dated log of changes to this skill's **output shape** — the root `CLAUDE.md`
template, the rule-file templates, and the workflow template. This is what
`--update` mode reads: it walks the entries top-down, checks each one's
"Detect" against the target project's already-migrated files, and applies
"Update action" for every entry that hasn't landed yet.

Newest entries first. Never rewrite a past entry — if a change is later
revised, add a new entry that supersedes it and say so.

## 2026-08-16 — `assumption-reviewer` + `spec-reviewer` merged into `spec-reviewer-ultra`

**Applies to:** `references/agent-templates.md` (was two agent blocks, now
one), `references/workflow-template.md` (step 1's assumption-gate and
spec-review-gate bullets), `SKILL.md` (agent list, target architecture tree).
The two agents ran back-to-back on the same artifact (assumption-reviewer
against the request before drafting, spec-reviewer against the draft before
approval) with no implementation step between them — one pass covers both
without losing coverage.
**Change:** the four-agent set is now three. `spec-reviewer-ultra` reads the
request AND the spec draft, runs both agents' full responsibility lists
(hidden assumptions, ambiguity, missing requirements, conflicts, hidden
scope, structure, EARS, coverage, decisions, boundaries, implementation
leakage) in a single 500-word-capped report, gated once before developer
approval. `workflow.md` step 1 loses its separate "Assumption gate" bullet;
the "Spec-review gate" bullet now names `spec-reviewer-ultra` and also feeds
it the original request.
**Detect:** target has `.claude/agents/assumption-reviewer.md` and/or
`.claude/agents/spec-reviewer.md`.
**Update action:** write `.claude/agents/spec-reviewer-ultra.md` from the
current template (reconcile any project-specific wording from the old
files), delete `assumption-reviewer.md` and `spec-reviewer.md`, collapse
`workflow.md` step 1's two gate bullets into the single spec-review-gate
bullet pointing at `spec-reviewer-ultra`.

## 2026-08-14c — Explicit tests-exist gate before implementation

**Applies to:** `references/agent-templates.md` (coder's Rules section),
`references/workflow-template.md` (step 3, "Implementation last"). Found via
a real migration where the coder agent sometimes started implementing (or
wrote its own tests) when the test-writing step had been skipped or left
gaps, because "tests exist before you start" was stated as a fact rather than
something to check.
**Change:** both the workflow's step 3 and the coder's Rules section now
require an explicit pre-flight check — map every acceptance criterion to an
existing failing test, run the suite to confirm red — before any
implementation code is written. Missing or already-green coverage means
step 2 was skipped: stop and report the gap, don't write tests inside the
implementation step and don't guess at intended behavior from the spec
alone.
**Detect:** target's coder agent file has "Tests = gate" but no "Pre-flight"
rule above it, or `workflow.md` step 3 has no "Tests-exist gate" bullet.
**Update action:** insert the pre-flight rule into the installed coder
agent's Rules section and the tests-exist gate bullet into step 3 of
`rules/workflow.md`, matching the current templates.

## 2026-08-14b — Hard word caps, no raw paste, coder silence, session-per-spec

**Applies to:** `references/agent-templates.md` (Output section of all four
agents), `references/workflow-template.md` (top-of-file context-hygiene note,
coder's opening, end of the compliance-gate step). Tightens the 2026-08-14
pass further: caps were a soft "terse" instruction with no number, so reports
still drifted long.
**Change:** each Output section now states a numeric hard cap (400 words for
assumption-reviewer and spec-reviewer, 500 for compliance-reviewer, 300 for
the coder's report — implementation code itself excluded from the coder's
cap) and forbids pasting raw code/diff/log, requiring file:line citations
instead; multi-item sections require one line per item, not a paragraph.
`workflow.md` gained an explicit top-of-file note stating the fork-dispatch
rule once for all four gates (the gate lines already said it individually).
The coder template gained a "no narration while working" line. The
compliance-gate step gained a "close the session per spec" note: once the
verdict lands and developer review is done, end the session rather than
starting the next feature in the same conversation.
**Detect:** target's agent files lack a numbered word cap in their Output
section, or `workflow.md`'s compliance-gate step has no session-close note.
**Update action:** add the numeric cap + no-raw-paste + one-line-per-item
wording to each installed agent's Output section; add the top-of-file
context-hygiene note and the session-close note to `rules/workflow.md`; add
the no-narration line to the installed coder agent file.

## 2026-08-14 — Fork dispatch + capped output for phase agents

**Applies to:** `references/agent-templates.md` (new "Invocation contract"
section + Output section of all four agents), `references/workflow-template.md`
(the four conditional gate/dispatch lines in steps 1/3/4). Found via a real
migration where the multiagent flow's context/token cost had grown large:
agents were spawned as fresh general-purpose subagents (re-paying full
context every dispatch) and wrote long prose reports back into the parent
conversation.
**Change:** every gate/dispatch line now says to invoke via the Agent tool
with `subagent_type: "fork"` instead of a fresh spawn — a fork shares the
parent's context and prompt cache instead of re-explaining it. Each agent's
Output section gained a terseness line (bullet points not prose, omit empty
sections instead of writing "None," no restating the input) and a new
"Invocation contract" section instructs the dispatcher to repeat the output
cap in the dispatch prompt itself, since a forked agent inherits chatty
context and can drift verbose.
**Detect:** target has `.claude/agents/*.md` from this skill, but
`workflow.md`'s gate lines don't mention `subagent_type: "fork"`, or the
agent files' Output sections lack a terseness line.
**Update action:** patch each present gate line in `rules/workflow.md` to
name fork dispatch, and append the terseness line to each installed agent's
Output section, matching the current templates.

## 2026-08-08b — Wire agent gates into workflow.md's numbered flow

**Applies to:** `references/workflow-template.md`. Found via a real
migration that installed spec-reviewer + coder agents (Step 5.5) but
`workflow.md`'s numbered spec→tests→implementation→validation steps never
said when to invoke them — only a loose "Skills" bullet list existed, so
the agents sat unused.
**Change:** workflow-template.md's steps 1/3/4 gained conditional
`{...agent name}` gate lines (assumption gate before drafting, spec-review
gate before presenting for approval, implementer dispatch at step 3,
compliance gate before calling the change done) — each survives only if
that agent was actually installed. "Skills" section renamed "Skills &
Agents" and gained an agent bullet format naming the exact step each plugs
into. SKILL.md Step 6 checklist tightened: an installed agent with no
matching gate line in the numbered flow now fails verification.
**Detect:** target has `.claude/agents/*.md` from this skill but
`rules/workflow.md`'s numbered steps contain no reference to any of those
agent names (only a Skills list, if that).
**Update action:** propose inserting the matching gate line(s) from the
current `references/workflow-template.md` at the correct step for each
agent actually present in `.claude/agents/`; rename "## Skills" to "## Skills
& Agents" and add the agent bullets.

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
