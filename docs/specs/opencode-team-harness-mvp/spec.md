# OpenCode Team Harness — MVP Specification

> **Change**: opencode-team-harness-mvp  
> **Status**: Draft  
> **Domains**: 4 new — identity-refactor, agent-generation, skill-generation, eval-infrastructure

---

## 1. identity-refactor

**Purpose**: Strip all Claude Code references from the fork's public surfaces so the project reads as an OpenCode-native tool.

### Requirements

| ID | Requirement | Strength |
|----|-------------|----------|
| IR-1 | All README files (EN, KO, JA) MUST reference OpenCode, not Claude Code | MUST |
| IR-2 | Recommended install paths in docs MUST use local per-project `.opencode/skills/` installation — no `~/.claude/` paths | MUST |
| IR-3 | `index.html` and `privacy.html` MUST remove Claude Code branding and platform references | MUST |
| IR-4 | `plugin.json` MUST reference OpenCode, not Claude Code | MUST |
| IR-5 | `.claude-plugin/` directory MUST be removed from the repository | MUST |
| IR-6 | All GitHub links MUST point to the opencode-team-harness fork, not the upstream | MUST |
| IR-7 | Skill generation output paths MUST default to `.opencode/` | MUST |
| IR-8 | `.agents/skills/` support MUST NOT remove or override `.opencode/skills/` as primary | MUST |

#### Scenario: READMEs fully migrated

- GIVEN README.md, README_KO.md, and README_JA.md
- WHEN each is reviewed for Claude Code references
- THEN no instance of "Claude Code", "Claude", or `~/.claude/` remains in prose
- AND install instructions recommend local per-project `.opencode/skills/` installation

#### Scenario: Orphaned plugin directory removed

- GIVEN the repository has a `.claude-plugin/` directory
- WHEN the identity refactor runs
- THEN `.claude-plugin/` no longer exists in the working tree

#### Scenario: No false positives on skill logic (edge)

- GIVEN core skill logic at `skills/harness/` contains Claude Code references in Korean reference files
- WHEN the identity refactor runs
- THEN those references MUST NOT be modified (out of scope — author's native language)

---

## 2. agent-generation

**Purpose**: Generate structured agent team definitions consumable by OpenCode — AGENTS.md as entry point plus individual agent specs.

### Requirements

| ID | Requirement | Strength |
|----|-------------|----------|
| AG-1 | The system MUST generate an AGENTS.md file as the team entry point | MUST |
| AG-2 | AGENTS.md MUST describe each agent's role, trigger, and responsibility | MUST |
| AG-3 | Individual agent definitions MUST be written to `.opencode/agents/{{AGENT_NAME}}.md` | MUST |
| AG-4 | Each agent definition MUST include: name, purpose, trigger conditions, and output contract | MUST |
| AG-5 | The system MUST generate `docs/agentic-development/team-architecture.md` with architecture rationale | MUST |
| AG-6 | The team architecture doc SHOULD include interaction flow between agents | SHOULD |
| AG-7 | MezaOS MUST appear as a reference example only — not as a runtime dependency | MUST |
| AG-8 | Agent roles MUST be deterministic from project context (no AI inference at generation time) | MUST |

#### Scenario: Full team generation

- GIVEN project context (name, description, tech stack)
- WHEN the harness generates an agent team
- THEN AGENTS.md is created at project root
- AND each agent has a corresponding file under `.opencode/agents/`
- AND `docs/agentic-development/team-architecture.md` describes the architecture

#### Scenario: MezaOS not operational (edge)

- GIVEN the MezaOS example reference
- WHEN the generated team is validated
- THEN no generated file imports, calls, or depends on MezaOS code or data at runtime

---

## 3. skill-generation

**Purpose**: Generate OpenCode-compatible skills that encode domain expertise for each agent role.

### Requirements

| ID | Requirement | Strength |
|----|-------------|----------|
| SG-1 | Generated skills MUST be placed in `.opencode/skills/{{SKILL_NAME}}/SKILL.md` | MUST |
| SG-2 | Each skill MUST include valid frontmatter with `name` and `description`; trigger guidance MUST live in description, metadata, or body, not a `trigger:` frontmatter key | MUST |
| SG-3 | The system SHOULD also generate a portable copy under `.agents/skills/{{SKILL_NAME}}/SKILL.md` | SHOULD |
| SG-4 | `.opencode/skills/` SHALL be treated as the primary/authoritative location | MUST |
| SG-5 | `.agents/skills/` MUST NOT be required for normal operation | MUST |
| SG-6 | Skills MUST NOT contain runtime AI logic — behavior is deterministic from context | MUST |
| SG-7 | Generated skill content MUST be auditable in Git (plain markdown, no binary artifacts) | MUST |

#### Scenario: Primary skill generated

- GIVEN an agent definition with a domain role
- WHEN skill generation runs
- THEN `.opencode/skills/{{SKILL_NAME}}/SKILL.md` exists with valid frontmatter
- AND the skill is a plain markdown file with no encoded prompts

#### Scenario: Portable compat optional (edge)

- GIVEN `.opencode/skills/` has been generated
- WHEN the system runs with portable compat enabled
- THEN `.agents/skills/{{SKILL_NAME}}/SKILL.md` mirrors the primary
- WHEN the system runs with portable compat disabled
- THEN `.agents/skills/` is NOT created

---

## 4. eval-infrastructure

**Purpose**: Provide promptfoo-based evals that validate the harness output matches spec requirements.

### Requirements

| ID | Requirement | Strength |
|----|-------------|----------|
| EI-1 | The system MUST include evals under `evals/promptfoo/` | MUST |
| EI-2 | Evals MUST validate generated file existence and structure for AGENTS.md, `.opencode/agents/`, `.opencode/skills/` | MUST |
| EI-3 | Evals MUST validate that no Claude Code references remain after identity refactor | MUST |
| EI-4 | Evals SHOULD validate `.agents/skills/` mirroring when portable compat is enabled | SHOULD |
| EI-5 | Evals MUST be runnable with `npx promptfoo eval` — no global install required | MUST |
| EI-6 | Eval config MUST be under `evals/promptfoo/promptfooconfig.yaml` | MUST |
| EI-7 | Evals MUST NOT require network access — all assertions are local file/system checks | MUST |
| EI-8 | A `docs/agentic-development/eval-strategy.md` MAY explain how to extend evals | MAY |

#### Scenario: Identity refactor validated

- GIVEN the identity refactor has been applied
- WHEN `npx promptfoo eval` runs from `evals/promptfoo/`
- THEN all assertions pass confirming no Claude Code references remain in READMEs, HTML, plugin.json

#### Scenario: Generated structure validated

- GIVEN agent and skill generation has run
- WHEN promptfoo evals execute
- THEN assertions pass confirming AGENTS.md, `.opencode/agents/`, `.opencode/skills/` all exist

#### Scenario: No network dependency (edge)

- GIVEN a machine with no internet access
- WHEN promptfoo eval runs
- THEN all tests pass because assertions are local file checks only

---

## Cross-Cutting Constraints

| ID | Requirement | Strength |
|----|-------------|----------|
| CC-1 | No global config modification (e.g., `~/.config/opencode/`, shell rc) | MUST NOT |
| CC-2 | No global dependency installation (all tools via `npx` or local install) | MUST NOT |
| CC-3 | No persistent AI model calls — generation is deterministic, prompt-free tooling | MUST NOT |
| CC-4 | No file deletion without documented justification in the proposal | MUST NOT |
| CC-5 | All changes MUST be auditable in Git (plain text, no binary blobs) | MUST |
| CC-6 | Consumer project rules (the project using the harness) take priority over harness conventions | MUST |

---

## Coverage Summary

| Domain | Requirements | Scenarios | Happy | Edge/Error |
|--------|-------------|-----------|-------|------------|
| identity-refactor | 8 | 3 | 2 | 1 |
| agent-generation | 8 | 2 | 1 | 1 |
| skill-generation | 7 | 2 | 1 | 1 |
| eval-infrastructure | 8 | 3 | 2 | 1 |
| cross-cutting | 6 | — | — | — |
| **Total** | **37** | **10** | **6** | **4** |
