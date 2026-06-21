# QA Agent Design Guide

A reference guide for including QA agents in build harnesses. Based on bug patterns discovered in real projects (SatangSlide) and root cause analysis, this provides a systematic verification methodology for catching defects that QA agents commonly miss.

---

## Table of Contents

1. Patterns of Defects QA Agents Miss
2. Integration Coherence Verification
3. QA Agent Design Principles
4. Verification Checklist Template
5. QA Agent Definition Template

---

## 1. Patterns of Defects QA Agents Miss

### 1-1. Boundary Mismatch

The most frequent defect. Two components are each "correctly" implemented, but the contract breaks at their connection point.

| Boundary | Mismatch Example | Why It's Missed |
|--------|-----------|-----------|
| API response → Front-end hook | API returns { projects: [...] }, hook expects SlideProject[] | Each passes individual verification; no cross-comparison done |
| API response field names → Type definitions | API returns thumbnailUrl (camelCase), type defines thumbnail_url (snake_case) | TypeScript generics cast without compiler catching it |
| File path → Link href | Page is at /dashboard/create but link points to /create | File structure and href not cross-compared |
| State transition map → Actual status updates | Map defines generating_template → template_approved, code misses the transition | Only checks map existence, doesn't trace all update code |
| API endpoint → Front-end hook | API exists but no corresponding hook (never called) | API list and hook list not mapped 1:1 |
| Immediate response → Async result | API returns immediate { status }, front-end accesses data.failedIndices | Type checked without distinguishing sync/async response |

### 1-2. Why Static Code Review Can't Catch These

- **TypeScript generic limitations**: fetchJson<SlideProject[]>() — compiles even when runtime response is { projects: [...] }
- **npm run build passing ≠ working correctly**: When type casting, any, or generics are used, builds succeed but fail at runtime
- **Existence verification vs connection verification**: "Does the API exist?" and "Does the API response match the caller's expectations?" are entirely different verifications

---

## 2. Integration Coherence Verification

**Cross-comparison verification** areas that must be included in QA agents.

### 2-1. API Response ↔ Front-end Hook Type Cross-verification

**Method**: Compare the shape of objects passed to NextResponse.json() in each API route with the fetchJson<T> type parameter in the corresponding hook.


Verification steps:
1. Extract the shape of objects passed to NextResponse.json() in API routes
2. Check the T type of fetchJson<T> in corresponding hooks
3. Compare whether shape and T match
4. Check wrapping (if API returns { data: [...] }, does the hook extract .data?)


**Pay special attention to these patterns:**
- Pagination APIs: { items: [], total, page } vs front-end expecting an array
- snake_case DB fields → camelCase API response → front-end type definition mismatches
- Immediate response (202 Accepted) vs final result shape differences

### 2-2. File Path ↔ Link/Router Path Mapping

**Method**: Extract URL paths from page files under src/app/ and cross-reference with all href, router.push(), redirect() values in code.


Verification steps:
1. Extract URL patterns from page.tsx file paths under src/app/
   - (group) → removed from URL
   - [param] → dynamic segment
2. Collect all href=, router.push(, redirect( values in code
3. Confirm each link matches an actually existing page path
4. Mind URL prefixes for pages inside route groups (e.g., under dashboard/)


### 2-3. State Transition Completeness Tracking

**Method**: Extract all status: updates from code and cross-reference with the state transition map.


Verification steps:
1. Extract allowed transitions from state transition map (STATE_TRANSITIONS)
2. Search all API routes for .update({ status: "..." }) patterns
3. Confirm each transition is defined in the map
4. Identify transitions defined in the map but never executed in code (dead transitions)
5. Especially: check that transitions from intermediate states (e.g., generating_template) to final states (template_approved) are not missing


### 2-4. API Endpoint ↔ Front-end Hook 1:1 Mapping

**Method**: List all API routes and front-end hooks to verify they are paired.


Verification steps:
1. Extract endpoint list by HTTP method from route.ts files under src/app/api/
2. Extract fetch call URL list from use*.ts files under src/hooks/
3. Identify API endpoints not called by any hook → flag as "unused"
4. Determine whether "unused" is intentional (admin APIs, etc.) or unintentional (missing call)


---

## 3. QA Agent Design Principles

### 3-1. Use general-purpose Type, Not Explore Type

If the QA agent is Explore type, it can only read. But effective QA requires:
- Grep for pattern searching (extract all NextResponse.json())
- Script execution for automated comparison (API shape vs hook type)
- Ability to modify when needed

**Recommended**: Set as general-purpose type, but specify a "verify → report → request fix" protocol in the agent definition.

### 3-2. Prioritize "Cross-comparison" Over "Existence Checking" in Checklists

| Weak Checklist | Strong Checklist |
|---------------|---------------|
| Does the API endpoint exist? | Does the API endpoint's response shape match the corresponding hook's type? |
| Is the state transition map defined? | Do all status update code paths match the map's transitions? |
| Does the page file exist? | Do all links in code point to actually existing pages? |
| Is TypeScript strict mode enabled? | Is there type safety bypassed through generic casting? |

### 3-3. "Read Both Sides Simultaneously" Principle

For QA to catch boundary bugs, reading only one side is insufficient. You must:
- Read the API route **and** the corresponding hook **together**
- Read the state transition map **and** the actual update code **together**
- Read the file structure **and** the link paths **together**

Explicitly state this principle in the agent definition.

### 3-4. Run QA After Each Module, Not Only After Full Build

If QA is placed only in "Phase 4: After full completion" in the orchestrator:
- Bugs accumulate and fix costs increase
- Early boundary mismatches propagate to subsequent modules

**Recommended pattern**: Perform cross-verification of each backend API + corresponding hook immediately upon completion (incremental QA).

---

## 4. Verification Checklist Template

An integration coherence checklist for web applications to include in QA agent definitions.

markdown
### Integration Coherence Verification (Web App)

#### API ↔ Front-end Connection
- [ ] Response shape of all API routes matches the generic type of corresponding hooks
- [ ] Wrapped responses ({ items: [...] }) are unwrapped in hooks
- [ ] snake_case ↔ camelCase conversion is applied consistently
- [ ] Immediate responses (202) and final results have distinct shapes handled in front-end
- [ ] All API endpoints have corresponding front-end hooks that are actually called

#### Routing Coherence
- [ ] All href/router.push values in code match actual page file paths
- [ ] Path verification accounts for route groups ((group)) being removed from URLs
- [ ] Dynamic segments ([id]) are filled with correct parameters

#### State Machine Coherence
- [ ] All defined state transitions are executed in code (no dead transitions)
- [ ] All status updates in code are defined in the transition map (no unauthorized transitions)
- [ ] Transitions from intermediate states to final states are not missing
- [ ] State-based branching in front-end (if status === "X") — X is actually reachable

#### Data Flow Coherence
- [ ] DB schema field names map consistently to API response field names
- [ ] Front-end type definitions match API response field names
- [ ] Optional field null/undefined handling is consistent on both sides


---

## 5. QA Agent Definition Template

Key sections to include in a build harness QA agent.

markdown
---
name: qa-inspector
description: "QA verification expert. Verifies spec compliance, integration coherence, and design quality."
---

# QA Inspector

## Core Role
Verify implementation quality against specs and **inter-module integration coherence**.

## Verification Priority

1. **Integration coherence** (highest) — Boundary mismatches are the primary cause of runtime errors
2. **Functional spec compliance** — API/state machine/data model
3. **Design quality** — Colors/typography/responsiveness
4. **Code quality** — Unused code, naming conventions

## Verification Method: "Read Both Sides Simultaneously"

Boundary verification must **open and compare code from both sides simultaneously**:

| Verification Target | Left (Producer) | Right (Consumer) |
|----------|-------------|---------------|
| API response shape | NextResponse.json() in route.ts | fetchJson<T> in hooks/ |
| Routing | src/app/ page file paths | href, router.push values |
| State transitions | STATE_TRANSITIONS map | .update({ status }) code |
| DB → API → UI | Table column names | API response fields → type definitions |

## Task Result Reporting Protocol

- Upon discovery, immediately request specific fixes from the relevant agent (file:line + fix method)
- For boundary issues, notify **both** agents on each side
- To parent: verification report (distinguish passed/failed/unverified items)


---

## Real-world Cases: Bugs Found in SatangSlide

All content in this guide was extracted from lessons learned from these actual bugs:

| Bug | Boundary | Cause |
|------|--------|------|
| projects?.filter is not a function | API→Hook | API returns {projects:[]}, hook expects array |
| All dashboard links 404 | File path→href | /dashboard/ prefix missing |
| Theme images not showing | API→Component | thumbnailUrl vs thumbnail_url |
| Theme selection not saving | API→Hook | select-theme API exists, no hook |
| Generation page waiting forever | State transition→Code | template_approved transition code missing |
| data.failedIndices crash | Immediate response→Front-end | Accessing background result from immediate response |
| View slides after completion 404 | File path→href | /projects/ → /dashboard/projects/ |
