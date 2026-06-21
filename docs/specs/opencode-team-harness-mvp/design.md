# Design: OpenCode Team Harness — MVP

## Technical Approach

Two-phase implementation: (1) strip all Claude Code identity from the fork's public surfaces, (2) add OpenCode-native agent/skill generation and eval infrastructure. The harness repo stays as the meta-skill + docs + evals — it generates outputs for consumer projects, not for itself. Generation is deterministic from templates (no AI inference, per AG-8 and CC-3).

---

## Architecture Decisions

### Decision: Harness repo vs generated output

| Option | Tradeoff | Decision |
|--------|----------|----------|
| Repo IS the generated output | Self-referential, blurs what serves what | **REJECTED** |
| Repo = meta-skill only, consumer output is documented elsewhere | Clear boundary, `skills/harness/` is source of truth | **SELECTED** |

**Rationale**: The harness is a generator. This repo holds the meta-skill (`skills/harness/`) that, when invoked by a user, generates `AGENTS.md`, `.opencode/agents/`, `.opencode/skills/` into the *consumer's project*. Example output lives in `docs/agentic-development/` as reference. Avoiding self-scaffolding (a harness generating its own harness config) keeps the dependency graph acyclic.

### Decision: `.claude-plugin/` removal

| Option | Tradeoff | Decision |
|--------|----------|----------|
| Modify plugin.json for OpenCode (IR-4) | Preserves metadata but directory is Claude-only concept | **REJECTED** |
| Remove entire directory (IR-5) | Clean break; metadata preserved in SKILL.md frontmatter | **SELECTED** |

**Rationale**: IR-5 (remove) takes precedence over IR-4 (modify). OpenCode has no plugin marketplace. The metadata (name, version, author, license) is already in `skills/harness/SKILL.md` frontmatter and repo-level documentation.

### Decision: Two output paths

| Option | Tradeoff | Decision |
|--------|----------|----------|
| `.opencode/skills/` only | Clean but breaks `.agents/`-compatible consumers | **REJECTED** |
| Both paths, `.opencode/` primary | Supports migration without lock-in | **SELECTED** |

**Rationale**: Per SG-3 (SHOULD) and IR-8 (MUST NOT remove primary). The generation logic always writes to `.opencode/skills/{{SKILL_NAME}}/SKILL.md`. If portable compat is enabled, a deterministic mirror/copy step writes to `.agents/skills/{{SKILL_NAME}}/SKILL.md`. The primary is always authoritative — `.agents/skills/` is a derived artifact.

### Decision: SKILL.md Claude compat line

**Choice**: Remove line 13 ("Claude 호환 경로도 지원한다") from `skills/harness/SKILL.md`.
**Rationale**: The project is OpenCode-only. The Korean reference files (`skills/harness/references/`) stay untouched per IR edge-case scenario. A single sentence in the main SKILL.md is not a "reference file" — it's the primary skill definition and should match the project's stated identity.

### Decision: AGENTS.md determinism

**Choice**: Template-based generation from project context (name, description, tech stack). No AI inference. Templates live at `skills/harness/templates/` (new directory).
**Rationale**: AG-8 mandates determinism. Templates use Handlebars-style `{{placeholder}}` substitution. The harness skill prompts the user for context values, fills templates, writes output. This keeps generation auditable in Git (CC-5).

### Decision: Eval infrastructure

**Choice**: promptfoo with local file assertions only (EI-7: no network). Config at `evals/promptfoo/promptfooconfig.yaml`. Assertions use promptfoo's built-in `file` and `not-contains` checks.
**Rationale**: Since the project has no build system (`openspec/config.yaml` confirms `runner: none`), promptfoo is the lightest validation layer — `npx promptfoo eval`, no global install (CC-2).

---

## Data Flow: Generation

```
User prompt: "Build a harness for this project"
    │
    ▼
skills/harness/SKILL.md (meta-skill activated)
    │
    ├── Phase 1-2: Analyze domain, choose architecture pattern
    │
    ├── Phase 3: Generate agent definitions
    │   └── templates/ → AGENTS.md + .opencode/agents/{{AGENT_NAME}}.md
    │
    ├── Phase 4: Generate skills
    │   └── templates/ → .opencode/skills/{{SKILL_NAME}}/SKILL.md
    │                  → (if compat) .agents/skills/{{SKILL_NAME}}/SKILL.md
    │
    └── Phase 5-6: Wire orchestration + validate
```

The generation is always into the **consumer's project directory**, not into the harness repo itself.

---

## Repo Structure After MVP

```
opencode-team-harness/
├── skills/harness/          # Core meta-skill (unchanged except SKILL.md line 13)
│   ├── SKILL.md             # Modified: remove Claude compat line
│   ├── templates/           # NEW — deterministic generation templates
│   └── references/          # 6 Korean files (unchanged)
├── docs/agentic-development/
│   ├── team-architecture.md # NEW — architecture doc (AG-5)
│   └── eval-strategy.md     # NEW — eval extension guide (EI-8, MAY)
├── evals/promptfoo/
│   └── promptfooconfig.yaml # NEW — eval config (EI-6)
├── examples/mezaos/         # NEW — reference example only (AG-7)
├── README.md                # Modified — Claude→OpenCode, paths, install
├── README_KO.md             # Modified — same transformations in Korean
├── README_JA.md             # Modified — same transformations in Japanese
├── index.html               # Modified — title, i18n, workflow, links
├── privacy.html             # Modified — platform references, output paths
├── .claude-plugin/          # REMOVED
├── CHANGES.md               # Unchanged
├── CHANGELOG.md             # Unchanged
├── openspec/config.yaml     # Unchanged
└── *.png                    # Unchanged
```

### File change summary

| File | Action | Description |
|------|--------|-------------|
| `README.md` | Modify | 9 changes: references Claude→OpenCode, `~/.claude/`→`~/.config/opencode/`, output `.claude/`→`.opencode/`, remove `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` |
| `README_KO.md` | Modify | Same transformations as English, in Korean |
| `README_JA.md` | Modify | Same transformations as Japanese |
| `index.html` | Modify | 10+ changes: `<title>`, hero badge, i18n strings (`hero.badge`, `usecases.desc`, `workflow.phase3.desc`), GitHub links, install steps |
| `privacy.html` | Modify | 7 changes: platform references, output paths, GitHub links |
| `.claude-plugin/plugin.json` | Delete | Part of `.claude-plugin/` removal |
| `.claude-plugin/marketplace.json` | Delete | Part of `.claude-plugin/` removal |
| `skills/harness/SKILL.md` | Modify | Remove line 13 Claude compat note |
| `skills/harness/templates/` | Create | Generation templates for agents and skills |
| `docs/agentic-development/team-architecture.md` | Create | Architecture rationale and interaction flow (AG-5, AG-6) |
| `docs/agentic-development/eval-strategy.md` | Create | How to extend evals (EI-8, MAY) |
| `evals/promptfoo/promptfooconfig.yaml` | Create | Local file assertion evals (EI-6) |
| `examples/mezaos/` | Create | Reference-only example (AG-7) |

---

## Identity Refactor: Exact Changes Per File

### README.md
| # | Line | Old | New |
|---|------|-----|-----|
| 1 | 34 | `.claude/agents/` and `.claude/skills/` | `.opencode/agents/` and `.opencode/skills/` |
| 2 | 34 | Claude Code's agent team system | OpenCode's parallel task system |
| 3 | 50 | `.claude/agents/` | `.opencode/agents/` |
| 4 | 52 | `.claude/skills/` | `.opencode/skills/` |
| 5 | 76-77 | `~/.claude/skills/harness/` | `.opencode/skills/harness/` in the consumer project |
| 6 | 101 | Trigger in Claude Code | Trigger in OpenCode |
| 7 | 137-148 | `.claude/` directory tree | `.opencode/` directory tree |
| 8 | 152 | Copy into Claude Code | Copy into OpenCode |
| 9 | 232 | `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | Remove section entirely (no OpenCode equivalent) |

### index.html
| # | Location | Old | New |
|---|----------|-----|-----|
| 1 | Line 6 `<title>` | for Claude Code | for OpenCode |
| 2 | Line 195 hero badge | Claude Code Plugin | OpenCode Skill |
| 3 | i18n hero.badge (lines 557-559) | Claude Code Plugin | OpenCode Skill |
| 4 | Use cases desc (line 269 + i18n lines 657-659) | Claude Code | OpenCode |
| 5 | Workflow phase 3 (line 371 + i18n lines 817-819) | `.claude/agents/*.md` | `.opencode/agents/*.md` |
| 6 | GitHub links (lines 200, 544, 546) | `revfactory/harness` | `{fork}/opencode-team-harness` |
| 7 | Privacy link (line 547) | `revfactory/harness` | `{fork}/opencode-team-harness` |
| 8 | Footer copyright (line 549 + i18n) | revfactory | {fork owner} |

### privacy.html
| # | Line | Old | New |
|---|------|-----|-----|
| 1 | 36 | Claude Code plugin | OpenCode skill |
| 2 | 40 | local Claude Code environment | local OpenCode environment |
| 3 | 54 | Claude Code runtime environment | OpenCode runtime environment |
| 4 | 57 | `.claude/agents/` and `.claude/skills/` | `.opencode/agents/` and `.opencode/skills/` |
| 5 | 58 | Claude Code's built-in tools | OpenCode's built-in tools |
| 6 | 65 | runs within Claude Code | runs within OpenCode |
| 7 | 73, 82, 85 | `revfactory/harness` | `{fork}/opencode-team-harness` |

---

## Interfaces / Contracts

### Generation template format

Templates use `{{PLACEHOLDER}}` substitution:

```markdown
<!-- .opencode/agents/{{AGENT_NAME}}.md -->
# {{AGENT_NAME}}

**Role**: {{AGENT_ROLE}}
**Trigger**: {{TRIGGER_CONDITION}}
```

### promptfoo config structure

```yaml
# evals/promptfoo/promptfooconfig.yaml
prompts: []
providers: []
tests:
  - description: "No Claude Code in README"
    assert:
      - type: not-contains
        value: "Claude Code"
        file: "README.md"
```

Assertions are file-based only — no promptfoo prompts or providers needed (EI-7).

---

## Testing Strategy

| Layer | What | Approach |
|-------|------|----------|
| **Static analysis** | Identity refactor correctness | promptfoo `not-contains` on all changed files |
| **Structure validation** | Generated file existence | promptfoo `file-exists` on AGENTS.md, `.opencode/agents/*`, `.opencode/skills/*` |
| **Compat mirroring** | `.agents/skills/` parity | promptfoo `file-exists` + content comparison when compat enabled |
| **Determinism** | Identical output for same input | Run generation twice, diff output (CC-3) |

---

## Open Questions

- [ ] MezaOS example content: What minimal files does `examples/mezaos/` need to serve as a "reference only" example without becoming a runtime dependency?
- [ ] The fork's GitHub identity: which org/user name to use for links in the refactored files? (placeholder `{fork}` used above)

---

## Key Design Decisions Map

| Decision | Spec Requirements | Rationale |
|----------|------------------|-----------|
| `.claude-plugin/` removal | IR-4 vs IR-5 conflict | IR-5 wins — metadata lives in SKILL.md |
| Korean refs untouched | IR edge scenario | Author's native language, core logic |
| `.opencode/` primary | IR-7, IR-8, SG-3, SG-4 | Primary authority, compat is derived |
| Template-based generation | AG-8, CC-3 | Deterministic, auditable, no AI |
| promptfoo only | EI-7, CC-2 | No global install, no network |
| No self-generation | Harness design principle | Acyclic — harness generates for consumers |
