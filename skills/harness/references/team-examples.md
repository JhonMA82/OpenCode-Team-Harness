# Task & Agent Examples

---

## Example 1: Research Team (Parallel Task Calls)

### Architecture: Fan-out/Fan-in
### Execution mode: Parallel Task calls


[parent/orchestrator]
    ├── Task({ name: "official", run_in_background: true })
    ├── Task({ name: "media", run_in_background: true })
    ├── Task({ name: "community", run_in_background: true })
    ├── Task({ name: "background", run_in_background: true })
    ├── wait for all Tasks to finish
    └── collect results and create integrated report


### Agent composition

| Agent | Role | Output |
|---------|------|------|
| official | Official docs/blogs | _workspace/01_official.md |
| media | Media/investment | _workspace/02_media.md |
| community | Community/SNS | _workspace/03_community.md |
| background | Background/competition/academic | _workspace/04_background.md |
| (parent = orchestrator) | Integrated report | integrated-report.md |

> Research agents may use the `general-purpose` built-in type, but they MUST be defined in `.opencode/agents/{{AGENT_NAME}}.md` files.

### Orchestrator workflow


Phase 1: Preparation
  - Analyze user input (identify topic and research mode)
  - Create _workspace/

> **Model setting:** Set the `model` parameter in `provider/model` format. Use a model configured in the user's OpenCode config. Examples: `anthropic/claude-sonnet-4-20250514`, `openrouter/minimax/minimax-m2.7`.

Phase 2: Run parallel research
  Task({ name: "official", prompt: "Research official channels...", model: "{provider/model}", run_in_background: true })
  Task({ name: "media", prompt: "Research media/investment trends...", model: "{provider/model}", run_in_background: true })
  Task({ name: "community", prompt: "Research community reactions...", model: "{provider/model}", run_in_background: true })
  Task({ name: "background", prompt: "Research background/competitive landscape...", model: "{provider/model}", run_in_background: true })

Phase 3: Collect results
  - Read 4 artifacts
  - Create integrated report
  - Cite sources alongside conflicting information

Phase 4: Cleanup
  - Preserve _workspace/ for post-hoc verification and audit trail


---

## Example 2: SF Novel Writing Team (Parallel + Sequential Tasks)

### Architecture: Pipeline + Fan-out
### Execution mode: Parallel → sequential → parallel → sequential Tasks


Phase 1 (parallel): worldbuilder + character-designer + plot-architect
Phase 2 (sequential): prose-stylist (drafting)
Phase 3 (parallel): science-consultant + continuity-manager (review)
Phase 4 (sequential): prose-stylist (final revision)


### Agent composition

| Agent | Role | Skill |
|---------|------|------|
| worldbuilder | Worldbuilding | world-setting |
| character-designer | Character design | character-profile |
| plot-architect | Plot structure | outline |
| prose-stylist | Prose editing + drafting | write-scene, review-chapter |
| science-consultant | Science validation | science-check |
| continuity-manager | Consistency validation | consistency-check |

### Agent file example: worldbuilder.md

markdown
---
name: worldbuilder
description: "Specialist who builds the world of an SF novel. Designs physical laws, social structures, technology levels, and history."
---

# Worldbuilder — SF Worldbuilding Specialist

You are an SF novel worldbuilding specialist.

## Core Responsibilities
1. Define the world's physical laws and technology level.
2. Design social structures, political systems, and economic systems.
3. Establish historical context and current conflict structures.
4. Describe environments and atmosphere by location.

## Working Principles
- Prioritize internal consistency — settings must not contradict each other.
- Infer the world's cascading effects through repeated "What if this technology exists?" questions.
- Worldbuilding serves the story — avoid excessive settings that obstruct the plot.

## Input/Output Protocol
- Input: user's worldbuilding concept and genre requirements.
- Output: _workspace/01_worldbuilder_setting.md
- Format: Markdown, sectioned by physics/society/technology/history/locations.

## Error Handling
- If the concept is ambiguous, propose 3 directions and ask the user to choose.
- When scientific errors are found, suggest alternatives as well.


### Detailed workflow


Phase 1: Parallel Task calls
  Task({ name: "worldbuilder", run_in_background: true })
  Task({ name: "character-designer", run_in_background: true })
  Task({ name: "plot-architect", run_in_background: true })
  → save artifacts in _workspace/

Phase 2: Sequential Task call
  Task({ name: "prose-stylist", prompt: "Read the 3 artifacts in _workspace/ and write the draft" })
  → _workspace/02_prose_draft.md

Phase 3: Parallel Task calls
  Task({ name: "science-consultant", run_in_background: true })
  Task({ name: "continuity-manager", run_in_background: true })
  → review draft and save reviews in _workspace/

Phase 4: Sequential Task call
  Task({ name: "prose-stylist", prompt: "Apply the review results and make final revisions" })
  → final artifact


---

## Example 3: Webtoon Production Team (Sequential Tasks - Producer-Reviewer)

### Architecture: Producer-Reviewer
### Execution mode: Sequential Task calls

> In the producer-reviewer pattern, there are only two agents and sequential result handoff is central, so sequential Tasks fit best.


Phase 1: Task(webtoon-artist) → generate panels
Phase 2: Task(webtoon-reviewer) → review
Phase 3: Task(webtoon-artist) → regenerate problem panels (up to 2 times)


### Agent composition

| Agent | Role | Skill |
|---------|------|------|
| webtoon-artist | Panel image generation | generate-webtoon |
| webtoon-reviewer | Quality review | review-webtoon, fix-webtoon-panel |

### Agent file example: webtoon-reviewer.md

markdown
---
name: webtoon-reviewer
description: "Specialist who reviews webtoon panel quality. Evaluates composition, character consistency, text readability, and direction."
---

# Webtoon Reviewer — Webtoon Quality Review Specialist

You are a specialist who reviews webtoon panel quality.

## Core Responsibilities
1. Evaluate each panel's composition and visual polish.
2. Validate character appearance consistency across panels.
3. Evaluate speech bubble text readability and placement.
4. Review the direction flow and pacing of the full episode.

## Working Principles
- Judge clearly using three levels: PASS/FIX/REDO.
- FIX applies when a partial correction can solve the issue; REDO requires full regeneration.
- Judge by objective criteria (consistency, readability, composition), not subjective taste.

## Input/Output Protocol
- Input: panel images in the _workspace/panels/ directory.
- Output: _workspace/review_report.md

## Error Handling
- If an image fails to load, mark that panel as REDO.
- If a panel is still REDO after two regenerations, pass it with a warning.


### Error handling


Retry policy:
- REDO panels → ask artist to regenerate them (include concrete correction instructions).
- Force PASS after at most 2 loops.
- If 50% or more of all panels are REDO, suggest prompt revision to the user.


---

## Example 4: Code Review Team (Parallel Task Calls)

### Architecture: Fan-out/Fan-in
### Execution mode: Parallel Task calls

> Code review can be faster when reviewers with different perspectives analyze independently in parallel.


[parent] → parallel Task calls
    ├── Task({ name: "security-reviewer", run_in_background: true })
    ├── Task({ name: "performance-reviewer", run_in_background: true })
    └── Task({ name: "test-reviewer", run_in_background: true })
    → reviewers return results to parent
    → parent synthesizes results


### Team communication pattern


security → performance: "This SQL query may be injectable; please also check the performance angle"
performance → test: "Found an N+1 query; please confirm whether related tests exist"
test → security: "No auth module tests; any priority opinion from a security perspective?"
→ all results are collected by parent


Key point: each reviewer analyzes independently, and parent collects and synthesizes the results.

---

## Example 5: Supervisor Pattern — Code Migration Team (Sequential Tasks)

### Architecture: Supervisor
### Execution mode: Sequential Task calls


[supervisor] → sequential Task calls
    ├→ Task({ name: "migrator-1" }) (batch A)
    ├→ Task({ name: "migrator-2" }) (batch B)
    └→ Task({ name: "migrator-3" }) (batch C)
    → each migrator processes its assigned file batch
    → supervisor performs final integration


### Agent composition

| Agent | Role |
|---------|------|
| (parent = migration-supervisor) | Analyze files, distribute batches, manage progress |
| migrator-1~3 | Migrate assigned file batches |

### Supervisor dynamic distribution logic


1. Collect the full target file list.
2. Estimate complexity (file size, import count, dependencies).
3. Split file batches into 3 groups.
4. Assign each batch to a migrator through sequential Task calls.
5. Collect results after each migrator completes.
6. All work complete → run integration tests.


---

## Output pattern summary

### Agent definition files
Location: project/.opencode/agents/{agent-name}.md
Required sections: core responsibilities, working principles, input/output protocol, error handling

### Skill file structure
Location: project/.opencode/skills/{skill-name}/SKILL.md (recommended project level)
Portable mirror: project/.agents/skills/{skill-name}/SKILL.md (optional)

### Integration skill (orchestrator)
A top-level skill that coordinates the whole team. Defines scenario-specific agent composition and workflow.
Template: see references/orchestrator-template.md.
**Execution mode MUST be explicit** — parallel Task or sequential Task.
