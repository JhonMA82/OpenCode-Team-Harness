# Format and Content Sanity Report

This report records the documentation and configuration cleanup pass for OpenCode Team Harness.

## Modified files

| File | Change |
| --- | --- |
| `AGENTS.md` | Reworked headings, output table, and section flow while preserving required generated routes. |
| `docs/compatibility/opencode.md` | Clarified project-local installation as recommended, global installation as advanced, and replaced repository links with official OpenCode documentation links. |
| `docs/compatibility/portable-agents.md` | Clarified that `.opencode/skills/` is authoritative and `.agents/skills/` is an optional derived mirror. |
| `skills/harness/templates/AGENTS.md` | Normalized Markdown headings, table formatting, route list, and placeholder readability. |
| `skills/harness/templates/skill.md` | Expanded YAML frontmatter to use `name`, `description`, `compatibility`, and `metadata`; moved trigger guidance into metadata and body content. |
| `evals/promptfoo/promptfooconfig.yaml` | Added local assertions for runtime dependency language, install defaults, OpenCode skill routes, optional portable mirror behavior, and MezaOS example scope. |
| `docs/specs/opencode-team-harness-mvp/tasks.md` | Added a completed cleanup phase with acceptance criteria and manual verification steps. |

## Problems corrected

- Documentation now leads with the recommended project-local OpenCode installation path.
- Global installation is documented only as an advanced option.
- OpenCode docs links now point to `https://opencode.ai/docs/` pages.
- Portable compatibility docs now state that `.agents/skills/` is optional and derived.
- Skill template frontmatter is valid YAML and does not rely on a top-level `trigger:` key.
- Promptfoo config includes explicit sanity checks for the requested content contracts.

## Pending problems

- No pending problems found in this cleanup pass.

## Manual verification steps

1. Parse `evals/promptfoo/promptfooconfig.yaml` as YAML.
2. Parse `skills/harness/templates/skill.md` frontmatter as YAML.
3. Parse `skills/harness/SKILL.md` frontmatter as YAML.
4. Run `git diff --check`.
5. Run `git diff --stat`.
6. Run `git status --short`.
