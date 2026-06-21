# Tasks: OpenCode Team Harness — MVP

## Review Workload Forecast

| Field | Value |
|---|---|
| Estimated changed lines | ~700–800 |
| 400-line budget risk | Medium |
| Chained PRs recommended | Yes |
| Suggested split | PR 1: Identity Refactor → PR 2: OpenCode Generation |
| Delivery strategy | ask-on-risk |
| Chain strategy | pending |

### Suggested Work Units

| Unit | Goal | Likely PR | Notes |
|---|---|---|---|
| 1 | **Identity Refactor** — strip Claude Code from READMEs (EN/KO/JA), index.html, privacy.html; remove `.claude-plugin/`; update GitHub links; clean SKILL.md line 13 | PR 1 | base=main; standalone completion; verifiable by `grep -r "Claude Code"` |
| 2 | **OpenCode Generation** — AGENTS.md, `skills/harness/templates/`, docs (compatibility, agentic-development), evals/promptfoo/, examples/mezaos/ | PR 2 | depends on PR 1 (clean identity first); additive only, no deletions |

## Phase 1: Identity Refactor

- [x] 1.1 Remove `.claude-plugin/` directory (plugin.json + marketplace.json)
- [x] 1.2 Strip Claude Code from README.md (9 changes: paths, platform refs, env var section)
- [x] 1.3 Strip Claude Code from README_KO.md (same 9 transformations in Korean)
- [x] 1.4 Strip Claude Code from README_JA.md (same 9 transformations in Japanese)
- [x] 1.5 Strip Claude Code from index.html (8 changes: title, badge, i18n, workflow, links)
- [x] 1.6 Strip Claude Code from privacy.html (7 changes: platform, paths, links)
- [x] 1.7 Remove Claude compat line from skills/harness/SKILL.md (line 13)
- [x] 1.8 Update GitHub links to fork across all files (READMEs, HTML, privacy)

## Phase 2: OpenCode Generation

- [x] 2.1 Create AGENTS.md entry point with agent roles and triggers
- [x] 2.2 Create `skills/harness/templates/` with deterministic `{{PLACEHOLDER}}` templates
- [x] 2.3 Create `docs/agentic-development/team-architecture.md`
- [x] 2.4 Create `docs/compatibility/opencode.md`
- [x] 2.5 Create `docs/compatibility/portable-agents.md`
- [x] 2.6 Create `examples/mezaos/` reference-only example
- [x] 2.7 Create `evals/promptfoo/promptfooconfig.yaml` with local file assertions
- [x] 2.8 Create `docs/agentic-development/eval-strategy.md`
