<!--
  This repo IS the harness meta-skill — it generates agent teams for consumer projects.
  See skills/harness/SKILL.md for the entry point.
  The templates at skills/harness/templates/ produce AGENTS.md files for NEW projects.
-->
# OpenCode Team Harness

**Orchestrator Agent**: Generate structured agent teams for OpenCode projects.

This repository contains the meta-skill (`skills/harness/SKILL.md`) that, when invoked, generates:

- `AGENTS.md` — team entry point with role definitions
- `.opencode/agents/*.md` — individual agent specs
- `.opencode/skills/*/SKILL.md` — domain skills per agent

## Agent Roles (generated)

| Agent | Role | Trigger |
|-------|------|---------|
| Domain Analyzer | Analyze project context and choose architecture | User invokes harness |
| Agent Writer | Generate agent definitions from templates | Domain analysis done |
| Skill Writer | Generate skill definitions from templates | Agent definitions done |
| Validator | Validate generated output against spec | All generation done |

## Usage

Invoke the harness from a consumer project. The meta-skill will prompt for project context, fill templates, and write generated artifacts into the consumer's project directory.

See `docs/agentic-development/team-architecture.md` for architecture rationale.
