# Tasks: OpenCode Team Harness — MVP

## Review Workload Forecast

| Field | Value |
| --- | --- |
| Estimated changed lines | ~700–800 |
| 400-line budget risk | Medium |
| Chained PRs recommended | Yes |
| Suggested split | PR 1: Identity Refactor → PR 2: OpenCode Generation |
| Delivery strategy | ask-on-risk |
| Chain strategy | pending |

### Suggested Work Units

| Unit | Goal | Likely PR | Notes |
| --- | --- | --- | --- |
| 1 | **Identity Refactor** — strip Claude Code from READMEs (EN/KO/JA), index.html, privacy.html; remove `.claude-plugin/`; update GitHub links; clean SKILL.md line 13 | PR 1 | base=main; standalone completion; verifiable by `grep -r "Claude Code"` |
| 2 | **OpenCode Generation** — AGENTS.md, `skills/harness/templates/`, docs (compatibility, agentic-development), evals/promptfoo/, examples/mezaos/ | PR 2 | depends on PR 1 (clean identity first); additive only, no deletions |

## Task List

The MVP work is split into reviewable phases. Phase 1 removes inherited platform identity from
public surfaces. Phase 2 adds OpenCode-native generation artifacts, compatibility docs, examples,
and local-file eval coverage. Phase 3 performs format and content sanity cleanup.

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

## Phase 3: Format and Content Sanity Cleanup

- [x] 3.1 Reformat `AGENTS.md` with real headings, lists, and route-preserving tables.
- [x] 3.2 Reformat `docs/compatibility/opencode.md` with local installation as recommended, global
  installation as advanced, and official OpenCode documentation links.
- [x] 3.3 Reformat `docs/compatibility/portable-agents.md` to make `.opencode/skills/` authoritative
  and `.agents/skills/` optional.
- [x] 3.4 Reformat `skills/harness/templates/AGENTS.md` with valid Markdown, a valid table, and
  readable placeholders.
- [x] 3.5 Correct `skills/harness/templates/skill.md` frontmatter to valid YAML using `name`,
  `description`, `compatibility`, and `metadata`.
- [x] 3.6 Complete `evals/promptfoo/promptfooconfig.yaml` with minimal local assertions for runtime,
  install, route, mirror, and MezaOS scope behavior.
- [x] 3.7 Create `docs/audit/format-content-sanity-report.md` documenting changes, pending problems,
  and manual verification.

## Acceptance Criteria

- [x] Public project documentation describes OpenCode Team Harness, not a Claude Code plugin.
- [x] Recommended installation path is local per-project installation under
  `.opencode/skills/harness`.
- [x] Generated OpenCode routes are `.opencode/agents/` and `.opencode/skills/`.
- [x] `.agents/skills/` is documented only as an optional portable compatibility mirror.
- [x] Skill frontmatter uses valid YAML with `name` and `description`; trigger guidance does not
  depend on a top-level `trigger:` key.
- [x] The MezaOS content is presented as a reference example, not as the default stack or required
  target.
- [x] Promptfoo coverage uses local file assertions and does not require network access.
- [x] Markdown cleanup preserves the required OpenCode routes and optional portable mirror route.
- [x] OpenCode compatibility docs recommend project-local installation and present global
  installation only as advanced.
- [x] Promptfoo config parses as YAML and includes sanity checks for no runtime AI recommendation,
  no default global install, OpenCode skill routes, optional `.agents/skills/`, and MezaOS example
  scope.

## Manual Verification

- [x] Inspect `README.md` and confirm the recommended install command copies `skills/harness` into
  `.opencode/skills/harness` inside the consumer project.
- [x] Inspect `AGENTS.md` and confirm generated outputs list `.opencode/agents/`,
  `.opencode/skills/`, and optional `.agents/skills/` compatibility only.
- [x] Parse `skills/harness/SKILL.md` frontmatter as YAML and confirm it has no top-level `trigger:`
  key.
- [x] Parse `evals/promptfoo/promptfooconfig.yaml` as YAML.
- [x] Run `git diff --check` before review.
- [x] Parse `skills/harness/templates/skill.md` frontmatter as YAML.
- [x] Parse `skills/harness/SKILL.md` frontmatter as YAML.
- [x] Run `git diff --stat` and `git status --short` before returning.
