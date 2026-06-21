# Format Cleanup Report

## Modified Files

- `AGENTS.md`
- `README.md`
- `skills/harness/SKILL.md`
- `docs/specs/opencode-team-harness-mvp/tasks.md`
- `evals/promptfoo/promptfooconfig.yaml`
- `docs/audit/format-cleanup-report.md`

## What Was Corrected

- Separated Markdown headings, lists, tables, and code fences for easier review.
- Kept `README.md` in Spanish and preserved the recommended local per-project installation path.
- Converted `skills/harness/SKILL.md` frontmatter to readable multi-line YAML and preserved Korean body content.
- Added explicit MVP acceptance criteria and manual verification checks to the tasks artifact.
- Completed promptfoo coverage for local installation, no runtime AI dependency, `.opencode` routes, optional `.agents/skills` compatibility, and MezaOS as an example only.

## What Remains Pending

- No functional work remains in this cleanup phase.
- Promptfoo execution itself was not required by this phase; the YAML configuration was validated locally.

## How to Verify

```bash
python - <<'PY'
import pathlib
import yaml

yaml.safe_load(pathlib.Path('evals/promptfoo/promptfooconfig.yaml').read_text())

skill = pathlib.Path('skills/harness/SKILL.md').read_text()
frontmatter = skill.split('---', 2)[1]
data = yaml.safe_load(frontmatter)
assert isinstance(data, dict)
assert 'trigger' not in data
PY

git diff --check
git diff --stat
git status --short
```
