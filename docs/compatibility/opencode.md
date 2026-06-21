<!--
  OpenCode compatibility documentation for the harness.
-->
# OpenCode Compatibility

> **Purpose**: Document how the OpenCode Team Harness integrates with OpenCode.

## Requirements

- **OpenCode 0.x+** — the harness uses the `task` tool for agent dispatch and standard I/O for communication
- **File system** — generated artifacts are plain markdown files placed in the consumer project's `.opencode/` directory
- **No runtime dependencies** — the harness is a meta-skill, not a daemon or CLI tool

## Integration Points

### 1. Agent Definitions

OpenCode reads agent definitions from `AGENTS.md` at the project root and individual agent files from `.opencode/agents/`. The harness generates both.

### 2. Skills

OpenCode loads project skills from `.opencode/skills/{{SKILL_NAME}}/SKILL.md`. The harness generates these with proper frontmatter that OpenCode can parse.

### 3. Tool Access

Generated agents use only OpenCode's built-in tools — `task`, `read`, `write`, `grep`, `bash`. No external tooling is required.

## Migration from Claude Code

If you are migrating from a Claude Code project:

1. Copy `skills/harness/` to `.opencode/skills/harness/` inside the consumer project
2. Install OpenCode (see [OpenCode docs](https://github.com/JhonMA82/OpenCode-Team-Harness))
3. Invoke the harness skill from within your consumer project
4. Generated output goes to `.opencode/`, keeping your existing `.claude/` config untouched

## Known Limitations

- OpenCode does not support `/plugin marketplace add` — skills are loaded from the filesystem
- OpenCode's `AGENTS.md` format may differ from Claude Code's `AGENTS.md` in trigger syntax — check the [OpenCode documentation](https://github.com/JhonMA82/OpenCode-Team-Harness) for the latest format
