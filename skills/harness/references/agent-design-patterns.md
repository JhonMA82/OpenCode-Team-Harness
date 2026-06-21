# Agent & Task Design Patterns

## Execution Modes: Task Parallel vs Sequential

Understand the key differences between two execution modes and choose the appropriate one.

### Parallel Task Invocation — Default Mode

Invoke multiple Tasks simultaneously with run_in_background: true for parallel execution. Each Task result is received and aggregated by the parent session.


[parent]
    ├── Task({ name: "agent-A", run_in_background: true })
    ├── Task({ name: "agent-B", run_in_background: true })
    └── Aggregate and integrate results


**Key Parameters:**
- Task({ name, prompt, model, tools, run_in_background })

**Characteristics:**
- Parallel execution reduces time
- Each Task executes in an independent context
- Result aggregation is handled by the parent

**Constraints:**
- No direct communication between Tasks (results are passed through the parent)
- Use sequential invocation for tasks with dependencies

### Sequential Task Invocation — Dependency Mode

Pass the result of the previous Task as input to the next Task, executing sequentially.


[parent]
    ├── Task({ name: "agent-1" }) → Result: _workspace/01_artifact.md
    └── Task({ name: "agent-2", input: 01 result }) → Result: _workspace/02_artifact.md


**Suitable when:**
- Each step depends on the output of the previous step
- Pipeline workflows

### Mode Selection Decision Tree


Can tasks run in parallel? (Independent work)
├── Yes → Parallel Task invocation (default)
│         Aggregate results in parent
│
└── No (sequential dependency) → Sequential Task invocation
                    Pass previous result as next input


> **Core Principle:** Independent tasks run in parallel; sequentially dependent tasks run sequentially.

---

## Task Architecture Types

### 1. Pipeline
Sequential workflow. The output of the previous Task is the input to the next.


[Analysis] → [Design] → [Implementation] → [Verification]


**Suitable when:** Each step strongly depends on the output of the previous step
**Example:** Novel writing — World-building → Characters → Plot → Writing → Editing
**Caution:** A bottleneck delays the entire pipeline. Design each step to be as independent as possible.

### 2. Fan-out/Fan-in
Parallel processing followed by result integration. Perform independent tasks simultaneously.


         ┌→ [Expert A] ─┐
[Distribute] → ├→ [Expert B] ─┼→ [Integrate]
         └→ [Expert C] ─┘


**Suitable when:** Different perspectives/domain analyses are needed for the same input
**Example:** Comprehensive research — Simultaneous investigation of official/media/community/background → Integrated report
**Caution:** The quality of the integration step determines overall quality.
**Task Pattern:** Must use parallel Task invocation.

### 3. Expert Pool
Select and invoke the appropriate expert based on the situation.


[Router] → { Expert A | Expert B | Expert C }


**Suitable when:** Different processing is needed depending on input type
**Example:** Code review — Invoke only the relevant expert among security/performance/architecture
**Caution:** Router classification accuracy is critical.
**Task Pattern:** Invoke Tasks sequentially or in parallel as needed.

### 4. Producer-Reviewer
Generation Task and verification Task operate as a pair.


[Producer] → [Reviewer] → (if issues) → [Producer] re-run


**Suitable when:** Output quality assurance is important and objective verification criteria exist
**Example:** Webtoon — Artist generates → Reviewer inspects → Re-generate problem panels
**Caution:** A maximum retry count (2–3 times) must be set to prevent infinite loops.

### 5. Supervisor
A central agent manages task state and dynamically distributes work to sub-agents.


         ┌→ [Worker A]
[Supervisor] ─┼→ [Worker B]    ← Supervisor monitors state and distributes dynamically
         └→ [Worker C]


**Suitable when:** Workload is variable or work distribution must be decided at runtime
**Example:** Large-scale code migration — Supervisor analyzes file list and assigns batches to workers
**Difference from Fan-out:** Fan-out distributes work in advance with fixed allocation; Supervisor adjusts dynamically based on progress

## Agent Invocation Methods

### 1. Task tool (explicit invocation)


Task({
  name: "explore",
  prompt: "Explore the codebase",
  model: "{provider/model}",
  tools: { edit: false, write: false }
})


### 2. @mention (direct invocation in message)


@explore Explore this codebase


- Invoke a subagent directly in a message
- Results are received in the parent session
- Suitable for simple tasks

## Agent Type Selection

Specify the agent type via the Task tool's name parameter.

### Built-in Types

| Type | Tool Access | Suitable For |
|------|----------|-----------|
| general-purpose | Full (including WebSearch, WebFetch) | Web research, general tasks |
| Explore | Read-only (no Edit/Write) | Codebase exploration, analysis |
| Plan | Read-only (no Edit/Write) | Architecture design, planning |

### Custom Types

Define an agent at `.opencode/agents/{{AGENT_NAME}}.md` and invoke it with name: "{{AGENT_NAME}}". Custom agents have full tool access.

### Selection Criteria

| Situation | Recommended | Reason |
|------|------|------|
| Complex role reused across multiple sessions | **Custom type** (.opencode/agents/) | Manage persona and working principles as files |
| Simple research/gathering where prompt alone suffices | **general-purpose** + detailed prompt | No agent file needed, instructions in prompt |
| Only code reading needed (analysis/review) | **Explore** | Prevents accidental file modification |
| Only design/planning needed | **Plan** | Focus on analysis, prevent code changes |
| Implementation tasks requiring file modifications | **Custom type** | Full tool access + specialized instructions |

**Principle:** All agents must be defined as `.opencode/agents/{{AGENT_NAME}}.md` files. Even for built-in types, create an agent definition file specifying role and principles. Files must exist to be reusable in subsequent sessions.

**Model configuration:** The `model` parameter is specified in `provider/model` format. Use models configured in the user's OpenCode settings (config). Examples: `anthropic/claude-sonnet-4-20250514`, `openrouter/minimax/minimax-m2.7`, etc.

## Agent Definition: Markdown vs JSON

### Markdown (`.opencode/agents/{{AGENT_NAME}}.md`)

markdown
---
description: "1-2 sentence role description. List trigger keywords."
mode: subagent
model: openrouter/minimax/minimax-m2.7
tools:
  write: false
  bash: false
---

# Agent Name — One-line Role Summary

You are a [role] expert in [domain].

## Core Role
1. Role 1
2. Role 2

## Working Principles
- Principle 1
- Principle 2

## Input/Output Protocol
- Input: [what is received and from where]
- Output: [what is written and where]
- Format: [file format, structure]

## Error Handling
- [Behavior on failure]
- [Behavior on timeout]


### JSON (opencode.json)

json
{
  "agent": {
    "my-agent": {
      "description": "Role description",
      "mode": "subagent",
      "model": "openrouter/minimax/minimax-m2.7",
      "tools": {
        "write": false,
        "bash": false
      }
    }
  }
}


### Selection Criteria
- **Markdown**: When the agent definition is complex and the system prompt is long
- **JSON**: For simple configuration changes or built-in agent overrides

## Agent Definition Structure

markdown
---
name: agent-name
description: "1-2 sentence role description. List trigger keywords."
---

# Agent Name — One-line Role Summary

You are a [role] expert in [domain].

## Core Role
1. Role 1
2. Role 2

## Working Principles
- Principle 1
- Principle 2

## Input/Output Protocol
- Input: [what is received and from where]
- Output: [what is written and where]
- Format: [file format, structure]

## Error Handling
- [Behavior on failure]
- [Behavior on timeout]


## Agent Separation Criteria

| Criterion | Separate | Merge |
|------|------|------|
| Specialization | Separate if domains differ | Merge if domains overlap |
| Parallelism | Separate if independently executable | Consider merging if sequentially dependent |
| Context | Separate if context burden is heavy | Merge if lightweight and fast |
| Reusability | Separate if used by other teams | Consider merging if only used by this team |

## Skill vs Agent Distinction

| Aspect | Skill | Agent |
|------|-------------|-----------------|
| Definition | Procedural knowledge + tool bundle | Expert persona + behavioral principles |
| Location | .opencode/skills/ | .opencode/agents/ |
| Trigger | User request keyword matching | Explicit invocation via Task tool |
| Size | Small to large (workflow) | Small (role definition) |
| Purpose | "How to do it" | "Who does it" |

Skills are **procedural guides** that agents reference when performing work.
Agents are **expert role definitions** that utilize skills.

## Skill ↔ Agent Connection Methods

Three ways agents utilize skills:

| Method | Implementation | Suitable When |
|------|------|-----------|
| **Skill tool invocation** | Specify /skill-name invocation as a Skill tool in agent prompt | Skill is an independent workflow and user-invokable |
| **Inline in prompt** | Include skill content directly within agent definition | Skill is short (under 50 lines) and exclusive to this agent |
| **Reference loading** | Load skill's references/ files as needed via Read | Skill content is large and only conditionally needed |

Recommendation: Use Skill tool for high reusability, inline for exclusive use, reference loading for large content.
