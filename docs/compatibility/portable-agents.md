<!--
  Portable agent compatibility documentation (SG-3, SG-5).
-->
# Portable Agent Compatibility

> **Purpose**: Document the `.agents/skills/` portable mirror and when to use it.

## Overview

The harness generates skills to `.opencode/skills/` as the primary path. When portable compat is enabled, a deterministic mirror is also written to `.agents/skills/<skill-name>/SKILL.md`.

This mirrors the structure used by earlier agent runtime conventions, allowing compatibility with tools that scan `.agents/` directories.

## When to Enable Portable Compat

| Scenario | Compat Needed? | Reason |
|----------|---------------|--------|
| Pure OpenCode project | No | `.opencode/skills/` is sufficient |
| Mixed toolchain | Yes | Other tools may scan `.agents/` |
| Transitional setup | Yes | Migration period before dropping `.agents/` |
| Fork of a Claude Code project | Possibly | Check if any tool depends on `.agents/skills/` |

## How It Works

1. The harness generates the primary skill at `.opencode/skills/<name>/SKILL.md`
2. If portable compat is enabled, a deterministic copy is written to `.agents/skills/<name>/SKILL.md`
3. `.opencode/skills/` is always authoritative — `.agents/skills/` is a derived artifact (SG-4)
4. `.agents/skills/` is NOT required for normal operation (SG-5)

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

Both files are identical. The mirror step is deterministic — same template input produces the same output.
