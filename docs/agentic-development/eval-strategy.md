<!--
  EI-8 (MAY): How to extend evals.
-->
# Eval Strategy

> **Purpose**: Explain how to extend and run the promptfoo-based evaluation suite.

## Quick Start

```bash
# Run all evals from the repo root
npx promptfoo eval -c evals/promptfoo/promptfooconfig.yaml
```

No global install required (CC-2). No network access needed — all assertions are local file checks (EI-7).

## What's Covered

| Category | Assertions | Spec Refs |
|----------|-----------|-----------|
| Identity refactor | `not-contains` on Claude Code refs across all public surfaces | EI-3 |
| File structure | `file-exists` on AGENTS.md, templates, docs, evals | EI-2 |
| Template determinism | `contains` on placeholder syntax (`{{PLACEHOLDER}}`) | AG-8, CC-3 |
| Example integrity | `file-exists` on MezaOS reference files | AG-7 |

## Adding New Eval Tests

### 1. Add a `not-contains` Assertion

For checking that no forbidden strings remain in a file:

```yaml
- description: "No legacy path in README"
  assert:
    - type: not-contains
      value: "~/.claude/"
      file: "../../README.md"
```

### 2. Add a `file-exists` Assertion

For checking that generated output exists:

```yaml
- description: "Generated agent exists"
  assert:
    - type: file-exists
      value: "../../.opencode/agents/code-reviewer.md"
```

### 3. Add a `contains` Assertion

For checking that template output has expected structure:

```yaml
- description: "Skill has frontmatter"
  assert:
    - type: contains
      value: "trigger:"
      file: "../../.opencode/skills/reviewer/SKILL.md"
```

## Writing New Test Categories

Add a new test group under `tests:` in `evals/promptfoo/promptfooconfig.yaml`. Each test is an object with `description` and `assert` array:

```yaml
tests:
  - description: "My new category"
    assert:
      - type: file-exists
        value: "../../path/to/file"
```

## Determinism Validation

To test generation determinism (CC-3):

```bash
# Run generation twice
npx promptfoo eval -c evals/promptfoo/promptfooconfig.yaml --output first-run.json
npx promptfoo eval -c evals/promptfoo/promptfooconfig.yaml --output second-run.json

# Compare outputs
diff first-run.json second-run.json  # Should be empty
```

## promptfoo Reference

| Assertion Type | Description |
|---------------|-------------|
| `file-exists` | Pass if file exists |
| `not-exists` | Pass if file does NOT exist |
| `contains` | Pass if file contains the string |
| `not-contains` | Pass if file does NOT contain the string |

For more assertion types, see the [promptfoo documentation](https://www.promptfoo.dev/docs/configuration/expected-outputs/).
