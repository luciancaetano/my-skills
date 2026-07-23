# Skills → Rules: Classification and Merge Guide

Skills and rules answer different questions. A **rule** applies whenever you touch its domain — it must be in (or one hop from) always-loaded context. A **skill** is an on-demand procedure — it loads only when its task comes up. Most bloated setups have skills that are really rules, and the model under-triggers them; the fix is moving that content to where it's always seen.

## Classify each skill

For every skill in the target project (`.claude/skills/`, `.agents/skills/`, plugin skills the project owns), ask:

**1. Is it an always-on constraint in disguise?** — Its content says how code in this repo must *always* be written (best practices, security posture, style mandates), and there's no meaningful "invocation": you'd want it active for any change in its domain.
→ **Merge into rules.** Small skills (< ~1 page of normative content) fold into the matching `rules/*.md` file. Large or multiple overlapping ones become an indexed knowledge base (below).

**2. Is it a genuine on-demand procedure?** — It performs or scaffolds a discrete task (writing commit messages, generating a component, running a release, driving a spec-change workflow), with clear start/finish.
→ **Keep as a skill.** List it in `rules/workflow.md`'s Skills section with a one-line "activate when …" so activation stops depending on the model remembering the catalog.

**3. Is it dead weight?** — Auto-generated filler, duplicated by another skill, or contradicted by the repo.
→ **Propose deprecation** in the step-2 plan. Never delete without explicit approval.

A skill can split: a testing skill may contain always-on conventions (→ `rules/testing.md`) *and* a scaffolding procedure (→ stays a skill, slimmed).

## Merging overlapping skills into a knowledge base

Whenever ≥2 skills cover the same domain from different angles, or one skill is too big to load whole, merge them into one `.claude/{kb-name}/` knowledge base indexed by development context, and supersede the old skills.

Mechanics:

1. **Deduplicate** their content into per-topic files (`rules/` for normative one-pagers, `references/` for deep dives), keeping the project's technology and versions.
2. **Annotate divergence**: wherever generic advice conflicts with a project rule, keep the generic line but annotate it ("here: {the project's way} — see `.claude/rules/{file}.md`"), and list these divergences in the INDEX precedence section. Project rules always win.
3. **Build `INDEX.md`** per the template in `target-architecture.md`: precedence, context → file tables, task → files map, one-line quick reference per file, validation checkpoints with the project's real commands.
4. **Point the root `CLAUDE.md`** at the index with the "consult the index, load only matching files" instruction.
5. **Supersede the source skills**: with approval, delete them; without, prepend a deprecation note to each SKILL.md ("Superseded by `.claude/{kb-name}/` — do not follow this file") and note the supersession in `rules/workflow.md`.

## What never merges

- Skills owned by plugins/marketplace installs the project doesn't control — reference them in workflow.md instead.
- Meta-skills (skill creators, config helpers) — they're about the harness, not the codebase.
- Anything whose content would change behavior if flattened (e.g., a skill that must ask the user questions interactively). Rules can't pause for input; procedures that do stay skills.
