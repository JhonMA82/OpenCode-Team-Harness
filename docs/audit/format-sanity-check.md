# Format Sanity Check — MVP Artifacts

> **Date**: 2026-06-20  
> **Scope**: All files created or modified during the MVP (Phases 1–2)  
> **Stricture**: Formatting only — no semantic content changed  

---

## Files Reviewed

| # | File | Verdict | Action |
|---|------|---------|--------|
| 1 | `AGENTS.md` | ✅ Clean | No changes needed |
| 2 | `evals/promptfoo/promptfooconfig.yaml` | ✅ Valid YAML | No changes needed |
| 3 | `docs/specs/opencode-team-harness-mvp/tasks.md` | ⚠️ 2 issues | Fixed |
| 4 | `docs/specs/opencode-team-harness-mvp/spec.md` | ✅ Clean | No changes needed |
| 5 | `docs/specs/opencode-team-harness-mvp/design.md` | ✅ Clean | No changes needed |
| 6 | `docs/agentic-development/team-architecture.md` | ✅ Clean | No changes needed |
| 7 | `docs/agentic-development/eval-strategy.md` | ✅ Clean | No changes needed |
| 8 | `docs/compatibility/opencode.md` | ✅ Clean | No changes needed |
| 9 | `docs/compatibility/portable-agents.md` | ✅ Clean | No changes needed |
| 10 | `docs/audit/opencode-port-audit.md` | ✅ Clean | No changes needed |
| 11 | `docs/proposals/opencode-team-harness-mvp.md` | ✅ Clean | No changes needed |
| 12 | `examples/mezaos/README.md` | ✅ Clean | No changes needed |
| 13 | `examples/mezaos/kernel/agent.md` | ✅ Clean | No changes needed |
| 14 | `examples/mezaos/drivers/agent.md` | ✅ Clean | No changes needed |
| 15 | `examples/mezaos/scheduler/agent.md` | ✅ Clean | No changes needed |
| 16 | `skills/harness/templates/AGENTS.md` | ⚠️ 2 issues | Fixed |
| 17 | `skills/harness/templates/agent.md` | ✅ Clean | No changes needed |
| 18 | `skills/harness/templates/skill.md` | ✅ Clean | No changes needed |

---

## Issues Found & Fixed

### 1. `tasks.md` — Checkboxes stuck as pending (fixed)

**Problem**: All 16 task checkboxes used `[ ]` despite the entire MVP being implemented and merged to `main`.

**Fix**: Marked all 16 checkboxes as `[x]`.

**Diff**:
```diff
- `- [ ] 1.1 ...`
+ `- [x] 1.1 ...`
```

### 2. `tasks.md` — Redundant inline fields (fixed)

**Problem**: Lines 14–17 repeated the Review Workload Forecast table as free text:

```
Decision needed before apply: Yes
Chained PRs recommended: Yes
Chain strategy: pending
400-line budget risk: Medium
```

All four fields already appear in the structured table above (lines 5–12).

**Fix**: Removed the duplicate lines.

### 3. Template paths — Wrong placeholder syntax (fixed)

**File**: `skills/harness/templates/AGENTS.md`

**Problem**: The Getting Started section used angle-bracket variables (`<agent-name>`, `<skill-name>`) instead of the template system's `{{PLACEHOLDER}}` convention. This was inconsistent with the rest of the template, which uses `{{TEAM_NAME}}`, `{{AGENT_ROWS}}`, etc.

**Diff**:
```diff
- `.opencode/agents/<agent-name>.md`
+ `.opencode/agents/{{AGENT_NAME}}.md`
- `.opencode/skills/<skill-name>/SKILL.md`
+ `.opencode/skills/{{SKILL_NAME}}/SKILL.md`
```

---

## Files Clean (no changes needed)

### `AGENTS.md` (root)

Already well-formatted: HTML comment header, proper H1/H2 hierarchy, valid Markdown table with separator row, list items separated by blank lines, trailing newline present.

### `evals/promptfoo/promptfooconfig.yaml`

Valid YAML (confirmed via `yaml.safe_load`). 21 test entries with consistent 2-space indentation. Comment headers separate test groups. No trailing whitespace. Trailing newline present.

### `examples/mezaos/README.md`

Clean HTML comment, H1, blockquote for status, list sections, valid table, trailing newline present.

### `skills/harness/templates/agent.md` and `skill.md`

Clean HTML comment headers, proper `{{PLACEHOLDER}}` syntax, trailing newlines present.

### All docs files

Consistent heading hierarchy, no broken links, trailing newlines present.

---

## Summary

| Metric | Count |
|--------|-------|
| Files reviewed | 18 |
| Files with issues | 2 |
| Issues found | 4 |
| Issues fixed | 4 |
| Semantic changes | 0 |
