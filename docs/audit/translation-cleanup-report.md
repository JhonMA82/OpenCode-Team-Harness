# Translation Cleanup Report

## Scope

- Translated Korean comments, instructions, and internal documentation under `skills/harness` to English.
- Preserved OpenCode terminology, routes, placeholders, commands, and architecture intent.
- No commits were created.

## Files updated

- `skills/harness/SKILL.md`
- `skills/harness/references/agent-design-patterns.md`
- `skills/harness/references/orchestrator-template.md`
- `skills/harness/references/qa-agent-guide.md`
- `skills/harness/references/skill-testing-guide.md`
- `skills/harness/references/skill-writing-guide.md`
- `skills/harness/references/team-examples.md`
- `docs/audit/translation-cleanup-report.md`

## Verification

Run:

```sh
grep -RInP "[\x{AC00}-\x{D7AF}]" skills/harness || true
git diff --check
git diff --stat
git status --short
```

## Korean remaining

The only intentional Korean remaining under `skills/harness` is placeholder content:

- Strict placeholders in braces, including examples such as `{테스트 프롬프트}`, `{스킬 이름}`, `{도메인}`, `{트리거 키워드}`, `{역할}`, `{무엇을 파악하는지}`, `{역할 설명 및 작업 지시}`, `{통합/검증 로직}`, `{입력}`, and `{분석 결과}`.
- Placeholder-like examples embedded in orchestrator-template text where the Korean token remains inside `{...}` and must not be translated.

## Manual verification

- Reviewed the remaining Hangul matches from `grep -RInP "[\x{AC00}-\x{D7AF}]" skills/harness || true`.
- Confirmed remaining Korean text is limited to strict `{...}` placeholders and orchestrator-template placeholder-like example tokens that intentionally remain unchanged.
- Confirmed Korean text outside strict placeholders in `team-examples.md` and `agent-design-patterns.md` was translated to English.

## Translation notes

- Kept `Task`, `OpenCode`, `Explore`, `Plan`, `general-purpose`, route paths, placeholders, and code identifiers unchanged.
- Translated Korean prose inside example prompts/descriptions because it functions as internal documentation and instruction text.
- Preserved Korean placeholder tokens such as `{도메인}`, `{역할}`, and `{입력}` in `orchestrator-template.md` because placeholders must not be changed.
- Preserved the harness architecture: parent orchestration, Task parallelism/sequencing, agent/skill separation, QA cross-boundary validation, and progressive disclosure.

## Translation doubts / uncertainties

No unresolved translation doubts remain. Remaining Korean text is intentional placeholder content only.
