<!--
  OpenCode compatibility documentation for the harness.
-->

# OpenCode Compatibility

This document explains how OpenCode Team Harness integrates with OpenCode
and which installation mode is recommended.

## Requirements

- **OpenCode 0.x+** — the harness uses OpenCode project files and task
  delegation.
- **File system access** — generated artifacts are plain Markdown files in
  the consumer project's `.opencode/` directory.
- **No runtime dependencies** — the harness is a meta-skill, not a daemon,
  service, or runtime AI component.

## Recommended installation

Install the harness locally in each consumer project:

```shell
mkdir -p .opencode/skills
cp -r /path/to/OpenCode-Team-Harness/skills/harness .opencode/skills/harness
```

Local per-project installation is recommended because the generated team,
prompts, and skill behavior stay versioned with the repository they support.

## Advanced installation

Global installation is possible for advanced users who intentionally want
the same harness available across projects:

```shell
mkdir -p ~/.config/opencode/skills
cp -r /path/to/OpenCode-Team-Harness/skills/harness ~/.config/opencode/skills/harness
```

Use the global path only when shared behavior across projects is intentional.
Project-local skills remain easier to review, pin, and audit.

## Integration points

### 1. Agent definitions

OpenCode uses project instructions from `AGENTS.md` and can load individual
agent files from `.opencode/agents/`. The harness generates both.

### 2. Skills

OpenCode loads project skills from `.opencode/skills/{{SKILL_NAME}}/SKILL.md`.
The harness generates valid `SKILL.md` files with YAML frontmatter that
OpenCode can parse.

### 3. Tool access

Generated agents use OpenCode's built-in tools. No external tooling is
required, and no external service, daemon, or runtime AI dependency is
required by the generated artifacts.

## Migration from another agent setup

If you are migrating an existing project:

1. Install OpenCode using the
   [official installation guide](https://opencode.ai/docs/#install).
2. Copy `skills/harness/` to `.opencode/skills/harness/` inside the consumer
   project.
3. Invoke the harness skill from within your consumer project.
4. Review generated output under `.opencode/` before committing it.

## Official OpenCode references

- [OpenCode docs](https://opencode.ai/docs/)
- [OpenCode config](https://opencode.ai/docs/config/)
- [OpenCode agents](https://opencode.ai/docs/agents/)
- [OpenCode skills](https://opencode.ai/docs/skills/)

## Known limitations

- OpenCode skills are loaded from the filesystem; this harness does not
  require a plugin marketplace.
- `trigger:` is not a required skill frontmatter key. Put trigger guidance in
  `description`, `metadata`, or the skill body.
