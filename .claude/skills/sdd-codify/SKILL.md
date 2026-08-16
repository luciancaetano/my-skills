---
name: sdd-codify
description: "Migrate any project's CLAUDE.md (or AGENTS.md) into a slim multi-file rules architecture — a short root CLAUDE.md with critical rules, `.claude/rules/*.md` split by concern, a spec-driven TDD workflow (spec → approval → failing tests → implementation → validation), phase-gated subagents (spec-reviewer-ultra, implementer, compliance-reviewer) with templates in references/agent-templates.md, and project skills merged into rules or an indexed knowledge base. Technology-agnostic — works on any language or stack: it reorganizes the target project's own content and installs the flow, never bringing technology, application architecture, or folder conventions from any other project. Use this skill whenever the user wants to reorganize, restructure, split, refactor, 'clean up', or 'apply this architecture to' a CLAUDE.md; says their CLAUDE.md is too big, monolithic, or messy; wants skills consolidated into rules; or wants a spec/approval/TDD workflow installed in another project — even if they don't say the word 'migrate'. Also supports a `--update` mode that re-syncs a project this skill already migrated against the current version of its templates (see `references/changelog.md`) — use it for 'update the CLAUDE.md this skill generated' or 're-sync my rules with the latest version of this skill'."
---

# sdd-codify — CLAUDE.md Migration

Restructure a project's agent instructions (CLAUDE.md, AGENTS.md, or scattered docs) into a proven multi-file architecture: a slim root file that is always in context, per-concern rule files loaded by reference, a dated decision log, a spec-driven TDD workflow, and skills demoted or promoted to where they actually belong.

## Modes

- **Default — fresh migration.** The target project has no `.claude/rules/` produced by this skill yet (a monolithic CLAUDE.md/AGENTS.md, or scattered docs). Run Steps 0–6 below.
- **`--update` — re-sync a prior migration.** The target project's `CLAUDE.md` + `.claude/rules/` already match this skill's shape (a previous run of this skill produced them). Triggered by `/sdd-codify --update`, or by requests like "update the CLAUDE.md this skill generated," "re-sync my rules with the latest version of sdd-codify," or "this skill changed, bring my project's rules up to date." Run the **Update Process** below instead of Steps 0–6. If `--update` is requested but the target doesn't have this skill's shape, say so and offer the default mode instead — don't silently switch modes.

## Update Process (`--update`)

Same two non-negotiable principles apply, plus: **only patch what changed.** Update mode never rewrites a file wholesale — it applies targeted deltas from `references/changelog.md` and leaves everything else, including the project's own custom sections, untouched.

### Step U0 — Detect prior migration (read-only)

Confirm the target has this skill's shape: root `CLAUDE.md` with a `## Critical Rules` section and a `## Rules` index, plus `.claude/rules/*.md`. If the shape is missing or looks hand-rolled rather than skill-produced, stop and tell the user — this isn't what `--update` is for.

### Step U1 — Diff against the changelog (read-only)

Read `references/changelog.md` top to bottom. For each entry, run its **Detect** check against the target's current files. Collect the entries that haven't landed yet — those are the deltas to apply. While reading, also check any skill named by name in the target's rule files against the *current* environment's catalog: availability may have changed since the original migration in either direction, in which case propose inlining a now-dangling reference, or simplifying now-inlined content back to a pointer if a matching skill has since appeared.

Also check `.claude/agents/` against `references/agent-templates.md`: if the project has some but not all three phase agents, or an agent's rule/gate references have drifted from the current `.claude/rules/*.md` filenames, that's a delta too — propose it in the plan same as any changelog entry, scoped by "Scaling down" in that reference.

Separately, check whether the project's own CLAUDE.md or rule files gained content since the last migration that was never classified (someone edited CLAUDE.md by hand instead of the rule files). Classify any such content per the Step 1 table so it isn't lost — except tool-managed blocks (Step 0's check), which are expected to change on their own and are never "new content to classify."

### Step U2 — Present the update plan and stop

Same discipline as Step 2: present the changelog entries that will be applied, the exact insertion point in each file, any newly classified hand-added content and its destination, and open questions with a recommended default. **End the turn there and wait for approval** — never follow it with an approval dialog.

### Step U3 — Apply deltas only

After approval, apply each entry's **Update action** verbatim at the file and location the changelog entry specifies. Do not touch unrelated content. Re-apply any newly classified hand-added content per Step 3's placement rules.

### Step U4 — Verify and report

Run the same Step 6 checklist, then report which changelog entries were applied (and which were skipped, and why — e.g. "Boundaries section skipped, no extractable material").

## Two non-negotiable principles

1. **Migrate the organization and the workflow, never the technology.** This skill is stack-agnostic. Everything technological in the migrated files (frameworks, test runner, commands, folder layout, application architecture) comes from the **target project's own CLAUDE.md and repo** — the skill only supplies the file structure and the development flow. Never introduce a framework, tool, application-architecture pattern, or folder convention from any other project, including the one this skill was modeled on. The templates in `references/` describe *shapes* to fill with the target project's content, never content to copy.

2. **Preserve, don't invent.** Every normative statement in the source CLAUDE.md must land in exactly one destination file, or be deliberately dropped with a justification in the migration report. Do not rewrite the meaning of rules while moving them; tightening wording is fine, changing behavior is not. If the source is silent about something the target architecture has a slot for (e.g., no decision log exists), create the file with what can be *extracted* from the source and the repo — never fabricate decisions or rules the team didn't make.

## Target architecture (summary)

Read `references/target-architecture.md` for the full templates before writing any file. The shape:

```
CLAUDE.md                      # slim: project one-liner, numbered Critical Rules, index of rules files
.claude/rules/
  architecture.md              # domain model, module boundaries, layering, response/error formats
  coding-style.md              # naming, patterns, language policy, "follow sibling files"
  testing.md                   # exact test commands (through wrappers if any), conventions, no backdoors
  workflow.md                  # spec-driven TDD flow + skills activation list
  decisions.md                 # dated decision log, newest first, supersede-never-rewrite
  <domain>.md                  # one file per systemic concern the project actually has (optional)
.claude/<kb-name>/             # indexed knowledge base, only if skills get merged (optional)
  INDEX.md                     # precedence + context tables + quick reference
.claude/agents/                # optional: phase-gated subagents for the spec-driven flow (see references/agent-templates.md)
  spec-reviewer-ultra.md       # reviews assumptions + spec before approval
  <coder-name>.md              # implements against approved spec + failing tests
  compliance-reviewer.md       # verifies implementation against approved spec after done
docs/specs/                    # dated confirmed specs (created by the workflow, dir may start empty)
```

## Process

Work through these steps in order. Steps 0–1 are read-only; nothing is written before the plan is approved in step 2.

### Step 0 — Inventory (read-only)

- Read the source CLAUDE.md / AGENTS.md in full, plus anything it imports or links (`@file` imports, referenced docs).
- List existing agent config: `.claude/rules/`, `.claude/skills/`, `.agents/skills/`, commands, hooks in settings files.
- Detect the project's real tooling from its own manifests, build files, container and CI configs — whatever its ecosystem uses: how tests actually run, whether commands must go through a wrapper (container, build-tool target, script), what lint/format tools exist. The migrated `testing.md` must name the project's *canonical* invocation, not a generic one.
- Note the project's language policy (what language docs/comments are written in) and keep it.
- Identify any **tool-managed blocks**: content wrapped in markers that name an external tool and get regenerated by it — e.g. `<laravel-boost-guidelines>...</laravel-boost-guidelines>`, `<!-- rtk-instructions v2 --> ... <!-- /rtk-instructions -->`, or similar auto-sync markers from any ecosystem. These are not the project's own statements; they're vendored boilerplate a CLI/package writes on its own schedule. Set them aside from Step 1 classification entirely — see the Tool-managed bucket there.
- If the target project already has a skill it uses for spec-writing or TDD flow, note it — step 4 reconciles with it rather than overwriting. Otherwise, the workflow this skill installs is self-contained (it does not depend on any other skill being present): if a rule file is ever about to name another skill by name, confirm it actually exists in the current environment's skill catalog first — a reference to a skill nobody can invoke is worse than no reference, since it reads as authoritative but silently resolves to nothing.

### Step 1 — Classify every statement

Walk the source content and tag each statement with its destination bucket:

| Bucket | Goes to | Test |
|--------|---------|------|
| Critical rule | root `CLAUDE.md` | Would violating it in any task be serious? Keep ≤ ~8, one line each, each pointing to the rule file with details. |
| Architecture | `rules/architecture.md` | Describes how the system is shaped (modules, layers, data boundaries, formats). |
| Style | `rules/coding-style.md` | Describes how code is written (naming, patterns, language). |
| Testing | `rules/testing.md` | Describes how to run or write tests. |
| Workflow | `rules/workflow.md` | Describes the order of development activities or when to use skills. |
| Decision | `rules/decisions.md` | A choice not derivable from the code ("we deliberately don't do X"). Date it if the date is recoverable (git blame/log); otherwise mark it `undated, pre-migration`. |
| Domain concern | `rules/<domain>.md` | A systemic concern with several rules of its own (API docs conventions, i18n, a compliance regime). Only create the file if it has real substance. |
| Stale/duplicated | dropped | Contradicted by the repo, or duplicated elsewhere — list in the report with justification. |
| Tool-managed | left in place, untouched | Wrapped in a tool's own auto-sync markers (identified in step 0). Never move, summarize, or fold this into a rule file — the owning tool will just rewrite it out of sync with wherever it got copied to. If it's duplicated verbatim across multiple files (e.g. the same block in both `CLAUDE.md` and `AGENTS.md`), that's the tool's normal sync behavior, not something to dedupe. Note its presence and scope in the migration report; don't count it against the root file's size budget. |

Then classify skills using `references/skills-to-rules.md`: which are always-on rules in disguise (merge into rules or a knowledge base), which are genuine on-demand procedures (keep, list in `workflow.md`), which overlap each other (merge into one indexed knowledge base).

### Step 2 — Present the migration plan and stop

Present, as plain text in the conversation: the proposed file tree, a mapping table (source section → destination file), the skills disposition (merge / keep / needs approval to deprecate), anything being dropped and why, and open questions as bullets with a recommended default. **End the turn there and wait for approval.** This mirrors the spec-first discipline the migration itself installs — do not write any file, and do not stack an approval dialog on top of the plan (it hides it).

### Step 3 — Write the files

After approval, write the root CLAUDE.md and rule files following the templates in `references/target-architecture.md`. Keep the root file short — its job is triage, not detail. Every rule file cross-links its siblings where they touch (e.g., style file points to architecture file for the pattern's rationale).

### Step 4 — Install the workflow

Write `rules/workflow.md` from `references/workflow-template.md`, filling the placeholders with the tooling detected in step 0 (test command, lint command, spec directory). The flow being installed is always: **spec (with assumption-surfacing, `[NEEDS CLARIFICATION]` markers, and EARS acceptance criteria) → developer approval → save spec to `docs/specs/` → failing tests → implementation (gated by a complexity check) → validation with the project's test tools**. This method is self-contained in the template — don't replace it with a pointer to an external spec-writing skill. If the source project already had a workflow, reconcile rather than overwrite: keep its extra obligations (e.g., "update the API collection when routes change") as additional workflow steps.

### Step 5 — Merge skills

Execute the disposition approved in step 2, following `references/skills-to-rules.md`. Never delete a skill without explicit approval — superseded skills get a deprecation note pointing at the rule/KB file that replaced them.

### Step 5.5 — Install subagents (when warranted)

The spec-driven workflow gets enforcement teeth from phase-gated subagents: assumption + spec review before approval, implementation, compliance review after. Decide scope during Step 0/2, not silently here — see "Scaling down" in `references/agent-templates.md` for when to propose all three vs. a subset vs. none (no prior multi-agent convention and no ask → propose two, mention the rest as extensions).

Write `.claude/agents/*.md` from `references/agent-templates.md`, filling placeholders with the target project's real rule filenames, gate commands, and vocabulary (e.g. name the implementer agent after the project's own convention, not forced to "coder"). If the project already has agents under different names covering these phases, reconcile — keep their names/wording, patch only genuine gaps (missing gate references, missing sections) instead of overwriting.

### Step 6 — Verify and report

Run this checklist before reporting the migration done:

- [ ] **Coverage**: every statement in the source CLAUDE.md is either present in a destination file or listed as dropped with a justification.
- [ ] **No placeholders**: grep the new files for leftover `{...}` template text.
- [ ] **No duplication**: each rule lives in exactly one file; the root file only summarizes and points.
- [ ] **Tech check**: grep the new files for technology names that don't appear in the target project's manifests — anything found is contamination from the reference architecture and must be removed.
- [ ] **Skill references resolve**: any skill named by name in the written files (a kept skill, a superseding KB) is confirmed available in the target environment's catalog — nothing points at a skill that doesn't exist there.
- [ ] **Critical rules ≤ ~8**, each a pointer, not a restatement.
- [ ] **Boundaries defined** in `workflow.md` (Always / Ask first / Never) if the source or repo gave enough material to extract them — otherwise noted as a TODO for the team, not silently skipped.
- [ ] **`decisions.md` has the Amending a decision section** verbatim.
- [ ] **`workflow.md`'s spec step has the full method**: assumption-surfacing, `[NEEDS CLARIFICATION]` markers, EARS acceptance criteria, and the pre-implementation gate check — none silently dropped while filling placeholders.
- [ ] **Subagents, if installed**: each references the project's real rule filenames and test/gate commands (no leftover `{placeholder}` or foreign tech), and `workflow.md`'s numbered flow (not just its Skills & Agents section) has a gate line naming each installed agent at the exact step it runs — an agent file with no matching gate line in the flow is a failed migration.

Produce a **migration report** in the conversation: file tree written, mapping table, skills disposition, dropped items with justification, and any TODOs left for the team (e.g., "decisions.md seeded with 2 extracted decisions; add future ones at the top").

## Maintaining this skill

Any change to the output shape — `target-architecture.md`, `workflow-template.md`, or `skills-to-rules.md` — must get a new dated entry at the top of `references/changelog.md` (Detect check + Update action), in the same turn as the template change. An undocumented template change is invisible to `--update` mode: projects migrated before it will never receive it.

## Output conventions

- Migrated files are written in the project's documented language policy; if none exists, keep the source CLAUDE.md's language.
- Never commit — leave the changes in the working tree for the developer to review.
- If the target repo has no CLAUDE.md at all, this skill does not apply (that's `/init` territory); it migrates existing instructions.

## Author

[Lucian Caetano](https://github.com/luciancaetano) — licensed under [MIT](../../../LICENSE).
