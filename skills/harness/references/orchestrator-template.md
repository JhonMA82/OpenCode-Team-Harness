# Orchestrator Skill Template

An orchestrator is a higher-level skill that coordinates the entire workflow. Two templates are provided depending on the execution mode.

---

## Template A: Parallel Task Invocation (Default)

Invoke Tasks simultaneously with run_in_background: true for parallel execution.

markdown
---
name: {domain}-orchestrator
description: "Orchestrator that coordinates {domain} workflows. {trigger keywords}."
---

# {Domain} Orchestrator

An integration skill that coordinates {domain} Tasks to produce {final output}.

## Execution Mode: Parallel Task Invocation

## Agent Configuration

| Agent | Role | Skill | Output |
|---------|------|------|------|
| {agent-1} | {role} | {skill} | _workspace/01_{agent-1}.md |
| {agent-2} | {role} | {skill} | _workspace/02_{agent-2}.md |
| ... | | | |

## Workflow

### Phase 1: Preparation
1. Analyze user input — {what to identify}
2. Create _workspace/ in working directory
3. Save input data to _workspace/00_input/

### Phase 2: Parallel Task Execution

> **Model configuration:** The `model` parameter is specified in `provider/model` format. Use models configured in the user's OpenCode settings (config). Examples: `anthropic/claude-sonnet-4-20250514`, `openrouter/minimax/minimax-m2.7`, etc.

Invoke all Tasks simultaneously:

Task({ name: "agent-1", prompt: "{role description and task instructions}", model: "{provider/model}", run_in_background: true })
Task({ name: "agent-2", prompt: "{role description and task instructions}", model: "{provider/model}", run_in_background: true })
Task({ name: "agent-3", prompt: "{role description and task instructions}", model: "{provider/model}", run_in_background: true })


Wait for all Tasks to complete → Aggregate results

**Output storage:**

| Agent | Output Path |
|---------|----------|
| {agent-1} | _workspace/01_{agent-1}_{artifact}.md |
| {agent-2} | _workspace/02_{agent-2}_{artifact}.md |

### Phase 3: Result Aggregation and Integration
1. Read all Task results
2. {Integration/verification logic}
3. Generate final output: {output-path}/{filename}

### Phase 4: Cleanup
1. Preserve _workspace/ directory (do not delete intermediate outputs — retained for post-verification and audit trails)
2. Report result summary to user

## Data Flow


[parent]
    ├── Task({ name: "agent-1", run_in_background: true })
    ├── Task({ name: "agent-2", run_in_background: true })
    │
    ├── Receive result ( artifact-1.md )
    └── Receive result ( artifact-2.md )
              │
              └────── Read ───────┘
                          │
                    [parent: Integration]
                          │
                    Final Output


## Error Handling

| Situation | Strategy |
|------|------|
| 1 Task fails | Retry once. On second failure, proceed without that result; note omission in report |
| Majority of Tasks fail | Notify user and confirm whether to proceed |
| Timeout | Use partial results collected so far |
| Data conflict | Cite sources and include both; do not delete |

## Test Scenarios

### Normal Flow
1. User provides {input}
2. Phase 1 produces {analysis result}
3. Phase 2 executes {N} Tasks in parallel
4. Phase 3 integrates outputs to generate final result
5. Expected result: {output-path}/{filename} generated

### Error Flow
1. {agent-2} fails in Phase 2
2. Still fails after 1 retry
3. Phase 3 proceeds with remaining results without {agent-2}
4. Final report states "{agent-2} area not collected"


---

## Template B: Sequential Task Invocation

Pass the result of the previous Task as input to the next Task, executing sequentially.

markdown
---
name: {domain}-orchestrator
description: "Orchestrator that coordinates {domain} workflows. {trigger keywords}."
---

# {Domain} Orchestrator

An integration skill that sequentially coordinates {domain} Tasks to produce {final output}.

## Execution Mode: Sequential Task Invocation

## Agent Configuration

| Agent | Role | Skill | Output |
|---------|------|------|------|
| {agent-1} | {role} | {skill} | _workspace/01_{agent-1}.md |
| {agent-2} | {role} | {skill} | _workspace/02_{agent-2}.md |
| ... | | | |

## Workflow

### Phase 1: Preparation
1. Analyze user input — {what to identify}
2. Create _workspace/ in working directory
3. Save input data to _workspace/00_input/

### Phase 2: Sequential Task Execution

Pass previous Task result as input to the next Task:


Phase 2-1: Task({ name: "agent-1", prompt: "{role description}", model: "{provider/model}" })
→ Result: _workspace/01_agent-1.md

Phase 2-2: Task({ name: "agent-2", prompt: "{role description}. Input: refer to _workspace/01_agent-1.md", model: "{provider/model}" })
→ Result: _workspace/02_agent-2.md

Phase 2-3: Task({ name: "agent-3", prompt: "{role description}. Input: refer to _workspace/02_agent-2.md", model: "{provider/model}" })
→ Result: _workspace/03_agent-3.md


### Phase 3: Final Integration
1. Read final Task result
2. {Final integration/verification logic}
3. Generate final output: {output-path}/{filename}

### Phase 4: Cleanup
1. Preserve _workspace/ directory
2. Report result summary to user

## Data Flow


[parent]
    │
    ├── Task({ name: "agent-1" }) → artifact-1.md
    │                                    │
    │                                    ↓
    ├── Task({ name: "agent-2" }) ← artifact-1.md input
    │                                    │
    │                                    ↓
    ├── Task({ name: "agent-3" }) ← artifact-2.md input
    │                                    │
    └────────────────────────────────────┘
                        │
                  [parent: Final Integration]
                        │
                  Final Output


## Error Handling

| Situation | Strategy |
|------|------|
| 1 Task fails | Retry once. On second failure, proceed without that result; note omission in report |
| Timeout | Use partial results collected so far |
| Data conflict | Cite sources and include both; do not delete |

## Test Scenarios

### Normal Flow
1. User provides {input}
2. Phase 1 produces {analysis result}
3. Phase 2 executes {N} Tasks sequentially
4. Phase 3 generates final result
5. Expected result: {output-path}/{filename} generated

### Error Flow
1. {agent-2} fails in Phase 2-2
2. Still fails after 1 retry
3. Skip {agent-2} and proceed to Phase 2-3
4. Final report states "{agent-2} not executed"


---

## Writing Principles

1. **Specify execution mode first** — State "Parallel Task Invocation" or "Sequential Task Invocation" at the top of the orchestrator
2. **Be specific with Task parameters** — name, prompt, model, run_in_background
3. **Use absolute file paths** — No relative paths; clear paths based on _workspace/
4. **Specify inter-Phase dependencies** — Which Phase depends on which Phase's results
5. **Keep error handling realistic** — Don't assume "everything will succeed"
6. **Test scenarios are mandatory** — At least 1 normal flow + 1 error flow

## Orchestrator Reference

Basic structure of a fan-out/fan-in pattern orchestrator:
Preparation → Invoke N Tasks in parallel → Aggregate results → Integrate → Cleanup.
See the research team example in references/team-examples.md.
