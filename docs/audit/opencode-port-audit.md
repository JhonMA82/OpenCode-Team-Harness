# OpenCode Port Audit — opencode-team-harness

## 1. Current State of the Fork

This repository is a fork of [revfactory/harness](https://github.com/revfactory/harness), originally a Claude Code plugin that provides a meta-skill for designing agent teams, defining specialized agents, and generating skills. It was ported to OpenCode by `PriuS2/harness-opencode` across three commits (`9e037bf`, `5538ba7`, `f032ae6`).

**What the port did:**
- Changed execution architecture from Claude Code's "Agent Teams" (TeamCreate + SendMessage) to OpenCode's "Parallel Tasks" (Task tool with `run_in_background: true`)
- Hardcoded model references → `{provider/model}` placeholders
- Updated `skills/harness/SKILL.md` to use `.opencode/agents/` and `.opencode/skills/` paths instead of `.claude/`
- Added warnings about Claude Code incompatibility in READMEs and badges

**What was NOT ported:**
- No `.opencode/` directory exists in the project (empty state — the harness generates these *for other projects*)
- No `AGENTS.md` file exists
- No `opencode.json` or `opencode.jsonc` configuration
- READMEs, landing page, and privacy policy still largely reference Claude Code
- Plugin metadata (`.claude-plugin/`) still points to the original `revfactory/harness` repo/ecosystem
- `plugin.json` still describes it as a Claude Code plugin

## 2. Parts Still Inherited from Claude Code

### Plugin Metadata — `.claude-plugin/` directory

| File | Issue |
|------|-------|
| `.claude-plugin/plugin.json` | Format is Claude Code plugin manifest; `homepage` and `repository` point to `revfactory/harness`; no OpenCode equivalent exists |
| `.claude-plugin/marketplace.json` | Claude Code marketplace format; owned by `revfactory` |

### Landing Page — `index.html`

| Location | Issue |
|----------|-------|
| Line 6 (`<title>`) | "Agent Team & Skill Architect for **Claude Code**" |
| Line 195 (hero badge) | `data-i18n="hero.badge"` default text: "**Claude Code** Plugin" |
| Line 269 (use cases desc) | "Copy any prompt into **Claude Code** after installing Harness" |
| Line 371 (workflow phase 3) | "Generate `.claude/agents/*.md` files" |
| Lines 557-559 (i18n `hero.badge`) | "Claude Code Plugin" in EN/KO/JA |
| Lines 657-659 (i18n `usecases.desc`) | "Copy any prompt into Claude Code" in EN/KO/JA |
| Lines 817-819 (i18n `workflow.phase3.desc`) | "Generate `.claude/agents/*.md`" in EN/KO/JA |

### Privacy Policy — `privacy.html`

| Line | Issue |
|------|-------|
| 36 | "open-source **Claude Code** plugin" |
| 40 | "runs entirely within your local **Claude Code** environment" |
| 54 | "operates exclusively within the **Claude Code** runtime environment" |
| 57 | "generates Markdown files (`.claude/agents/` and `.claude/skills/`)" |
| 58 | "All processing happens locally through **Claude Code's** built-in tools" |
| 65 | "Harness runs within **Claude Code**, which is operated by Anthropic" |

### GitHub URLs

All GitHub links in `index.html`, `privacy.html`, `plugin.json`, and READMEs point to `revfactory/harness` — the original repo, not this fork. Footer links, "Read the Paper", "Harness 100", and privacy policy "open an issue" links all reference `revfactory/harness`.

### Requirements Section

All three READMEs reference `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` and link to `code.claude.com/docs/en/agent-teams` — a Claude Code feature that does not exist in OpenCode.

## 3. Parts Already Adapted to OpenCode

### Skills/Harness/SKILL.md

This is the **only file fully adapted** to OpenCode:
- Uses `.opencode/agents/` and `.opencode/skills/` paths (lines 11, 78, 99, 265-266)
- Uses Task tool with `run_in_background: true` as default execution mode
- Architecture patterns reference Task-based parallelism, not Claude Code Agent Teams
- Model format uses `{provider/model}` placeholders (line 85)
- Checklist references `.opencode/` paths (lines 265-266)

### CHANGES.md

Documents the port cleanly with before/after comparison table and compatibility warning.

### README badges

- "Fork — Opencode Port" badge
- "Changed — Agent Team → Parallel Tasks" badge (partially — still references "Agent Team" as the "before")

## 4. Files Mentioning .claude, CLAUDE.md, Claude Code, or Wrong Paths

### README.md (English)

| Line | Content | Issue |
|------|---------|-------|
| 34 | "`.claude/agents/` and `.claude/skills/`" | Wrong path for OpenCode |
| 50 | "Agent Definition Generation (.claude/agents/)" | Wrong path |
| 52 | "Skill Generation (.claude/skills/)" | Wrong path |
| 76-77 | "Copy the skills directory to `~/.claude/skills/harness/`" | Wrong install path |
| 101 | "Trigger in **Claude Code** with prompts like:" | Wrong platform |
| 137-148 | `.claude/` directory tree in "Output" section | Wrong path |
| 152 | "Copy any prompt below into **Claude Code**" | Wrong platform |
| 232 | `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | Claude-specific env var |

### README_KO.md (Korean)

| Line | Content | Issue |
|------|---------|-------|
| 34 | "`.claude/agents/` 와 `.claude/skills/`" | Wrong path |
| 50 | "에이전트 정의 생성 (.claude/agents/)" | Wrong path |
| 52 | "스킬 생성 (.claude/skills/)" | Wrong path |
| 76-77 | "`~/.claude/skills/harness/`에 복사" | Wrong install path |
| 101 | "**Claude Code**에서 다음과 같이 트리거한다" | Wrong platform |
| 137-148 | `.claude/` directory tree | Wrong path |
| 152 | "프롬프트를 **Claude Code**에 복사" | Wrong platform |
| 225 | `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | Claude-specific env var |

### README_JA.md (Japanese)

| Line | Content | Issue |
|------|---------|-------|
| 34 | "`.claude/agents/` と `.claude/skills/`" | Wrong path |
| 50 | "エージェント定義の生成（.claude/agents/）" | Wrong path |
| 52 | "スキル生成（.claude/skills/）" | Wrong path |
| 76-77 | "`~/.claude/skills/harness/` にコピー" | Wrong install path |
| 101 | "**Claude Code**で以下のように呼び出します" | Wrong platform |
| 137-148 | `.claude/` directory tree | Wrong path |
| 152 | "プロンプトを**Claude Code**にコピー" | Wrong platform |
| 232 | `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` | Claude-specific env var |

### index.html

| Line | Content | Issue |
|------|---------|-------|
| 6 | "for Claude Code" | Wrong platform in `<title>` |
| 195 | "Claude Code Plugin" badge | Wrong platform label |
| 269 | "Copy any prompt into **Claude Code**" | Wrong platform in use cases |
| 371 | "Generate `.claude/agents/*.md`" | Wrong path in workflow phase 3 |

### privacy.html

| Line | Content | Issue |
|------|---------|-------|
| 36 | "open-source **Claude Code** plugin" | Wrong platform identity |
| 40 | "local **Claude Code** environment" | Wrong runtime |
| 54 | "within the **Claude Code** runtime environment" | Wrong runtime |
| 57 | "`.claude/agents/` and `.claude/skills/`" | Wrong output paths |
| 58 | "**Claude Code's** built-in tools" | Wrong platform tools |
| 65 | "runs within **Claude Code**" | Wrong platform |

### skills/harness/SKILL.md

| Line | Content | Issue |
|------|---------|-------|
| 13 | "Claude 호환 경로(.claude/, ~/.claude/)도 지원한다" | Maintains Claude compat as a feature — contradicts the "OpenCode only" stance in READMEs |

### plugin.json

| Key | Value | Issue |
|-----|-------|-------|
| homepage | `https://github.com/revfactory/harness` | Points to original repo |
| repository | `https://github.com/revfactory/harness` | Points to original repo |

## 5. Files in Korean or Other Languages to Translate/Normalize

### Korean Primary Content

| File | Language | Lines | Notes |
|------|----------|-------|-------|
| `skills/harness/SKILL.md` | Korean (primary) + English terms | 283 | Technical content with Korean explanations and English code terms. References use Korean. Line 3 mentions "Claude 호환 경로도 지원" — needs decision on whether to keep or remove Claude compat. |
| `skills/harness/references/agent-design-patterns.md` | Korean | 295 | Technical reference — should remain Korean if the skill authoring language is Korean. |
| `skills/harness/references/orchestrator-template.md` | Korean | Unknown | Same as above. |
| `skills/harness/references/team-examples.md` | Korean | Unknown | Same as above. |
| `skills/harness/references/skill-writing-guide.md` | Korean | 268 | Same as above. |
| `skills/harness/references/skill-testing-guide.md` | Korean | Unknown | Same as above. |
| `skills/harness/references/qa-agent-guide.md` | Korean | Unknown | Same as above. |
| `README_KO.md` | Korean | 229 | Mixed state — OpenCode port notice + Claude Code content. |
| `CHANGELOG.md` | Korean | 26 | Changelog, fine to stay in Korean. |

### Japanese Content

| File | Language | Lines | Notes |
|------|----------|-------|-------|
| `README_JA.md` | Japanese | 236 | Mixed state — OpenCode port notice + Claude Code content. Same issues as English/Korean READMEs. |

### Assessment

The Korean content in `skills/harness/` is the **author's native language** for technical writing — it is the source of truth for the harness skill logic. These files should stay in Korean. Only the interface files (READMEs, landing page) need Claude Code references cleaned up. The i18n system in `index.html` is well-designed and should be preserved.

## 6. Parts That Serve OpenCode Directly

### Skill Logic (skills/harness/)

- `SKILL.md` — already adapted to OpenCode Task-based architecture. The content (phases, patterns, guardrails) is platform-agnostic at its core and works for OpenCode.
- All 6 reference files — platform-agnostic technical content about agent design patterns, skill writing, testing, orchestration, QA. These are valid for OpenCode.

### Architecture Patterns

The 6 patterns (Pipeline, Fan-out/Fan-in, Expert Pool, Producer-Reviewer, Supervisor, Hierarchical Delegation) are platform-agnostic and equally applicable to OpenCode's Task tool.

### CHANGES.md

Documents the port history clearly. Valuable as-is.

### openspec/config.yaml

Already configured for SDD workflow with OpenCode-specific context. Valid as-is.

## 7. Parts Requiring Adaptation

### README.md (English) — High Priority

| Change Needed | Location |
|---------------|----------|
| Replace `~/.claude/skills/harness/` install path with `~/.config/opencode/skills/` | Lines 76-77 |
| Replace `.claude/agents/` with `.opencode/agents/` | Lines 34, 50, 137-148 |
| Replace `.claude/skills/` with `.opencode/skills/` | Lines 34, 52, 137-148 |
| Replace "Trigger in Claude Code" with "Trigger in OpenCode" | Line 101 |
| Replace "Copy any prompt below into Claude Code" with OpenCode equivalent | Line 152 |
| Remove `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` requirement or replace with OpenCode equivalent | Line 232 |
| Replace overview paragraph about "Claude Code's agent team system" | Line 34 |

### README_KO.md and README_JA.md — High Priority

Same changes as English README, translated appropriately.

### index.html — High Priority

| Change Needed | Location |
|---------------|----------|
| Change `<title>` from "for Claude Code" to "for OpenCode" or remove platform reference | Line 6 |
| Change hero badge from "Claude Code Plugin" to "OpenCode Skill" or similar | Line 195 + i18n lines 557-559 |
| Change use cases description from "Claude Code" to "OpenCode" | Lines 269, 657-659 |
| Change workflow phase 3 description from `.claude/agents/*.md` to `.opencode/agents/*.md` | Lines 371, 817-819 |
| Change install steps from `/plugin marketplace` to OpenCode equivalent | Lines 524-534 |

### privacy.html — Medium Priority

| Change Needed | Location |
|---------------|----------|
| Replace "Claude Code plugin" with "OpenCode skill/plugin" | Line 36 |
| Replace "local Claude Code environment" with "local OpenCode environment" | Line 40 |
| Replace "Claude Code runtime environment" with "OpenCode runtime environment" | Line 54 |
| Replace `.claude/agents/` and `.claude/skills/` with `.opencode/` equivalents | Line 57 |
| Replace "Claude Code's built-in tools" with "OpenCode's built-in tools" | Line 58 |
| Update Anthropic privacy reference to OpenCode privacy context | Line 65 |

### plugin.json — Medium Priority

The entire `.claude-plugin/` directory is a Claude Code concept. OpenCode does not use plugin manifests. This directory should either be removed or replaced with an OpenCode equivalent. The metadata (name, description, version, author, license) should be preserved in a format OpenCode understands.

### skills/harness/SKILL.md — Low Priority

Line 13 maintains "Claude 호환 경로(.claude/, ~/.claude/)도 지원한다" (Claude-compatible paths also supported). If the project is truly "OpenCode only", this line should be removed or clarified that it's for backwards compatibility only.

## 8. Compatibility Risks

### Identity Confusion

The most significant risk is that the project claims "Opencode only" in its README badges while every other surface (metadata, landing page, privacy policy, install instructions) describes it as a Claude Code plugin. Users coming from Claude Code will find broken install paths; users coming from OpenCode will be confused about what platform this is for.

### Broken Installation Paths

All READMEs instruct users to copy skills to `~/.claude/skills/harness/`. OpenCode stores skills at `~/.config/opencode/skills/`. Users following the README will install to the wrong location and the skill won't load.

### Plugin Metadata Orphaned

The `.claude-plugin/` directory and its JSON files have no meaning in OpenCode. If an OpenCode user inspects the repo, they'll wonder what these files are. If a Claude Code user finds this fork (which claims to be OpenCode-only), they'll get a broken experience.

### GitHub Link Mismatch

All GitHub links point to `revfactory/harness` (the original), not to this fork. Users clicking "GitHub" from the landing page will go to the Claude Code original, not the OpenCode fork. The privacy policy's "audit the source code" link points to the wrong repo.

### `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` Requirement

All three READMEs list this as a requirement. This environment variable is Claude Code-specific and has no effect in OpenCode. Users may either skip it (fine) or be confused about why they need a Claude Code env var for an OpenCode tool.

## 9. What to Keep

### Skill Content (skills/harness/)

All of `skills/harness/SKILL.md` and its 6 reference files have high value. The architecture patterns, design methodologies, skill writing guides, testing frameworks, and orchestration templates are platform-agnostic and well-written. These are the core value of the project.

### CHANGES.md

Clear port documentation. Keep as-is.

### CHANGELOG.md

Useful history. Keep as-is (Korean is fine for project history).

### openspec/ Directory

Already configured for SDD workflow. Keep as-is.

### i18n System in index.html

The JavaScript-based i18n system (EN/KO/JA toggle) is well-implemented and should be preserved. Only the translation strings mentioning "Claude Code" need updating.

### Visual Assets

All PNG files (banner, icons, team diagram, social card) are platform-agnostic. Keep as-is.

## 10. What to Defer

### Deep i18n Translation of Reference Files

The 6 reference files in `skills/harness/references/` are in Korean. Translating them to English would be valuable for wider adoption but is not critical for the port to function. These should stay in Korean for now — the author's primary technical language is Korean, and the skill descriptions in `SKILL.md` use Korean which triggers correctly in both languages.

### OpenCode-Specific Plugin Format

OpenCode may or may not have a plugin/skill marketplace equivalent. Creating an OpenCode-specific metadata format should be deferred until OpenCode's plugin system is confirmed.

### Renaming the Project

"Harness" is a generic term, but the project's identity is tied to the `revfactory/harness` ecosystem. Forking the branding (repo name, URLs, badges) would be a larger effort. For now, the fork name `opencode-team-harness` and its badges are sufficient differentiation.

### Removing Claude Code Compat from SKILL.md

Line 13 of SKILL.md mentions Claude-compatible paths. This could be kept as a compatibility note or removed. Low priority — it's a single line in a Korean comment and doesn't affect OpenCode operation.

### Testing Infrastructure

The project has no test infrastructure, no CI pipeline (`openspec/config.yaml` confirms `runner: none`). Setting up tests is a separate initiative.

### Full Landing Page Redesign

The landing page (`index.html`) is visually polished and well-designed. It needs content updates (Claude Code → OpenCode) but not a redesign. Defer any visual/structural overhaul.
