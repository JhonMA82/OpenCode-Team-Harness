---
name: harness
description: >-
  Configures a harness. Defines specialized agents and creates the skills those agents will use — a meta-skill.
  (1) When requested to 'configure a harness' or 'build a harness', (2) When requested for 'harness design' or 'harness engineering',
  (3) When building a harness-based automation system for a new domain/project,
  (4) When restructuring or extending an existing harness configuration.
---

# Harness — Agent & Skill Architect

A meta-skill that configures a harness tailored to a domain/project, defines each agent's role, and creates the skills those agents will use.

**Core Principles:**

1. Creates agent definitions (`.opencode/agents/{{AGENT_NAME}}.md`) and skills (`.opencode/skills/{{SKILL_NAME}}/SKILL.md`).
2. **Uses Task tool with parallel/sequential invocation as the default execution mode.**

## Workflow

### Phase 1: Domain Analysis
1. Identify the domain/project from the user's request
2. Identify core task types (creation, validation, editing, analysis, etc.)
3. Check existing agents/skills (prevent conflicts/duplicates)
4. Explore the project codebase — identify tech stack, data models, key modules
5. **Detect user proficiency** — Gauge technical level from conversational context cues (terminology used, question depth), and adjust communication tone accordingly. Avoid using terms like "assertion" or "JSON schema" without explanation for users with less coding experience.

### Phase 2: Task Architecture Design

#### 2-1. Execution Mode Selection: Task Parallel vs Sequential

**The default is parallel Task invocation.** When 2 or more agents perform independent work in parallel, invoke Tasks simultaneously with run_in_background: true. Each Task result is received and aggregated by the parent session.

Sequential invocation is chosen when a previous Task's result is needed as input for the next Task.

#### 2-2. Architecture Pattern Selection

1. Decompose work into specialized domains
2. Determine Task invocation structure (see references/agent-design-patterns.md for architecture patterns)
   - **Pipeline**: Sequential dependent tasks
   - **Fan-out/Fan-in**: Parallel independent tasks
   - **Expert Pool**: Situational selective invocation
   - **Producer-Reviewer**: Generation followed by quality review
   - **Supervisor**: Central agent manages state and dynamically distributes work

#### 2-3. Agent Separation Criteria

Evaluate along 4 axes: specialization, parallelism, context, and reusability. See the "Agent Separation Criteria" table in references/agent-design-patterns.md for details.

#### Task Tool Basic Parameters

| Parameter | Type | Description |
| --- | --- | --- |
| name | string | Name of the agent to invoke |
| prompt | string | Instructions to pass to the agent |
| model | string | provider/model format (e.g., openrouter/minimax/minimax-m2.7) |
| tools | object | { tool_name: boolean } to enable/disable tools |
| run_in_background | boolean | true for parallel execution, false for sequential wait |
| steps | number | Maximum iteration count (optional) |

#### Agent Invocation Methods

**1. Task tool (explicit invocation)**

```text
Task({
  name: "explore",
  prompt: "Explore the codebase",
  model: "{provider/model}",
  tools: { edit: false, write: false }
})
```

**2. @mention (direct invocation in message)**

```text
@explore Explore this codebase
```

- Invokes a subagent directly in a message
- Results are received in the parent session

### Phase 3: Agent Definition Creation

**All agents must be defined as `.opencode/agents/{{AGENT_NAME}}.md` files.** Putting roles directly in the Task tool's prompt is prohibited. Reasons:

- Agent definitions must exist as files to be reusable across sessions
- Inter-Task collaboration protocols must be explicit to ensure result aggregation quality
- The core value of a harness is the separation of agents (who) and skills (how)

Even when using built-in types (general-purpose, Explore, Plan), create an agent definition file. Specify via the Task tool's name parameter, and include roles and principles in the agent definition file.

**Model configuration:** The `model` parameter is specified in `provider/model` format. Use models configured in the user's OpenCode settings (config). Examples: `anthropic/claude-sonnet-4-20250514`, `openrouter/minimax/minimax-m2.7`, etc.

Define each agent in `.opencode/agents/{{AGENT_NAME}}.md`. Required sections: Core Role, Working Principles, Input/Output Protocol, Error Handling.

> For definition templates and full file examples, see "Agent Definition Structure" in references/agent-design-patterns.md + references/team-examples.md.

**Required when including a QA agent:**

- Use general-purpose type for the QA agent (Explore is read-only and cannot execute validation scripts)
- The core of QA is not "existence checking" but **"boundary cross-comparison"** — simultaneously reading API responses and front-end hooks to compare shapes
- QA should not run once after full completion, but **incrementally after each module is completed** (incremental QA)
- Detailed guide: references/qa-agent-guide.md

### Phase 4: Skill Creation

Create skills for each agent at `.opencode/skills/{{SKILL_NAME}}/SKILL.md`. Mirror to `.agents/skills/{{SKILL_NAME}}/SKILL.md` if portable compatibility is needed. See references/skill-writing-guide.md for detailed writing guide.

#### 4-1. Skill Structure

```text
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (name, description required)
│   └── Markdown body
└── Bundled Resources (optional)
    ├── scripts/    - Executable code for repetitive/deterministic tasks
    ├── references/ - Reference documents loaded conditionally
    └── assets/     - Files used in output (templates, images, etc.)
```

#### 4-2. Description Writing — Aggressive Trigger Promotion

The description is the skill's primary trigger mechanism. OpenCode does not rely on a `trigger:` frontmatter key, so trigger information should be written in the description, metadata, or body. Write descriptions **aggressively ("pushy")**.

**Bad example:** "A skill that processes PDF documents"

**Good example:** "Performs all PDF operations including reading, text/table extraction, merging, splitting, rotating, watermarking, encryption, and OCR. Always use this skill when a .pdf file is mentioned or PDF output is requested."

Key: Describe both what the skill does + specific trigger situations, and differentiate from similar cases that should NOT trigger.

#### 4-3. Body Writing Principles

| Principle | Description |
| --- | --- |
| **Explain the Why** | Instead of coercive directives like "ALWAYS/NEVER", convey the reason why. LLMs make correct judgments on edge cases when they understand the reasoning. |
| **Keep it Lean** | The context window is a shared resource. Target SKILL.md body at under 500 lines; remove or move content that doesn't pull its weight to references/. |
| **Generalize** | Explain principles rather than narrow rules that only fit specific examples, enabling handling of diverse inputs. No overfitting. |
| **Bundle repetitive code** | If agents commonly write the same scripts during test runs, pre-bundle them in scripts/. |
| **Write in imperative mood** | Use imperative/directive tone: "Do X", "Use Y". |

#### 4-4. Progressive Disclosure

Skills manage context through a 3-tier loading system:

| Tier | Loading Timing | Size Target |
| --- | --- | --- |
| **Metadata** (name + description) | Always in context | ~100 words |
| **SKILL.md body** | When skill triggers | <500 lines |
| **references/** | Only when needed | Unlimited (scripts can be executed without loading) |

**Size management rules:**

- When SKILL.md approaches 500 lines, split details into references/ and leave a pointer in the body indicating "when to read this file"
- Reference files over 300 lines should include a **Table of Contents (ToC)** at the top
- If there are domain/framework variations, split by domain under references/ so only relevant files are loaded

#### 4-5. Skill-Agent Connection Principles

- 1 agent ↔ 1~N skills (1:1 or 1:many)
- Skills shared across multiple agents are possible
- Skills contain "how to do it", agents contain "who does it"

> For detailed writing patterns, examples, and data schema standards, see references/skill-writing-guide.md.

### Phase 5: Integration and Orchestration

An orchestrator is a specialized form of skill that weaves individual agents and skills into a single workflow to coordinate the whole. See references/orchestrator-template.md for specific templates.

#### 5-0. Orchestrator Patterns by Mode

**Parallel Task Invocation (default):**

Invoke multiple Tasks simultaneously with run_in_background: true for parallel execution. Each Task result is received and aggregated by the parent session.

```text
[Orchestrator]
    ├── Task({ name: "agent-1", run_in_background: true })
    ├── Task({ name: "agent-2", run_in_background: true })
    ├── Wait for all Tasks to complete
    ├── Aggregate and integrate results
    └── Generate final output
```

**Sequential Task Invocation:**

Pass previous Task results as input to the next Task, executing sequentially.

```text
[Orchestrator]
    ├── Task({ name: "agent-1" }) → Result: _workspace/01_artifact.md
    ├── Task({ name: "agent-2", input: 01 result }) → Result: _workspace/02_artifact.md
    └── Integrate results
```

#### 5-1. Data Passing Protocol

| Strategy | Method | Suitable When |
| --- | --- | --- |
| **File-based** | Write to and read from _workspace/ | Large data, structured outputs, audit trails |
| **Task result passing** | Pass Task result as next Task input | Sequential dependency between tasks |

File-based passing rules:

- Create a _workspace/ folder under the working directory to store intermediate outputs
- File naming convention: {phase}_{agent}_{artifact}.{ext} (e.g., 01_analyst_requirements.md)
- Only final outputs go to the user-specified path; intermediate files (_workspace/) are preserved (for post-verification and audit trails)

#### 5-2. Error Handling

Include error handling strategies in the orchestrator. Core principle: retry once, then proceed without that result on second failure (note the omission in the report); conflicting data is not deleted but cited with sources.

> For error type strategy tables and implementation details, see "Error Handling" in references/orchestrator-template.md.

#### 5-3. Team Size Guidelines

| Work Scale | Recommended Task Count | Notes |
| --- | --- | --- |
| Small (5–10 tasks) | 2–3 parallel Tasks | 3–5 tasks per Task |
| Medium (10–20 tasks) | 3–5 parallel Tasks | 4–6 tasks per Task |
| Large (20+ tasks) | 5–7 parallel Tasks | 4–5 tasks per Task |

> More tasks means more coordination overhead. 3 focused tasks are better than 5 scattered ones.

### Phase 6: Verification and Testing

Verify the generated harness. See references/skill-testing-guide.md for detailed testing methodology.

#### 6-1. Structural Verification

- Confirm all agent files are in the correct location
- Validate skill frontmatter (name, description)
- Check Task invocation pattern consistency

#### 6-2. Execution Mode Verification

- Parallel Task mode: Verify run_in_background settings and result aggregation paths
- Sequential Task mode: Verify input/output connections between Tasks

#### 6-3. Skill Execution Testing

1. **Write test prompts** — Create 2–3 realistic test prompts for each skill.

2. **With-skill vs Without-skill comparison** — When possible, run both with and without the skill in parallel to confirm the skill's added value. Run two Tasks for each:
   - **With-skill**: Read skill and perform task
   - **Without-skill (baseline)**: Perform same prompt without skill

3. **Evaluate results** — Assess output quality both qualitatively (user review) + quantitatively (assertion-based).

4. **Iterative improvement loop** — When problems are found in test results, generalize and modify the skill, then retest.

5. **Bundle repetitive patterns** — Pre-bundle commonly written code from test runs into scripts/.

#### 6-4. Trigger Verification

Verify each skill's description triggers correctly:

1. **Should-trigger queries** (8–10) — Various phrasings that should trigger the skill
2. **Should-NOT-trigger queries** (8–10) — "Near-miss" queries with similar keywords but where a different tool/skill is appropriate

**Key for near-miss writing:** Obviously unrelated queries like "write a Fibonacci function" have no test value. **Queries with ambiguous boundaries** make good test cases.

#### 6-5. Dry-run Testing

- Review whether the orchestrator skill's Phase order is logical
- Verify there are no dead links in data passing paths
- Confirm all Task inputs match the outputs of previous Phases

#### 6-6. Test Scenario Writing

- Add a ## Test Scenarios section to the orchestrator skill
- Describe at least 1 normal flow + 1 error flow

## Output Checklist

Confirm after generation:

- [ ] `.opencode/agents/{{AGENT_NAME}}.md` — **Agent definition files must be created** (even for built-in types)
- [ ] `.opencode/skills/{{SKILL_NAME}}/SKILL.md` — Skill files (SKILL.md + references/)
- [ ] `.agents/skills/{{SKILL_NAME}}/SKILL.md` — Create only when portable compatibility is needed
- [ ] 1 orchestrator skill (includes data flow + error handling + test scenarios)
- [ ] Execution mode specified (Task parallel or sequential)
- [ ] All Task invocations include model: "{provider/model}" parameter
- [ ] No conflicts with existing agents/skills
- [ ] Skill description is written aggressively ("pushy")
- [ ] SKILL.md body is under 500 lines; split to references/ if exceeded
- [ ] Execution verified with 2–3 test prompts
- [ ] Trigger verification (should-trigger + should-NOT-trigger) completed

## References

- Harness patterns: references/agent-design-patterns.md
- Existing harness examples: references/team-examples.md
- Orchestrator template: references/orchestrator-template.md
- **Skill writing guide**: references/skill-writing-guide.md
- **Skill testing guide**: references/skill-testing-guide.md
- **QA agent guide**: references/qa-agent-guide.md
