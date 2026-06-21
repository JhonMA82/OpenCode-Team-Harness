---
name: "{{SKILL_NAME}}"
description: >-
  {{SKILL_DESCRIPTION}}
compatibility:
  opencode: ">=0.x"
metadata:
  trigger_guidance: "{{TRIGGER_CONDITION}}"
  deterministic: "true"
---

# {{SKILL_NAME}}

<!--
  Template: OpenCode skill definition
  Placeholders: {{SKILL_NAME}}, {{SKILL_DESCRIPTION}}, {{TRIGGER_CONDITION}},
                {{SKILL_CONTENT}}
  Deterministic — no AI inference (AG-8, CC-3, SG-2, SG-6)
-->

{{SKILL_CONTENT}}

## Trigger guidance

Use this skill when the user request matches this condition:

> {{TRIGGER_CONDITION}}

## References

{{REFERENCES}}
