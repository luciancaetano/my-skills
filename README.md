# my-skills

Claude Code plugins by [Lucian Santos Caetano](https://github.com/luciancaetano). Licensed under [MIT](./LICENSE).

## Plugins

### sdd-codify

Migrates a project's `CLAUDE.md` (or `AGENTS.md`) into a slim multi-file rules architecture: a short root file with critical rules, `.claude/rules/*.md` split by concern, a spec-driven TDD workflow, and project skills merged into rules or an indexed knowledge base. Technology-agnostic — works on any language or stack.

See [`.claude/skills/sdd-codify/SKILL.md`](./.claude/skills/sdd-codify/SKILL.md) for details.

## Install

Add this repository as a marketplace, then install the plugin:

```
/plugin marketplace add luciancaetano/my-skills
/plugin install sdd-codify@luciancaetano-marketplace
```
