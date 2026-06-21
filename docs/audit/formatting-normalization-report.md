# Formatting Normalization Report

This report records the formatting normalization pass for OpenCode Team
Harness. No new features, intent changes, architecture changes, or commits were
made.

## Files normalized

| # | File | Change |
| --- | --- | --- |
| 1 | `AGENTS.md` | Verified. Already well-formatted (38 lines, proper headings, tables, all required routes). No changes needed. |
| 2 | `README.md` | Wrapped long prose lines at natural sentence boundaries. Long lines reduced from 10 to 5 (remaining are HTML badges, table rows, and blockquote references — structurally required). |
| 3 | `skills/harness/SKILL.md` | Fixed YAML frontmatter: added `compatibility: [opencode]` and `metadata: {portable_skills: true}`. Added missing closing `---` delimiter. Frontmatter now parses as valid YAML. |
| 4 | `evals/promptfoo/promptfooconfig.yaml` | Verified. Already valid multiline YAML. 28 tests, all with properly separated assertions. No compressed single-line lists. |
| 5 | `examples/mezaos/README.md` | Verified. Already well-formatted (33 lines, proper headings, table, notes). |
| 6 | `docs/specs/opencode-team-harness-mvp/tasks.md` | Verified. Already well-formatted (79 lines, proper headings, tables, checkboxes, acceptance criteria). |

## Verification results

| Check | Result |
| --- | --- |
| `python3 -c "import yaml; yaml.safe_load(open('evals/promptfoo/promptfooconfig.yaml'))"` | PASS |
| `python3 -c "import yaml; yaml.safe_load(SKILL.md frontmatter)"` | PASS (after adding closing `---`) |
| `grep -RInP "[\x{AC00}-\x{D7AF}]" skills/harness/SKILL.md` | PASS — no Korean text in SKILL.md |
| `grep -RInP "[\x{AC00}-\x{D7AF}]" skills/harness/` | Korean text detected ONLY in `references/` files (placeholders in templates: `{도메인}`, `{역할}`, etc.) — these are intentional template examples, not in the core skill |
| `git diff --check` | PASS — no whitespace errors |
| AGENTS.md route check (3 required paths) | PASS — all three routes present |

## Residual Korean text (non-blocking)

Korean characters remain as placeholder values inside `skills/harness/references/`:

- `references/skill-testing-guide.md` — 2 lines with `{테스트 프롬프트}`, `{스킬 이름}`
- `references/orchestrator-template.md` — 16+ lines with `{도메인}`, `{역할}`, `{무엇을 파악하는지}`, `{최종 산출물}`, etc.

These are **intentional template examples** showing what generated output looks like
for a Korean-language harness project. They are not translation remnants in the
core skill logic. The Korean text is valid placeholder content that demonstrates
the template system works for non-English projects.

To remove these, the reference files would need structural changes to use
English-only placeholders — that is an intentional choice, not an oversight.

## Manual verification

1. ✅ `evals/promptfoo/promptfooconfig.yaml` parses as YAML (28 tests)
2. ✅ `skills/harness/SKILL.md` frontmatter parses as YAML (keys: name, description, compatibility, metadata)
3. ✅ `git diff --check` — no whitespace errors
4. ✅ `git diff --stat` — see summary below

## Diff summary

```text
Modified:  README.md, skills/harness/SKILL.md
Created:   docs/audit/formatting-normalization-report.md
Unchanged: AGENTS.md, evals/promptfoo/promptfooconfig.yaml,
           examples/mezaos/README.md,
           docs/specs/opencode-team-harness-mvp/tasks.md
```
