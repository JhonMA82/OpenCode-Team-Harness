<!--
  Template: AGENTS.md entry point
  Placeholders: {{TEAM_NAME}}, {{TEAM_DESCRIPTION}}, {{AGENT_ROWS}}, {{TEAM_OVERVIEW}}
  Deterministic — no AI inference (AG-8, CC-3)
-->
# {{TEAM_NAME}} — Agent Team

{{TEAM_DESCRIPTION}}

## Agents

| Agent | Role | Trigger |
|-------|------|---------|
{{AGENT_ROWS}}

## Overview

{{TEAM_OVERVIEW}}

## Communication Model

This team uses OpenCode's parallel task system. Agents do not call each other directly — they coordinate through the orchestrator, which delegates work based on trigger conditions. Each agent operates independently within its defined scope.

## Getting Started

1. Place each agent definition under `.opencode/agents/<agent-name>.md`
2. Place each skill under `.opencode/skills/<skill-name>/SKILL.md`
3. Reference agent roles and triggers in your orchestrator configuration
