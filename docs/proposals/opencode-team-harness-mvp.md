# Proposal: opencode-team-harness-mvp

## Intent

Fork of `revfactory/harness` ported to OpenCode, but all surfaces still describe a Claude Code plugin: broken install paths (`~/.claude/skills/`), orphaned `.claude-plugin/`, wrong platform references. Clean identity first, then add OpenCode-native generation.

## Scope

### In Scope
- Identity refactor: README.md, README_KO.md, README_JA.md, index.html, privacy.html, plugin.json
- Remove `.claude-plugin/`
- Update GitHub links to this fork
- AGENTS.md as entry point
- `.opencode/skills/` generation (primary) + `.agents/skills/` (optional compat)
- `evals/promptfoo/` for validation
- `examples/mezaos/` as reference only

### Out of Scope
- Translating Korean reference files (`skills/harness/references/`)
- OpenCode plugin format (defer)
- Renaming "Harness" brand
- Full landing page redesign
- CI pipeline
- MezaOS as operational dependency

## Capabilities

### New Capabilities
- `identity-refactor`: Clean Claude Code from all surfaces
- `agent-generation`: Generate AGENTS.md + `.opencode/agents/`
- `skill-generation`: Generate skills to `.opencode/skills/` + `.agents/skills/` compat
- `eval-infrastructure`: promptfoo evals validating generated structure

### Modified Capabilities
None.

## Approach

**Phase 1 — Identity Refactor**: Scrub Claude Code from READMEs, index.html, privacy.html, plugin.json. Update recommended install paths to local per-project `.opencode/skills/`. Remove `.claude-plugin/`. Update GitHub links. Core skill logic already adapted — leave it.

**Phase 2 — OpenCode Generation**: AGENTS.md. Update harness to emit `.opencode/` paths. Add `.agents/skills/` compat. Add evals. Add MezaOS reference.

Korean content in `skills/harness/` stays — author's native language.

## Affected Areas

| Area | Impact | |
|------|--------|-|
| READMEs (EN/KO/JA) | Modified | Claude→OpenCode, paths, install |
| index.html | Modified | Title, i18n, workflow |
| privacy.html | Modified | Platform, output paths |
| `.claude-plugin/` | Removed | Orphaned metadata |
| AGENTS.md | New | Entry point |
| skills/harness/SKILL.md | Modified | Generation paths |
| evals/promptfoo/ | New | Eval infra |
| examples/mezaos/ | New | Reference only |

## Risks

| Risk | Likelihood | Mitigation |
|------|------------|------------|
| Identity confusion during transition | Med | Phase 1 first |
| MezaOS as mistaken dependency | Low | Clear docs, no runtime link |

## Rollback Plan

Phase 1: revert single commit. Phase 2: additive (git revert or remove). `.claude-plugin/` deletion recovered via `git restore`.

## Dependencies

OpenCode's `.opencode/` convention (stable). No external packages.

## Success Criteria

- [ ] No Claude Code references in any public surface
- [ ] GitHub links point to this fork
- [ ] `.claude-plugin/` removed
- [ ] AGENTS.md is the entry point
- [ ] Harness generates `.opencode/agents/` and `.opencode/skills/`
- [ ] `.agents/skills/` exists as compat
- [ ] promptfoo evals validate generated structure
- [ ] MezaOS reference-only, not a dependency
