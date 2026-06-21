<!--
  Template: AGENTS.md entry point
  Placeholders: {{TEAM_NAME}}, {{TEAM_DESCRIPTION}}, {{AGENT_ROWS}}, {{TEAM_OVERVIEW}}
  Deterministic — no AI inference (AG-8, CC-3)
-->
# {{TEAM_NAME}} — Agent Team

{{TEAM_DESCRIPTION}}

## Agents

| Agent | Role | Trigger guidance |
| --- | --- | --- |
{{AGENT_ROWS}}

## Overview

{{TEAM_OVERVIEW}}

## Generated file routes

- Agent definitions: `.opencode/agents/{{AGENT_NAME}}.md`
- Authoritative skill definitions: `.opencode/skills/{{SKILL_NAME}}/SKILL.md`
- Optional portable skill mirror: `.agents/skills/{{SKILL_NAME}}/SKILL.md`

## Communication model

This team uses OpenCode's parallel task system. Agents do not call each other directly — they coordinate through the orchestrator, which delegates work based on trigger conditions. Each agent operates independently within its defined scope.

## Getting started

1. Place each agent definition under `.opencode/agents/{{AGENT_NAME}}.md`.
2. Place each authoritative skill under `.opencode/skills/{{SKILL_NAME}}/SKILL.md`.
3. Optionally mirror portable skills under `.agents/skills/{{SKILL_NAME}}/SKILL.md`.
4. Reference agent roles and trigger guidance in your orchestrator configuration.
