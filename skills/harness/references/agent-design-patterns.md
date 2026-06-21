# Agent & Task Design Patterns

## Execution mode: Parallel vs sequential Task

Understand the core difference between the two execution modes and choose the appropriate mode.

### Parallel Task calls — default mode

Call multiple Tasks concurrently with `run_in_background: true`. Each Task result is received and aggregated in the parent session.

[parent]
    ├── Task({ name: "agent-A", run_in_background: true })
    ├── Task({ name: "agent-B", run_in_background: true })
    └── collect and integrate results


**Key parameters:**
- Task({ name, prompt, model, tools, run_in_background })

**Characteristics:**
- Reduces time through parallel execution.
- Each Task runs in an independent context.
- Parent is responsible for collecting results.

**Constraints:**
- Tasks cannot communicate directly with each other (results are passed through parent).
- Use sequential calls for work with dependencies.

### Sequential Task calls — dependency mode

Pass the previous Task result into the next Task as input and execute sequentially.

[parent]
    ├── Task({ name: "agent-1" }) → result: _workspace/01_artifact.md
    └── Task({ name: "agent-2", input: 01 result }) → result: _workspace/02_artifact.md


**Best fit:**
- Each step depends on the previous step's artifact.
- Pipeline work.

### Mode selection decision tree

Can the work run in parallel? (independent work)
├── Yes → parallel Task calls (default)
│         collect results in parent
│
└── No (sequential dependency) → sequential Task calls
                    pass previous result into next input

> **Core principle:** independent work is parallel; sequential dependency is sequential.

---

## Task architecture types

### 1. Pipeline
Sequential workflow. The previous Task output becomes the next Task input.

[analysis] → [design] → [implementation] → [verification]

**Best fit:** each step strongly depends on the previous step's artifact.
**Example:** novel writing — worldbuilding → characters → plot → drafting → editing.
**Caution:** bottlenecks delay the entire pipeline. Design each step to be as independent as possible.

### 2. Fan-out/Fan-in
Parallel processing followed by integration. Independent work runs concurrently.

         ┌→ [SpecialistA] ─┐
[distribute] → ├→ [SpecialistB] ─┼→ [integrate]
         └→ [SpecialistC] ─┘

**Best fit:** the same input needs analysis from different perspectives/areas.
**Example:** integrated research — official/media/community/background research in parallel → integrated report.
**Caution:** integration quality determines overall quality.
**Task pattern:** MUST use parallel Task calls.

### 3. Expert Pool
Select and call the appropriate specialist based on the situation.

[router] → { expertA | expertB | expertC }

**Best fit:** different handling is needed depending on input type.
**Example:** code review — call only the relevant security/performance/architecture specialists.
**Caution:** router classification accuracy is critical.
**Task pattern:** call Tasks sequentially or in parallel as needed.

### 4. Producer-Reviewer
A generation Task and validation Task work as a pair.

[produce] → [review] → (if issue) → rerun [produce]

**Best fit:** artifact quality assurance matters and objective validation criteria exist.
**Example:** webtoon — artist generates → reviewer checks → regenerate problem panels.
**Caution:** set a maximum retry count (2–3) to prevent infinite loops.

### 5. Supervisor
A central agent manages work state and dynamically distributes work to child agents.

         ┌→ [WorkerA]
[supervisor] ─┼→ [WorkerB]    ← supervisor watches state and distributes dynamically
         └→ [WorkerC]

**Best fit:** workload is variable or work distribution must be decided at runtime.
**Example:** large code migration — supervisor analyzes the file list and assigns batches to workers.
**Difference from fan-out:** fan-out pre-assigns fixed work; supervisor adjusts dynamically based on progress.

## Agent call methods

### 1. Task tool (explicit call)

Task({
  name: "explore",
  prompt: "Explore the codebase",
  model: "{provider/model}",
  tools: { edit: false, write: false }
})


### 2. @mention (direct call inside a message)

@explore Explore this codebase

- Call a subagent directly from the message.
- Receive the result in the parent session.
- Suitable for simple work.

## Agent type selection

Specify the agent type through the Task tool `name` parameter.

### Built-in types

| Type | Tool access | Best fit |
|------|----------|-----------|
| general-purpose | Full (including WebSearch, WebFetch) | Web research, general-purpose work |
| Explore | Read-only (no Edit/Write) | Codebase exploration, analysis |
| Plan | Read-only (no Edit/Write) | Architecture design, planning |

### Custom types

If an agent is defined in `.opencode/agents/{{AGENT_NAME}}.md`, it can be called with `name: "{{AGENT_NAME}}"`. Custom agents have access to the full toolset.

### Selection criteria

| Situation | Recommended | Reason |
|------|------|------|
| Role is complex and reused across sessions | **Custom type** (`.opencode/agents/`) | Manages persona and working principles in a file |
| Simple research/collection where a prompt is enough | **general-purpose** + detailed prompt | No agent file needed; instructions can live in the prompt |
| Only code reading is needed (analysis/review) | **Explore** | Prevents accidental file changes |
| Only design/planning is needed | **Plan** | Focuses on analysis and prevents code changes |
| Implementation work requiring file changes | **Custom type** | Full tool access + specialized instructions |

**Principle:** every agent MUST be defined in a `.opencode/agents/{{AGENT_NAME}}.md` file. Even for built-in types, create an agent definition file that states role and principles. The file must exist so it can be reused in later sessions.

**Model setting:** Set the `model` parameter in `provider/model` format. Use a model configured in the user's OpenCode config. Examples: `anthropic/claude-sonnet-4-20250514`, `openrouter/minimax/minimax-m2.7`.

## Agent definition: Markdown vs JSON

### Markdown (`.opencode/agents/{{AGENT_NAME}}.md`)

markdown
---
description: "1–2 sentence role description. List trigger keywords."
mode: subagent
model: openrouter/minimax/minimax-m2.7
tools:
  write: false
  bash: false
---

# Agent Name — one-line role summary

You are a [role] specialist in [domain].

## Core Responsibilities
1. Responsibility 1
2. Responsibility 2

## Working Principles
- Principle 1
- Principle 2

## Input/Output Protocol
- Input: [where input comes from and what it contains]
- Output: [where output is written and what it contains]
- Format: [file format, structure]

## Error Handling
- [behavior on failure]
- [behavior on timeout]


### JSON (opencode.json)

json
{
  "agent": {
    "my-agent": {
      "description": "role description",
      "mode": "subagent",
      "model": "openrouter/minimax/minimax-m2.7",
      "tools": {
        "write": false,
        "bash": false
      }
    }
  }
}


### Selection criteria
- **Markdown**: agent definition is complex and has a long system prompt.
- **JSON**: simple configuration changes or built-in agent overrides.

## Agent definition structure

markdown
---
name: agent-name
description: "1–2 sentence role description. List trigger keywords."
---

# Agent Name — one-line role summary

You are a [role] specialist in [domain].

## Core Responsibilities
1. Responsibility 1
2. Responsibility 2

## Working Principles
- Principle 1
- Principle 2

## Input/Output Protocol
- Input: [where input comes from and what it contains]
- Output: [where output is written and what it contains]
- Format: [file format, structure]

## Error Handling
- [behavior on failure]
- [behavior on timeout]


## Agent separation criteria

| Criterion | Separate | Combine |
|------|------|------|
| Specialization | Separate when areas differ | Combine when areas overlap |
| Parallelism | Separate when independent execution is possible | Consider combining when sequentially dependent |
| Context | Separate when context burden is high | Combine when lightweight and fast |
| Reusability | Separate when useful for other teams | Consider combining when only used by this team |

## Skill vs Agent distinction

| Category | Skill | Agent |
|------|-------------|-----------------|
| Definition | Procedural knowledge + tool bundle | Specialist persona + behavioral principles |
| Location | .opencode/skills/ | .opencode/agents/ |
| Trigger | User request keyword matching | Explicit call through Task tool |
| Size | Small to large (workflow) | Small (role definition) |
| Purpose | "How to do it" | "Who does it" |

A skill is a **procedural guide** an agent references while doing work.
An agent is the **specialist role definition** that uses skills.

## Skill ↔ Agent connection methods

Three ways an agent can use a skill:

| Method | Implementation | Best fit |
|------|------|-----------|
| **Skill tool call** | Agent prompt explicitly instructs using the Skill tool with /skill-name | Skill is an independent workflow and user-invocable |
| **Inline in prompt** | Include skill content directly in the agent definition | Skill is short (50 lines or less) and specific to this agent |
| **Load reference** | Read skill references/ files as needed | Skill content is large and only conditionally needed |

Recommendation: use the Skill tool for reusable skills, inline for dedicated short skills, and reference loading for large content.
