<!--
  This repo IS the harness meta-skill — it generates agent teams for consumer projects.
  See skills/harness/SKILL.md for the entry point.
  The templates at skills/harness/templates/ produce AGENTS.md files for NEW projects.
-->
# OpenCode Team Harness

**Orchestrator agent**: Generates structured agent teams for OpenCode projects.

This repository contains the meta-skill at `skills/harness/SKILL.md`. When invoked from a consumer project, it generates:

- `AGENTS.md` — team entry point with role definitions.
- `.opencode/agents/{{AGENT_NAME}}.md` — individual agent specs.
- `.opencode/skills/{{SKILL_NAME}}/SKILL.md` — domain skills per agent.
- `.agents/skills/{{SKILL_NAME}}/SKILL.md` — optional portable skill mirror.

## Generated agent roles

| Agent | Role | Trigger guidance |
|---|---|---|
| Domain Analyzer | Analyze project context and choose architecture. | User invokes Harness for a project/domain. |
| Agent Writer | Generate agent definitions from templates. | Domain analysis is complete. |
| Skill Writer | Generate skill definitions from templates. | Agent definitions are complete. |
| Validator | Validate generated output against spec. | All generation tasks are complete. |

## Usage

Invoke Harness from a consumer project. The meta-skill prompts for project context, fills templates, and writes generated artifacts into that consumer project's directory.

See `docs/agentic-development/team-architecture.md` for architecture rationale.
