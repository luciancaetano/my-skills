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
/plugin install sdd-codify@luciancaetano-skills
```

## Updating

New commits to this repo don't reach an installed plugin automatically — refresh the marketplace, then update the plugin:

```
/plugin marketplace update luciancaetano-skills
/plugin update sdd-codify@luciancaetano-skills
```

Check what's installed and whether an update is available:

```
/plugin
```

`sdd-codify` also has its own in-project sync, separate from the plugin version above — once it has migrated a project, re-run it there with `/sdd-codify --update` to apply template changes (see [`references/changelog.md`](./.claude/skills/sdd-codify/references/changelog.md)) without touching the project's own customizations.
