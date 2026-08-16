# Workflow Template — Spec-Driven TDD

Template for the migrated project's `rules/workflow.md`. Fill every `{placeholder}` with values detected in step 0 (never leave placeholders in the written file). If the source project already documented workflow obligations (collection sync, docs regeneration, changelog), keep them as additional sections — reconcile, don't overwrite.

The corresponding root-CLAUDE.md critical rule should summarize this in two lines and point here.

---

```markdown
# Development Workflow — Spec-Driven TDD

> **Context hygiene.** Every subagent this workflow names (`{spec-reviewer-ultra
> agent name}`, `{implementer agent name}`, `{compliance-reviewer agent name}`)
> must be dispatched through a fork — the
> Agent tool with `subagent_type: "fork"` — never a fresh spawn. The fork reads
> the subagent's full report and relays back only what that agent's capped
> Output section asks for; the subagent's raw tool calls, exploration, and logs
> stay in the fork and never reach this conversation. See agent-templates.md's
> invocation contract for the exact dispatch prompt.

For every new feature AND every alteration/change to existing behavior, follow
this order strictly:

1. **Spec first.** Write a spec (behavior, inputs/outputs, edge cases — for
   alterations, describe the diff from current behavior) and present it to the
   developer for approval. Do not write any code yet.
   - **How to present it:** the spec goes in the conversation as a plain
     markdown message, and the turn ends there — wait for the developer to
     approve or request changes in their reply. Never follow the spec with an
     approval dialog (AskUserQuestion or similar): it overlays the message and
     the spec effectively isn't seen. Open design questions belong inside the
     spec text as bullet points with a recommended default, to be answered in
     the reply.
   - **Surface assumptions first.** Before drafting the spec, list what's being
     assumed (stack details, auth model, scope boundaries) and give the
     developer the chance to correct them — assumptions are the most dangerous
     form of misunderstanding, and the spec's job is to catch them before code
     exists.
   - **Mark unresolved ambiguity inline.** Anywhere the spec text depends on
     something not yet decided, write `[NEEDS CLARIFICATION: specific
     question]` at that exact point instead of guessing a plausible value
     (e.g. "Reset link expires. [NEEDS CLARIFICATION: after how long?]"). This
     is stronger than an upfront list alone — it's greppable and pinned to the
     clause it affects. **The spec is not ready for approval while any marker
     remains**: each one must become either a resolved requirement or an
     explicit Open Questions entry with a recommended default.
   - **Acceptance criteria in EARS syntax.** Write testable acceptance
     criteria using the keyword `SHALL` (never "should"/"must"/"will") in one
     of these forms:
     - Ubiquitous: `THE <system> SHALL <response>`
     - Event-driven: `WHEN <trigger>, THE <system> SHALL <response>`
     - State-driven: `WHILE <precondition>, THE <system> SHALL <response>`
     - Unwanted behavior: `IF <condition>, THEN THE <system> SHALL <response>`
     Reject vague terms ("appropriate", "quickly", "properly") — each
     criterion must be independently verifiable.
   {- **Spec-review gate.** Before presenting for developer approval, dispatch
     `{spec-reviewer-ultra agent name}` via the Agent tool with `subagent_type:
     "fork"` against the drafted spec (and the original request, for
     assumption/scope checks) and fold its findings in. — include only if
     that agent was installed.}
   - **Save on confirmation.** Once the developer confirms the spec (and only
     then — never before confirmation), save it verbatim as
     `docs/specs/YYYY-MM-DD-short-slug.md` so the project keeps a dated history
     of confirmed specs.
2. **Tests second.** Only after the spec is approved, write/update tests based
   on it using {test framework} — each EARS acceptance criterion becomes one
   or more test cases. New tests should fail, and any existing test whose
   expected behavior changed should be updated to reflect the new spec (no
   implementation change exists yet).
3. **Implementation last.** Only after the tests are written, implement the
   code to make them pass, following TDD.
   - **Tests-exist gate.** Before writing or dispatching any implementation
     code, confirm every acceptance criterion has a corresponding test and
     that the suite is currently red for the new behavior. This is what
     removes ambiguity for whoever implements next — code written against a
     failing test has one unambiguous target; code written without one
     invites guessing at what "done" means. Step 2 was skipped if any
     criterion has no test — go back, don't improvise tests inside this step.
   {- Dispatch implementation to `{implementer agent name}` via the Agent
     tool with `subagent_type: "fork"`, working against the approved spec
     and the failing tests only. — include only if that agent was installed;
     otherwise implement directly.}
   - **Gate check.** Before writing implementation code, confirm: the change
     doesn't introduce a new abstraction layer where using the underlying
     framework/library directly would do, and it doesn't add scope beyond
     what the spec asked for. If a gate genuinely needs to be crossed (a real
     abstraction is warranted, extra scope is unavoidable), don't skip it
     silently — add a short **Complexity Tracking** note to the spec (what
     gate, why it's justified) so the exception is visible to the next
     reader instead of discovered later.
4. **Validation.** Prove the change with the project's own tools before calling
   it done:
   - Affected tests: `{single-file/filter test command}`
   - Full suite when the change is cross-cutting: `{full suite command}`
   {- Lint/format: `{lint command}` — if the project has one}
   {- Other checkpoints detected in step 0: type checker, schema generation,
      arch tests}
   {- **Compliance gate.** Dispatch `{compliance-reviewer agent name}` via
      the Agent tool with `subagent_type: "fork"` against the approved spec
      once implementation and tests pass, before calling the change done. —
      include only if that agent was installed.}
   - **Close the session per spec.** Once the verdict is PASS or PASS WITH
     MINOR GAPS and the developer's own review is done, end the session
     instead of starting the next feature in the same conversation. The
     saved spec (`docs/specs/`) and the diff are the durable record — the
     path taken to reach them (assumption lists, spec drafts, review relays)
     has no reuse value and only costs context: staying in one long
     conversation across features forces repeated compaction. Start a fresh
     session for the next spec.

{## Boundaries — keep if the source project had (or the migration extracted)
an Always/Ask-first/Never split for this kind of change; otherwise omit. Shape:
**Always do** (run tests before commits, follow conventions), **Ask first**
(schema changes, new dependencies, CI config), **Never do** (commit secrets,
edit vendor directories, remove failing tests without approval) — filled with
the target project's own boundaries, not these examples verbatim. Boundaries
are constitution-like: point to `rules/decisions.md`'s amendment rule for how
to change one once it's set, rather than letting them get silently edited.}

{## Scope selection / clarification — keep if the source project had a
"ask before scaffolding when X is ambiguous" rule; otherwise omit.}

{## Sync obligations — obligations carried over from the source CLAUDE.md,
e.g., "when creating or updating a route, update the matching {API client
collection} file alongside it".}

## Staying on the Rails

This flow erodes under deadline pressure unless the excuses for skipping it
are named up front.

| Rationalization | Reality |
|---|---|
| "This is too small for a spec" | Small changes get a two-line spec, not zero. |
| "I'll write the spec after coding" | That's documentation, not a spec — its value is forcing clarity *before* code. |
| "Tests can come after, I know this works" | Tests-after is how the failing-test gate silently stops meaning anything. |
| "It's obvious what to build" | Obvious to the person who already decided; the spec is what lets the developer disagree before cost is sunk. |

Red flags — stop and go back a step if you notice:
- Writing code with no spec the developer has seen.
- Tests added or changed *after* the implementation they're supposed to gate.
- Skipping validation because "it's a small change."

## Skills & Agents

{One bullet per kept skill: **name** — activate when {trigger}. Include
deprecation notes for superseded skills ("**old-name** — superseded by
`.claude/{kb-name}/`, do not activate").}

{One bullet per installed phase agent, naming the exact gate step above it
plugs into: **{agent-name}** — {phase} gate, invoked at step {N}. Omit this
list entirely if no `.claude/agents/*.md` were installed (see Step 5.5 of
the skill).}
```

---

## Adaptation notes

- **Test commands must be the canonical ones.** If tests run through a wrapper (container, `make`, a script), the template's commands use the wrapper and `testing.md` explains why — a workflow that names a command developers must not run bare will be followed literally.
- **"Spec" scales with the change.** The flow applies to behavior changes; make clear (as the reference does via "new feature AND every alteration") that refactors with no behavior change and pure doc/config edits don't need a spec. Don't add this as an escape hatch broader than that.
- **Validation is step 4, not an afterthought.** The user-visible promise of this workflow is that nothing is reported done without the test tooling proving it. If the project has no tests at all, the workflow still installs — step 2 then *establishes* the test harness for the feature at hand, and the migration report flags the missing harness.
- **"Staying on the Rails" is fixed content, not a placeholder.** Unlike the other sections, this table isn't sourced from the target project — it's the skill's own guardrail against the workflow decaying after installation, so copy it as-is. If the source project already had its own rationalizations/red-flags list, merge rather than duplicate rows.
- **Agent gates are conditional, not automatic.** Each `{...agent name}` bracket above only survives into the written file if Step 5.5 actually installed that agent — a fresh migration that scopes down to two agents (spec-reviewer-ultra + implementer, the common default) must delete the compliance-gate line, not leave it pointing at an agent that doesn't exist. Whichever subset gets installed, `workflow.md` is the single place naming *when* each one runs — an agent file existing under `.claude/agents/` with no corresponding gate line in the numbered flow is a half-finished install and must be caught by Step 6.
- **The spec method (NEEDS CLARIFICATION, EARS, gate check) is fixed content too, self-contained.** This skill does not depend on any other skill being installed in the target environment — every mechanic the workflow needs (assumption-surfacing, ambiguity markers, EARS acceptance criteria, the gate check) is written directly into the template. Don't reintroduce a pointer to an external spec-method skill; if a future project genuinely has one already installed and prefers it, that's a reconciliation call for step 4, not a default.
