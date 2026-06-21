---
name: harness
description: >-
  Configure a harness. A meta skill that defines specialized agents and creates
  the skills those agents use. Use this when (1) the user asks to "configure a
  harness" or "build a harness", (2) the user asks for "harness design" or
  "harness engineering", (3) building a harness-based automation system for a
  new domain/project, or (4) reconfiguring or extending an existing harness.
compatibility:
  - opencode
metadata:
  portable_skills: true
---

# Harness — Agent & Skill Architect

A meta skill that configures a harness for a domain/project, defines each agent's role, and creates
the skills those agents use.

**Core principles:**

1. Create agent definitions (`.opencode/agents/{{AGENT_NAME}}.md`) and skills
   (`.opencode/skills/{{SKILL_NAME}}/SKILL.md`).
2. **Use parallel/sequential Task tool calls as the default execution mode.**

## Workflow

### Phase 1: Domain Analysis
1. Identify the domain/project from the user request.
2. Identify the core work types (generation, validation, editing, analysis, etc.).
3. Check existing agents/skills to prevent conflicts and duplication.
4. Explore the project codebase — understand the tech stack, data models, and major modules.
5. **Detect user proficiency** — infer technical level from conversational clues (terminology used,
   question depth), then adjust communication tone. Do not use terms like "assertion" or "JSON
   schema" with less experienced users without explaining them.

### Phase 2: Task Architecture Design

#### 2-1. Choose execution mode: parallel vs sequential Task

**The default is parallel Task calls.** When two or more agents can perform independent work in
parallel, call Task concurrently with `run_in_background: true`. Receive each Task result in the
parent session and aggregate them.

Use sequential calls when a previous Task result is required as input for the next Task.

#### 2-2. Choose architecture pattern

1. Decompose the work into specialist areas.
2. Decide the Task call structure (see architecture patterns in
   references/agent-design-patterns.md).
   - **Pipeline**: sequential dependent work
   - **Fan-out/Fan-in**: parallel independent work
   - **Expert Pool**: choose specialists by situation
   - **Producer-Reviewer**: generate, then review quality
   - **Supervisor**: central agent manages state and dynamic distribution

#### 2-3. Agent separation criteria

Evaluate across four axes: specialization, parallelism, context, and reusability. See the "Agent
separation criteria" table in references/agent-design-patterns.md for details.

#### Task tool default parameters

| Parameter | Type | Description |
| --- | --- | --- |
| name | string | Name of the agent to call |
| prompt | string | Instructions passed to the agent |
| model | string | `provider/model` format (example: openrouter/minimax/minimax-m2.7) |
| tools | object | Enable/disable tools in `{ tool_name: boolean }` form |
| run_in_background | boolean | `true` for parallel execution, `false` to wait sequentially |
| steps | number | Maximum iteration count (optional) |

#### Agent call methods

**1. Task tool (explicit call)**

```text
Task({
  name: "explore",
  prompt: "Explore the codebase",
  model: "{provider/model}",
  tools: { edit: false, write: false }
})
```

**2. @mention (direct call inside a message)**

```text
@explore Explore this codebase
```

- Call a subagent directly from the message.
- Receive the result in the parent session.

### Phase 3: Create Agent Definitions

**Every agent MUST be defined as a `.opencode/agents/{{AGENT_NAME}}.md` file.** Do not put roles
directly into Task tool prompts. Reasons:

- Agent definitions must exist as files so they can be reused in later sessions.
- Task collaboration protocols must be explicit to ensure high-quality aggregation.
- The harness's core value is separating agents (who) from skills (how).

Even when using built-in types (`general-purpose`, `Explore`, `Plan`), create an agent definition
file. Specify it through the Task tool `name` parameter, and put the role and principles in the
agent definition file.

**Model setting:** Set the `model` parameter in `provider/model` format. Use a model configured in
the user's OpenCode config. Examples: `anthropic/claude-sonnet-4-20250514`,
`openrouter/minimax/minimax-m2.7`.

Define each agent in `.opencode/agents/{{AGENT_NAME}}.md`. Required sections: core responsibilities,
working principles, input/output protocol, and error handling.

> For the definition template and full examples, see "Agent definition structure" in
> references/agent-design-patterns.md and references/team-examples.md.

**Required when including a QA agent:**

- Use the `general-purpose` type for QA agents (`Explore` is read-only, so it cannot run validation
  scripts).
- QA is not about "existence checks"; its core is **cross-boundary comparison** — read API responses
  and frontend hooks together, then compare their shapes.
- Run QA incrementally **immediately after each module is completed**, not only once after the whole
  system is finished.
- Detailed guide: references/qa-agent-guide.md.

### Phase 4: Create Skills

Create each agent's skills in `.opencode/skills/{{SKILL_NAME}}/SKILL.md`. If portable compatibility
is needed, mirror them under `.agents/skills/{{SKILL_NAME}}/SKILL.md`. See
references/skill-writing-guide.md for the detailed writing guide.

#### 4-1. Skill structure

```text
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (name and description required)
│   └── Markdown body
└── Bundled Resources (optional)
    ├── scripts/    - executable code for repeatable/deterministic work
    ├── references/ - reference docs loaded conditionally
    └── assets/     - files used in outputs (templates, images, etc.)
```

#### 4-2. Writing descriptions — encourage active triggering

The `description` is the skill's main trigger mechanism. OpenCode does not rely on a `trigger:`
frontmatter key, so put trigger information in the description, metadata, or body. Write
descriptions **actively ("pushy")**.

**Bad example:** "A skill for processing PDF documents"

**Good example:** "Perform all PDF tasks, including reading PDF files, text/table extraction,
merging, splitting, rotating, watermarking, encryption, and OCR. Use this skill whenever a .pdf file
is mentioned or a PDF output is requested."

Key point: describe both what the skill does and the concrete trigger situations, and distinguish it
from similar cases that should not trigger it.

#### 4-3. Body writing principles

| Principle | Description |
| --- | --- |
| **Explain why** | Instead of coercive instructions like "ALWAYS/NEVER", explain why the rule exists. LLMs make better edge-case decisions when they understand the reason. |
| **Keep it lean** | The context window is a shared resource. Aim for SKILL.md bodies under 500 lines; remove anything that does not justify its weight or move it to references/. |
| **Generalize** | Explain principles so the skill handles diverse inputs, instead of narrow rules fitted to one example. Do not overfit. |
| **Bundle repeated code** | If agents repeatedly write the same helper scripts during tests, bundle them in scripts/. |
| **Use imperative voice** | Write as instructions using direct imperative wording. |

#### 4-4. Progressive Disclosure

Skills use a three-stage loading system to manage context:

| Stage | When loaded | Size target |
| --- | --- | --- |
| **Metadata** (name + description) | Always present in context | ~100 words |
| **SKILL.md body** | When the skill triggers | <500 lines |
| **references/** | Only when needed | Unlimited (scripts can run without being loaded) |

**Size management rules:**

- When SKILL.md approaches 500 lines, split details into references/ and leave pointers in the body
  explaining when to read each file.
- Include a **table of contents (ToC)** at the top of reference files over 300 lines.
- If there are domain/framework variants, split them into domain-specific files under references/ so
  only relevant files are loaded.

#### 4-5. Skill-agent connection principles

- One agent ↔ one or more skills (1:1 or 1:N).
- Multiple agents may share a skill.
- Skills contain "how to do it"; agents contain "who does it".

> For detailed writing patterns, examples, and data schema standards, see
> references/skill-writing-guide.md.

### Phase 5: Integration and Orchestration

An orchestrator is a special kind of skill that links individual agents and skills into one workflow
and coordinates the whole system. See references/orchestrator-template.md for concrete templates.

#### 5-0. Orchestrator patterns by mode

**Parallel Task calls (default):**

Call multiple Tasks concurrently with `run_in_background: true`. Receive each Task result in the
parent session and aggregate them.

```text
[orchestrator]
    ├── Task({ name: "agent-1", run_in_background: true })
    ├── Task({ name: "agent-2", run_in_background: true })
    ├── wait for all Tasks to finish
    ├── collect and integrate results
    └── create final output
```

**Sequential Task calls:**

Pass the previous Task result into the next Task and execute sequentially.

```text
[orchestrator]
    ├── Task({ name: "agent-1" }) → result: _workspace/01_artifact.md
    ├── Task({ name: "agent-2", input: 01 result }) → result: _workspace/02_artifact.md
    └── integrate results
```

#### 5-1. Data handoff protocol

| Strategy | Method | Best fit |
| --- | --- | --- |
| **File-based** | Write files to _workspace/ and read them | Large data, structured artifacts, audit trail |
| **Task result handoff** | Pass Task results as input to the next Task | Work with sequential dependencies |

Rules for file-based handoff:

- Create an _workspace/ folder under the working directory and store intermediate artifacts there.
- Filename convention: {phase}_{agent}_{artifact}.{ext} (example: 01_analyst_requirements.md).
- Output only the final artifact to the user-specified path; preserve intermediate files
  (_workspace/) for post-hoc verification and audit trail.

#### 5-2. Error handling

Include an error handling plan in the orchestrator. Core principle: retry once; if retry also fails,
proceed without that result and explicitly note the omission in the report. Do not delete
conflicting data; cite sources side by side.

> For the strategy table by error type and implementation details, see "Error handling" in
> references/orchestrator-template.md.

#### 5-3. Team size guidelines

| Work size | Recommended Task count | Notes |
| --- | --- | --- |
| Small (5–10 tasks) | 2–3 parallel Tasks | 3–5 tasks per Task |
| Medium (10–20 tasks) | 3–5 parallel Tasks | 4–6 tasks per Task |
| Large (20+ tasks) | 5–7 parallel Tasks | 4–5 tasks per Task |

> More Tasks increase coordination overhead. Three focused Tasks are better than five scattered
> Tasks.

### Phase 6: Validation and Testing

Validate the generated harness. See references/skill-testing-guide.md for the detailed testing
methodology.

#### 6-1. Structure validation

- Confirm every agent file is in the correct location.
- Validate skill frontmatter (`name`, `description`).
- Confirm Task call pattern consistency.

#### 6-2. Validation by execution mode

- Parallel Task mode: check `run_in_background` settings and result aggregation paths.
- Sequential Task mode: check input/output connections between Tasks.

#### 6-3. Skill execution tests

1. **Write test prompts** — write 2–3 realistic test prompts for each skill.

2. **Compare With-skill vs Without-skill runs** — when possible, run with the skill and without the
   skill in parallel to confirm the skill's added value. Execute two Tasks:
   - **With-skill**: read the skill and perform the work.
   - **Without-skill (baseline)**: perform the same prompt without the skill.

3. **Evaluate results** — evaluate artifact quality qualitatively (user review) and quantitatively
   (assertion-based).

4. **Iterative improvement loop** — when test results reveal issues, generalize the fix into the
   skill and retest.

5. **Bundle repeated patterns** — if tests repeatedly create the same code, bundle it under
   scripts/.

#### 6-4. Trigger validation

Validate that each skill's description triggers correctly:

1. **Should-trigger queries** (8–10) — varied expressions that should trigger the skill.
2. **Should-NOT-trigger queries** (8–10) — near-miss queries with similar keywords where another
   tool/skill is more appropriate.

**Key point for near-misses:** obviously unrelated queries such as "write a Fibonacci function" have
little test value. **Ambiguous boundary queries** make better test cases.

#### 6-5. Dry-run tests

- Review whether the orchestrator skill's Phase order is logical.
- Check for dead links in data handoff paths.
- Confirm every Task input matches the previous Phase output.

#### 6-6. Write test scenarios

- Add a `## Test Scenarios` section to the orchestrator skill.
- Include one happy path and at least one error path.

## Output Checklist

After generation, confirm:

- [ ] `.opencode/agents/{{AGENT_NAME}}.md` — **agent definition file is required** (even for
  built-in types)
- [ ] `.opencode/skills/{{SKILL_NAME}}/SKILL.md` — skill files (SKILL.md + references/)
- [ ] `.agents/skills/{{SKILL_NAME}}/SKILL.md` — only when portable compatibility is needed
- [ ] One orchestrator skill (including data flow, error handling, and test scenarios)
- [ ] Execution mode is explicit (parallel or sequential Task)
- [ ] Every Task call includes the `model: "{provider/model}"` parameter
- [ ] No conflicts with existing agents/skills
- [ ] Skill descriptions are written actively ("pushy")
- [ ] SKILL.md body is under 500 lines, with excess split into references/
- [ ] Execution validated with 2–3 test prompts
- [ ] Trigger validation complete (should-trigger + should-NOT-trigger)

## References

- Harness patterns: references/agent-design-patterns.md
- Existing harness examples: references/team-examples.md
- Orchestrator template: references/orchestrator-template.md
- **Skill writing guide**: references/skill-writing-guide.md
- **Skill testing guide**: references/skill-testing-guide.md
- **QA agent guide**: references/qa-agent-guide.md
