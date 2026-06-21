# Task & Agent Examples

---

## Example 1: Research Team (Parallel Task Invocation)

### Architecture: Fan-out/Fan-in
### Execution Mode: Parallel Task Invocation


[parent/orchestrator]
    ├── Task({ name: "official", run_in_background: true })
    ├── Task({ name: "media", run_in_background: true })
    ├── Task({ name: "community", run_in_background: true })
    ├── Task({ name: "background", run_in_background: true })
    ├── Wait for all Tasks to complete
    └── Aggregate results and generate comprehensive report


### Agent Configuration

| Agent | Role | Output |
|---------|------|------|
| official | Official documents/blogs | _workspace/01_official.md |
| media | Media/investment | _workspace/02_media.md |
| community | Community/social media | _workspace/03_community.md |
| background | Background/competition/academic | _workspace/04_background.md |
| (parent = orchestrator) | Integrated report | comprehensive_report.md |

> Research agents use the general-purpose built-in type, but must be defined as `.opencode/agents/{{AGENT_NAME}}.md` files.

### Orchestrator Workflow


Phase 1: Preparation
  - Analyze user input (identify topic, research mode)
  - Create _workspace/

> **Model configuration:** The `model` parameter is specified in `provider/model` format. Use models configured in the user's OpenCode settings (config). Examples: `anthropic/claude-sonnet-4-20250514`, `openrouter/minimax/minimax-m2.7`, etc.

Phase 2: Parallel Research Execution
  Task({ name: "official", prompt: "Research official channels...", model: "{provider/model}", run_in_background: true })
  Task({ name: "media", prompt: "Research media/investment trends...", model: "{provider/model}", run_in_background: true })
  Task({ name: "community", prompt: "Research community reactions...", model: "{provider/model}", run_in_background: true })
  Task({ name: "background", prompt: "Research background/competitive landscape...", model: "{provider/model}", run_in_background: true })

Phase 3: Result Aggregation
  - Read 4 outputs
  - Generate comprehensive report
  - Cite sources and include both for conflicting information

Phase 4: Cleanup
  - Preserve _workspace/ (for post-verification and audit trails)


---

## Example 2: SF Novel Writing Team (Parallel + Sequential Tasks)

### Architecture: Pipeline + Fan-out
### Execution Mode: Parallel → Sequential → Parallel → Sequential


Phase 1 (parallel): worldbuilder + character-designer + plot-architect
Phase 2 (sequential): prose-stylist (writing)
Phase 3 (parallel): science-consultant + continuity-manager (review)
Phase 4 (sequential): prose-stylist (final revision)


### Agent Configuration

| Agent | Role | Skill |
|---------|------|------|
| worldbuilder | World-building | world-setting |
| character-designer | Character design | character-profile |
| plot-architect | Plot structure | outline |
| prose-stylist | Writing + style editing | write-scene, review-chapter |
| science-consultant | Science verification | science-check |
| continuity-manager | Consistency verification | consistency-check |

### Agent File Example: worldbuilder.md

markdown
---
name: worldbuilder
description: "Expert in building SF novel worlds. Designs physics laws, social structures, technology levels, and history."
---

# Worldbuilder — SF World Design Expert

You are an expert in SF novel world design.

## Core Role
1. Define the world's physics laws and technology level
2. Design social structure, political system, economic system
3. Establish historical context and current conflict structure
4. Describe environment and atmosphere for each location

## Working Principles
- Internal consistency is the top priority — no contradictions between settings
- Use chain questioning "If this technology exists, then what?" to infer ripple effects
- World-building serves the story — avoid excessive settings that hinder the plot

## Input/Output Protocol
- Input: User's world concept, genre requirements
- Output: _workspace/01_worldbuilder_setting.md
- Format: Markdown. Sections by (physics/society/technology/history/locations)

## Error Handling
- If concept is vague, propose 3 directions and request selection
- When scientific errors are found, present alternatives alongside


### Detailed Workflow


Phase 1: Parallel Task Invocation
  Task({ name: "worldbuilder", run_in_background: true })
  Task({ name: "character-designer", run_in_background: true })
  Task({ name: "plot-architect", run_in_background: true })
  → Save outputs to _workspace/

Phase 2: Sequential Task Invocation
  Task({ name: "prose-stylist", prompt: "Read 3 outputs from _workspace/ and write" })
  → _workspace/02_prose_draft.md

Phase 3: Parallel Task Invocation
  Task({ name: "science-consultant", run_in_background: true })
  Task({ name: "continuity-manager", run_in_background: true })
  → Review draft, save reviews to _workspace/

Phase 4: Sequential Task Invocation
  Task({ name: "prose-stylist", prompt: "Apply review results for final revision" })
  → Final output


---

## Example 3: Webtoon Production Team (Sequential Tasks - Producer-Reviewer)

### Architecture: Producer-Reviewer
### Execution Mode: Sequential Task Invocation

> With only 2 agents in the producer-reviewer pattern and sequential result passing as the key concern, sequential Tasks are appropriate.


Phase 1: Task(webtoon-artist) → Generate panels
Phase 2: Task(webtoon-reviewer) → Quality review
Phase 3: Task(webtoon-artist) → Regenerate problem panels (max 2 times)


### Agent Configuration

| Agent | Role | Skill |
|---------|------|------|
| webtoon-artist | Panel image generation | generate-webtoon |
| webtoon-reviewer | Quality review | review-webtoon, fix-webtoon-panel |

### Agent File Example: webtoon-reviewer.md

markdown
---
name: webtoon-reviewer
description: "Expert in reviewing webtoon panel quality. Evaluates composition, character consistency, text readability, and direction."
---

# Webtoon Reviewer — Webtoon Quality Review Expert

You are an expert in reviewing webtoon panel quality.

## Core Role
1. Evaluate composition and visual completeness of each panel
2. Verify character appearance consistency across panels
3. Evaluate speech bubble text readability and placement
4. Review overall episode direction flow and pacing

## Working Principles
- Judge clearly using 3 levels: PASS/FIX/REDO
- FIX for cases fixable with partial modification; REDO for cases requiring full regeneration
- Judge by objective criteria (consistency, readability, composition), not subjective taste

## Input/Output Protocol
- Input: Panel images in _workspace/panels/ directory
- Output: _workspace/review_report.md

## Error Handling
- If image fails to load, judge that panel as REDO
- Panels still REDO after 2 regenerations: PASS with warning


### Error Handling


Retry policy:
- REDO panels → Request regeneration from artist (include specific fix instructions)
- Force PASS after max 2 loops
- If 50%+ of total panels are REDO, suggest prompt revision to user


---

## Example 4: Code Review Team (Parallel Task Invocation)

### Architecture: Fan-out/Fan-in
### Execution Mode: Parallel Task Invocation

> Code review allows faster review by having reviewers with different perspectives analyze independently in parallel.


[parent] → Parallel Task Invocation
    ├── Task({ name: "security-reviewer", run_in_background: true })
    ├── Task({ name: "performance-reviewer", run_in_background: true })
    └── Task({ name: "test-reviewer", run_in_background: true })
    → Reviewers return results to parent
    → Parent aggregates results


### Team Communication Pattern


security → performance: "SQL injection possible in this query, please also check from performance perspective"
performance → test: "N+1 query found, check if related tests exist"
test → security: "No auth module tests, your opinion on priority from security perspective?"
→ All results are aggregated by parent


Key: Each reviewer analyzes independently, and results are aggregated by the parent for a comprehensive review.

---

## Example 5: Supervisor Pattern — Code Migration Team (Sequential Tasks)

### Architecture: Supervisor
### Execution Mode: Sequential Task Invocation


[supervisor] → Sequential Task Invocation
    ├→ Task({ name: "migrator-1" }) (batch A)
    ├→ Task({ name: "migrator-2" }) (batch B)
    └→ Task({ name: "migrator-3" }) (batch C)
    → Each migrator processes assigned file batch
    → Supervisor performs final integration


### Agent Configuration

| Agent | Role |
|---------|------|
| (parent = migration-supervisor) | File analysis, batch distribution, progress management |
| migrator-1~3 | Migrate assigned file batches |

### Supervisor's Dynamic Distribution Logic


1. Collect complete target file list
2. Estimate complexity (file size, import count, dependencies)
3. Split file batch into 3 groups
4. Assign batches to each migrator via sequential Task invocation
5. Aggregate results after each migrator completes
6. All work complete → Run integration tests


---

## Output Pattern Summary

### Agent Definition Files
Location: project/.opencode/agents/{agent-name}.md
Required sections: Core Role, Working Principles, Input/Output Protocol, Error Handling

### Skill File Structure
Location: project/.opencode/skills/{skill-name}/SKILL.md (recommended project level)
Portable mirror: project/.agents/skills/{skill-name}/SKILL.md (optional)

### Integration Skill (Orchestrator)
A higher-level skill that coordinates the entire team. Defines agent configuration and workflow for each scenario.
Template: See references/orchestrator-template.md.
**Execution mode must be specified** — Parallel Tasks or Sequential Tasks.
