<!--
  Portable agent compatibility documentation (SG-3, SG-5).
-->
# Portable Agent Compatibility

This document explains when the optional `.agents/skills/` mirror is useful and why `.opencode/skills/` remains authoritative.

## Overview

The harness generates skills to `.opencode/skills/` as the primary path. That directory is authoritative for OpenCode projects.

When portable compatibility is enabled, a deterministic mirror is also written to `.agents/skills/{{SKILL_NAME}}/SKILL.md`.

This mirrors the structure used by earlier agent runtime conventions, allowing compatibility with tools that scan `.agents/` directories.

Pure OpenCode projects do not need `.agents/skills/`.

## When to Enable Portable Compat

| Scenario | Mirror needed? | Reason |
| --- | --- | --- |
| Pure OpenCode project | No | `.opencode/skills/` is sufficient and authoritative. |
| Mixed toolchain | Yes | Other tools may scan `.agents/`. |
| Transitional setup | Yes | The mirror can support migration before dropping `.agents/`. |
| Existing project with `.agents/` consumers | Possibly | Check whether any tool still depends on `.agents/skills/`. |

## How It Works

1. The harness generates the authoritative skill at `.opencode/skills/{{SKILL_NAME}}/SKILL.md`.
2. If portable compatibility is enabled, a deterministic copy is written to `.agents/skills/{{SKILL_NAME}}/SKILL.md`.
3. `.agents/skills/` is a derived artifact, not the source of truth.
4. `.agents/skills/` is not required for pure OpenCode operation.

## File Structure with Compat

```
consumer-project/
├── .opencode/           # Primary (authoritative)
│   └── skills/
│       └── agent-alpha/
│           └── SKILL.md
└── .agents/             # Portable mirror (derived)
    └── skills/
        └── agent-alpha/
            └── SKILL.md
```

Both files are identical when the mirror is enabled. The mirror step is deterministic: the same template input produces the same output.
