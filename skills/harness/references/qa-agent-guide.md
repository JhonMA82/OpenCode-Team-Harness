# QA Agent Design Guide

A guide for including a QA agent in a build harness. Based on bug patterns found in a real project (SatangSlide) and root-cause analysis, it provides a validation methodology for systematically catching defects QA often misses.

---

## Table of Contents

1. Patterns of defects QA agents miss
2. Integration Coherence Verification
3. QA agent design principles
4. Validation checklist template
5. QA agent definition template

---

## 1. Patterns of defects QA agents miss

### 1-1. Boundary Mismatch

The most frequent defect. Two components are each implemented "correctly," but the contract breaks at the connection point.

| Boundary | Mismatch example | Why it is missed |
|--------|-----------|-----------|
| API response → frontend hook | API returns `{ projects: [...] }`, hook expects `SlideProject[]` | Each side passes individual validation; no cross-comparison |
| API response field name → type definition | API uses `thumbnailUrl` (camelCase), type uses `thumbnail_url` (snake_case) | TypeScript generic casts can hide it from the compiler |
| File path → link href | Page is under `/dashboard/create`, but link points to `/create` | File structure is not cross-checked against href values |
| State transition map → actual status update | Map defines `generating_template` → `template_approved`, code omits the transition | Checks only that the map exists; does not trace all update code |
| API endpoint → frontend hook | API exists but no corresponding hook exists (never called) | API list and hook list are not mapped 1:1 |
| Immediate response → async result | API immediately returns `{ status }`, frontend accesses `data.failedIndices` | Type-only validation ignores sync/async response differences |

### 1-2. Why static code review misses these

- **Limits of TypeScript generics**: `fetchJson<SlideProject[]>()` compiles even if the runtime response is `{ projects: [...] }`.
- **`npm run build` passing ≠ correct behavior**: when casts, `any`, or generics are used, the build can succeed while runtime fails.
- **Existence validation vs connection validation**: "Does the API exist?" and "Does the API response match the caller's expectation?" are entirely different checks.

---

## 2. Integration Coherence Verification

Cross-comparison validation areas that MUST be included in a QA agent.

### 2-1. Cross-validate API responses ↔ frontend hook types

**Method**: Compare each API route's `NextResponse.json()` call site with the corresponding hook's `fetchJson<T>` type parameter.

Validation steps:
1. Extract the object shape passed to `NextResponse.json()` in the API route.
2. Check the `T` type in the corresponding hook's `fetchJson<T>` call.
3. Compare the shape against `T`.
4. Check wrapping/unwrapping behavior (if the API returns `{ data: [...] }`, confirm the hook reads `.data`).

**Patterns requiring special attention:**
- Pagination APIs: `{ items: [], total, page }` vs frontend expecting an array.
- snake_case DB field → camelCase API response → frontend type definition mismatches.
- Immediate response (`202 Accepted`) vs final result shape differences.

### 2-2. Map file paths ↔ link/router paths

**Method**: Extract URL paths from `page` files under `src/app/`, then compare them with every `href`, `router.push()`, and `redirect()` value in code.

Validation steps:
1. Extract URL patterns from `page.tsx` paths under `src/app/`.
   - `(group)` → remove from URL.
   - `[param]` → dynamic segment.
2. Collect every `href=`, `router.push(`, and `redirect(` value in code.
3. Confirm each link matches a real page path.
4. Pay attention to URL prefixes for pages inside route groups (example: under `dashboard/`).

### 2-3. Trace state transition completeness

**Method**: Extract every `status:` update in code and compare it with the state transition map.

Validation steps:
1. Extract the allowed transition list from the state transition map (`STATE_TRANSITIONS`).
2. Search all API routes for `.update({ status: "..." })` patterns.
3. Confirm each transition is defined in the map.
4. Identify transitions defined in the map but never executed in code (dead transitions).
5. Especially confirm that transitions from intermediate states (example: `generating_template`) to final states (`template_approved`) are not missing.

### 2-4. Map API endpoints ↔ frontend hooks 1:1

**Method**: List all API routes and frontend hooks, then confirm they are paired correctly.

Validation steps:
1. Extract endpoint lists by HTTP method from `route.ts` files under `src/app/api/`.
2. Extract fetch call URLs from `use*.ts` files under `src/hooks/`.
3. Identify API endpoints that no hook calls → flag as "unused."
4. Decide whether "unused" is intentional (admin API, etc.) or not (missed call).

---

## 3. QA agent design principles

### 3-1. Use the `general-purpose` type, not `Explore`

If a QA agent uses the `Explore` type, it can only read. Effective QA needs to:
- Search patterns with Grep (extract every `NextResponse.json()`).
- Run scripts for automatic comparison (API shape vs hook type).
- Apply fixes when needed.

**Recommended**: set the agent to `general-purpose`, and specify a "validate → report → request fixes" protocol in the agent definition.

### 3-2. Prioritize cross-comparison over "existence checks"

| Weak checklist | Strong checklist |
|---------------|---------------|
| Does the API endpoint exist? | Does the API endpoint response shape match the corresponding hook type? |
| Is the state transition map defined? | Do all status update statements match transitions in the map? |
| Does the page file exist? | Do all links in code point to real pages? |
| Is TypeScript strict mode enabled? | Is type safety bypassed through generic casts? |

### 3-3. "Read both sides together" principle

To catch boundary bugs, QA must not read only one side. It MUST:
- Read the API route **and** corresponding hook **together**.
- Read the state transition map **and** actual update code **together**.
- Read the file structure **and** link paths **together**.

State this principle explicitly in the agent definition.

### 3-4. Run QA immediately after each module, not only after build

If the orchestrator places QA only in "Phase 4: after everything is complete":
- Bugs accumulate and become more expensive to fix.
- Early boundary mismatches propagate into later modules.

**Recommended pattern**: immediately after each backend API is completed, cross-validate that API and its corresponding hook (incremental QA).

---

## 4. Validation checklist template

Integration coherence checklist for web applications to include in QA agent definitions.

markdown
### Integration Coherence Verification (Web App)

#### API ↔ Frontend Connection
- [ ] Every API route response shape matches the corresponding hook's generic type.
- [ ] Wrapped responses (`{ items: [...] }`) are unwrapped by hooks.
- [ ] snake_case ↔ camelCase conversion is applied consistently.
- [ ] Frontend distinguishes immediate responses (202) from final result shapes.
- [ ] Every API endpoint has a corresponding frontend hook and is actually called.

#### Routing Coherence
- [ ] Every `href`/`router.push` value in code matches a real page file path.
- [ ] Route groups (`(group)`) are removed from URLs during path validation.
- [ ] Dynamic segments (`[id]`) are filled with correct parameters.

#### State Machine Coherence
- [ ] Every defined state transition is executed in code (no dead transitions).
- [ ] Every status update in code is defined in the transition map (no unauthorized transitions).
- [ ] Transitions from intermediate states to final states are not missing.
- [ ] Frontend branches based on status (`if status === "X"`) can actually be reached.

#### Data Flow Coherence
- [ ] DB schema field names and API response field mappings are consistent.
- [ ] Frontend type definitions match API response field names.
- [ ] Null/undefined handling for optional fields is consistent on both sides.

---

## 5. QA agent definition template

Core sections to include in a build harness QA agent.

markdown
---
name: qa-inspector
description: "QA validation specialist. Validates spec compliance, integration coherence, and design quality."
---

# QA Inspector

## Core Responsibilities
Validate implementation quality against the spec and **integration coherence between modules**.

## Validation Priority

1. **Integration coherence** (highest) — boundary mismatches are a major cause of runtime errors.
2. **Functional spec compliance** — API/state machine/data model.
3. **Design quality** — color/typography/responsive behavior.
4. **Code quality** — unused code, naming rules.

## Validation Method: "Read both sides together"

For boundary validation, always **open both sides of the code together** and compare:

| Validation target | Left side (producer) | Right side (consumer) |
|----------|-------------|---------------|
| API response shape | `NextResponse.json()` in route.ts | `fetchJson<T>` in hooks/ |
| Routing | `src/app/` page file path | `href`, `router.push` values |
| State transition | `STATE_TRANSITIONS` map | `.update({ status })` code |
| DB → API → UI | Table column names | API response fields → type definitions |

## Task Result Reporting Protocol

- When an issue is found, immediately ask the relevant agent for a concrete fix (file:line + fix method).
- Notify **both** agents involved in boundary issues.
- Report to parent: validation report distinguishing passed/failed/unverified items.

---

## Real cases: bugs found in SatangSlide

Every lesson in this guide was extracted from the real bugs below:

| Bug | Boundary | Cause |
|------|--------|------|
| `projects?.filter is not a function` | API→hook | API returned `{projects:[]}`, hook expected an array |
| All dashboard links 404 | file path→href | Missing `/dashboard/` prefix |
| Theme image not visible | API→component | `thumbnailUrl` vs `thumbnail_url` |
| Theme selection not saved | API→hook | `select-theme` API exists, no hook |
| Generate page waits forever | state transition→code | missing `template_approved` transition code |
| `data.failedIndices` crash | immediate response→frontend | frontend accessed background result from immediate response |
| View slides after completion 404 | file path→href | `/projects/` → `/dashboard/projects/` |
