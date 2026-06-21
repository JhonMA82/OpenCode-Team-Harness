# Orchestrator Skill Template

The orchestrator is the top-level skill that coordinates the entire workflow. This file provides two templates based on execution mode.

---

## Template A: Parallel Task Calls (default)

Call Tasks concurrently with `run_in_background: true` for parallel execution.

markdown
---
name: {domain}-orchestrator
description: "Orchestrator that coordinates the {도메인} workflow. {트리거 키워드}."
---

# {Domain} Orchestrator

An integration skill that coordinates Tasks for {도메인} and creates {최종 산출물}.

## Execution mode: Parallel Task calls

## Agent composition

| Agent | Role | Skill | Output |
|---------|------|------|------|
| {agent-1} | {역할} | {skill} | _workspace/01_{agent-1}.md |
| {agent-2} | {역할} | {skill} | _workspace/02_{agent-2}.md |
| ... | | | |

## Workflow

### Phase 1: Preparation
1. Analyze user input — {무엇을 파악하는지}
2. Create _workspace/ in the working directory
3. Save input data under _workspace/00_input/

### Phase 2: Run parallel Tasks

> **Model setting:** Set the `model` parameter in `provider/model` format. Use a model configured in the user's OpenCode config. Examples: `anthropic/claude-sonnet-4-20250514`, `openrouter/minimax/minimax-m2.7`.

Call all Tasks concurrently:

Task({ name: "agent-1", prompt: "{역할 설명 및 작업 지시}", model: "{provider/model}", run_in_background: true })
Task({ name: "agent-2", prompt: "{역할 설명 및 작업 지시}", model: "{provider/model}", run_in_background: true })
Task({ name: "agent-3", prompt: "{역할 설명 및 작업 지시}", model: "{provider/model}", run_in_background: true })


Wait for all Tasks to finish → collect results

**Artifact storage:**

| Agent | Output path |
|---------|----------|
| {agent-1} | _workspace/01_{agent-1}_{artifact}.md |
| {agent-2} | _workspace/02_{agent-2}_{artifact}.md |

### Phase 3: Collect and integrate results
1. Read all Task results.
2. {통합/검증 로직}
3. Create final artifact: {output-path}/{filename}

### Phase 4: Cleanup
1. Preserve the _workspace/ directory (do not delete intermediate artifacts — they are used for post-hoc verification and audit trail).
2. Report a summary of results to the user.

## Data flow

[parent]
    ├── Task({ name: "agent-1", run_in_background: true })
    ├── Task({ name: "agent-2", run_in_background: true })
    │
    ├── receive result ( artifact-1.md )
    └── receive result ( artifact-2.md )
              │
              └────── Read ───────┘
                          │
                    [parent: integration]
                          │
                    final artifact


## Error handling

| Situation | Strategy |
|------|------|
| One Task fails | Retry once. If it fails again, proceed without that result and note the omission in the report. |
| Majority of Tasks fail | Inform the user and confirm whether to proceed. |
| Timeout | Use the partial results collected so far. |
| Data conflict | Cite sources side by side; do not delete conflicting data. |

## Test Scenarios

### Happy path
1. User provides {입력}.
2. Phase 1 produces {분석 결과}.
3. Phase 2 runs {N} Tasks in parallel.
4. Phase 3 integrates artifacts and creates the final result.
5. Expected result: {output-path}/{filename} is created.

### Error path
1. {agent-2} fails in Phase 2.
2. It still fails after one retry.
3. Phase 3 proceeds with the remaining results and without {agent-2}'s result.
4. Final report states "{agent-2} area not collected."


---

## Template B: Sequential Task Calls

Pass each previous Task result into the next Task and execute sequentially.

markdown
---
name: {domain}-orchestrator
description: "Orchestrator that coordinates the {도메인} workflow. {트리거 키워드}."
---

# {Domain} Orchestrator

An integration skill that sequentially coordinates Tasks for {도메인} and creates {최종 산출물}.

## Execution mode: Sequential Task calls

## Agent composition

| Agent | Role | Skill | Output |
|---------|------|------|------|
| {agent-1} | {역할} | {skill} | _workspace/01_{agent-1}.md |
| {agent-2} | {역할} | {skill} | _workspace/02_{agent-2}.md |
| ... | | | |

## Workflow

### Phase 1: Preparation
1. Analyze user input — {무엇을 파악하는지}
2. Create _workspace/ in the working directory
3. Save input data under _workspace/00_input/

### Phase 2: Run sequential Tasks

Pass the previous Task result as input to the next Task:

Phase 2-1: Task({ name: "agent-1", prompt: "{역할 설명}", model: "{provider/model}" })
→ result: _workspace/01_agent-1.md

Phase 2-2: Task({ name: "agent-2", prompt: "{역할 설명}. Input: refer to _workspace/01_agent-1.md", model: "{provider/model}" })
→ result: _workspace/02_agent-2.md

Phase 2-3: Task({ name: "agent-3", prompt: "{역할 설명}. Input: refer to _workspace/02_agent-2.md", model: "{provider/model}" })
→ result: _workspace/03_agent-3.md


### Phase 3: Final integration
1. Read the final Task result.
2. {최종 통합/검증 로직}
3. Create final artifact: {output-path}/{filename}

### Phase 4: Cleanup
1. Preserve the _workspace/ directory.
2. Report a summary of results to the user.

## Data flow

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
                  [parent: final integration]
                        │
                  final artifact


## Error handling

| Situation | Strategy |
|------|------|
| One Task fails | Retry once. If it fails again, proceed without that result and note the omission in the report. |
| Timeout | Use the partial results collected so far. |
| Data conflict | Cite sources side by side; do not delete conflicting data. |

## Test Scenarios

### Happy path
1. User provides {입력}.
2. Phase 1 produces {분석 결과}.
3. Phase 2 runs {N} Tasks sequentially.
4. Phase 3 creates the final result.
5. Expected result: {output-path}/{filename} is created.

### Error path
1. {agent-2} fails in Phase 2-2.
2. It still fails after one retry.
3. Skip {agent-2} and proceed to Phase 2-3.
4. Final report states "{agent-2} not executed."


---

## Writing principles

1. **State the execution mode first** — at the top of the orchestrator, explicitly state "Parallel Task calls" or "Sequential Task calls."
2. **Make Task parameters concrete** — name, prompt, model, run_in_background.
3. **Use absolute file paths** — avoid relative ambiguity; make _workspace/ paths explicit.
4. **State dependencies between Phases** — which Phase depends on which previous Phase output.
5. **Handle errors realistically** — do not assume everything succeeds.
6. **Test scenarios are required** — at least one happy path and one error path.

## Real orchestrator reference

Basic structure for a fan-out/fan-in orchestrator:
prepare → N parallel Task calls → collect results → integrate → cleanup.
See the research team example in references/team-examples.md.
