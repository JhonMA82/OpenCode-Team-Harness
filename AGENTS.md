<!--
  This repo IS the harness meta-skill — it generates agent teams for consumer projects.
  See skills/harness/SKILL.md for the entry point.
  The templates at skills/harness/templates/ produce AGENTS.md files for NEW projects.
-->
# OpenCode Team Harness

OpenCode Team Harness is a meta-skill for generating structured OpenCode agent teams in consumer projects.

## Purpose

The harness acts as the orchestrator for a generated team. It analyzes the target project, chooses an agent-team shape, and writes deterministic OpenCode artifacts.

## Generated outputs

| Output | Purpose |
| --- | --- |
| `AGENTS.md` | Team entry point with role definitions. |
| `.opencode/agents/{{AGENT_NAME}}.md` | Individual OpenCode agent definitions. |
| `.opencode/skills/{{SKILL_NAME}}/SKILL.md` | Authoritative project skill definitions. |
| `.agents/skills/{{SKILL_NAME}}/SKILL.md` | Optional portable skill mirror. |

## Generated agent roles

| Agent | Role | Trigger guidance |
| --- | --- | --- |
| Domain Analyzer | Analyze project context and choose architecture. | User invokes Harness for a project/domain. |
| Agent Writer | Generate agent definitions from templates. | Domain analysis is complete. |
| Skill Writer | Generate skill definitions from templates. | Agent definitions are complete. |
| Validator | Validate generated output against spec. | All generation tasks are complete. |

## Usage

Invoke Harness from a consumer project. The meta-skill prompts for project context, fills templates, and writes generated artifacts into that consumer project's directory.

## References

See `docs/agentic-development/team-architecture.md` for architecture rationale.
