# QA Agent Design Guide

A reference guide for including a QA agent in a build harness. Based on bug patterns discovered in a real project (SatangSlide) and root-cause analysis, this guide provides a verification methodology for systematically catching defects that QA tends to miss.

---

## Table of Contents

1. Patterns of Defects That QA Agents Miss
2. Integration Coherence Verification
3. QA Agent Design Principles
4. Verification Checklist Template
5. QA Agent Definition Template

---

## 1. Patterns of Defects That QA Agents Miss

### 1-1. Boundary Mismatch

The most frequent defect. Two components are each implemented "correctly," but the contract breaks at the connection point.

| Boundary | Mismatch Example | Why It's Missed |
|----------|-----------------|-----------------|
| API response → frontend hook | API returns `{ projects: [...] }`, hook expects `SlideProject[]` | Each is validated individually and never cross-compared |
| API response field name → type definition | API uses `thumbnailUrl` (camelCase), type uses `thumbnail_url` (snake_case) | TypeScript generics cast it, so the compiler doesn't catch it |
| File path → link href | Page lives at `/dashboard/create`, but link points to `/create` | File structure and href are not cross-compared |
| State transition map → actual status updates | Map defines `generating_template → template_approved`, but transition is missing from code | Only confirms the map exists without tracing all update code |
| API endpoint → frontend hook | API exists but no corresponding hook (never called) | API list and hook list are not mapped 1:1 |
| Immediate response → async result | API immediately returns `{ status }`, frontend accesses `data.failedIndices` | Types are checked without distinguishing sync/async responses |

### 1-2. Why Static Code Review Fails to Catch This

- **Limits of TypeScript generics**: `fetchJson<SlideProject[]>()` — compiles even if the runtime response is `{ projects: [...] }`
- **`npm run build` passing ≠ correct behavior**: Type casting, `any`, and generics let the build succeed while failing at runtime
- **Existence verification vs. connection verification**: "Does the API exist?" and "Does the API's response match what the caller expects?" are entirely different verifications

---

## 2. Integration Coherence Verification

**Cross-comparison verification** areas that must be included in a QA agent.

### 2-1. API Response ↔ Frontend Hook Type Cross-Validation

**Method**: Compare the `NextResponse.json()` call in each API route with the `fetchJson<T>` type parameter in the corresponding hook.

```
Verification steps:
1. Extract the shape of the object passed to NextResponse.json() in the API route
2. Check the T type in fetchJson<T> in the corresponding hook
3. Compare whether shape and T match
4. Check for wrapping (if API returns { data: [...] }, verify that the hook unwraps .data)
```

**Patterns to watch out for especially:**
- Pagination APIs: `{ items: [], total, page }` vs. frontend expecting an array
- Mismatches across snake_case DB field → camelCase API response → frontend type definition
- Shape difference between immediate response (202 Accepted) vs. final result

### 2-2. File Path ↔ Link/Router Path Mapping

**Method**: Extract URL paths from page files under `src/app/` and cross-check against all `href`, `router.push()`, and `redirect()` values in the code.

```
Verification steps:
1. Extract URL patterns from page.tsx file paths under src/app/
   - (group) → removed from URL
   - [param] → dynamic segment
2. Collect all href=, router.push(, redirect( values in the code
3. Verify that each link matches an actually existing page path
4. Watch out for URL prefixes of pages inside route groups (e.g., under dashboard/)
```

### 2-3. State Transition Completeness Tracking

**Method**: Extract all `status:` updates from code and cross-check against the state transition map.

```
Verification steps:
1. Extract the list of allowed transitions from the state transition map (STATE_TRANSITIONS)
2. Search for .update({ status: "..." }) patterns in all API routes
3. Verify that each transition is defined in the map
4. Identify transitions defined in the map but never executed in code (dead transitions)
5. Especially: confirm that the transition from an intermediate state (e.g., generating_template)
   to a final state (template_approved) is not missing
```

### 2-4. API Endpoint ↔ Frontend Hook 1:1 Mapping

**Method**: List all API routes and frontend hooks and verify they are paired.

```
Verification steps:
1. Extract endpoint list by HTTP method from route.ts files under src/app/api/
2. Extract fetch call URL list from use*.ts files under src/hooks/
3. Identify API endpoints not called by any hook → flag as "unused"
4. Determine whether "unused" is intentional (e.g., admin API) or a missing call
```

---

## 3. QA Agent Design Principles

### 3-1. Use general-purpose type, not Explore type

If the QA agent is `Explore` type, it can only read. But effective QA requires:
- Grep to search for patterns (extract all `NextResponse.json()`)
- Running scripts to automate cross-checking (API shape vs. hook type)
- The ability to make fixes when needed

**Recommendation**: Set to `general-purpose` type, but explicitly state a "verify → report → request fix" protocol in the agent definition.

### 3-2. Prioritize "cross-comparison" over "existence check" in checklists

| Weak Checklist | Strong Checklist |
|---------------|-----------------|
| Does the API endpoint exist? | Does the API endpoint's response shape match the type of its corresponding hook? |
| Is the state transition map defined? | Do all status update code paths match transitions in the map? |
| Does the page file exist? | Do all links in the code point to actually existing pages? |
| Is TypeScript strict mode on? | Is there no type safety bypassed via generic casting? |

### 3-3. "Read both sides simultaneously" principle

To catch boundary bugs, QA must not read only one side. Always:
- Read the API route **and** the corresponding hook **together**
- Read the state transition map **and** the actual update code **together**
- Read the file structure **and** the link paths **together**

State this principle explicitly in the agent definition.

### 3-4. Run QA immediately after each module is complete, not after the build

If QA is placed in the Orchestrator only at "Phase 4: After everything is complete":
- Bugs accumulate and the cost of fixing them grows
- Early boundary mismatches propagate to subsequent modules

**Recommended pattern**: Perform cross-validation of each backend API + its corresponding hook immediately after each API is complete (incremental QA).

---

## 4. Verification Checklist Template

An integration coherence checklist for web applications to include in QA agent definitions.

```markdown
### Integration Coherence Verification (Web App)

#### API ↔ Frontend Connection
- [ ] Response shape of all API routes matches the generic type of their corresponding hooks
- [ ] Wrapped responses ({ items: [...] }) are unwrapped in the hook
- [ ] snake_case ↔ camelCase conversion is applied consistently
- [ ] Immediate responses (202) and final result shapes are distinguished in the frontend
- [ ] All API endpoints have a corresponding frontend hook that is actually called

#### Routing Coherence
- [ ] All href/router.push values in code match actual page file paths
- [ ] Path validation accounts for route groups ((group)) being removed from the URL
- [ ] Dynamic segments ([id]) are populated with the correct parameters

#### State Machine Coherence
- [ ] All defined state transitions are executed in code (no dead transitions)
- [ ] All status updates in code are defined in the transition map (no unauthorized transitions)
- [ ] Transitions from intermediate states to final states are not missing
- [ ] Values X in frontend status-based branches (if status === "X") are actually reachable

#### Data Flow Coherence
- [ ] Mapping between DB schema field names and API response field names is consistent
- [ ] Frontend type definitions and API response field names match
- [ ] null/undefined handling for optional fields is consistent on both sides
```

---

## 5. QA Agent Definition Template

Key sections to include in the QA agent for a build harness.

```markdown
---
name: qa-inspector
description: "QA verification specialist. Validates spec compliance, integration coherence, and design quality."
---

# QA Inspector

## Core Role
Validate implementation quality against spec and **integration coherence across modules**.

## Verification Priority

1. **Integration coherence** (highest) — boundary mismatches are the primary cause of runtime errors
2. **Functional spec compliance** — API / state machine / data model
3. **Design quality** — colors / typography / responsiveness
4. **Code quality** — unused code, naming conventions

## Verification Method: "Read Both Sides Simultaneously"

Boundary verification must be done by **opening both sides of the code at the same time** and comparing:

| Verification Target | Left (Producer) | Right (Consumer) |
|--------------------|----------------|-----------------|
| API response shape | NextResponse.json() in route.ts | fetchJson<T> in hooks/ |
| Routing | Page file paths in src/app/ | href, router.push values |
| State transitions | STATE_TRANSITIONS map | .update({ status }) code |
| DB → API → UI | Table column names | API response fields → type definitions |

## Team Communication Protocol

- Request specific fixes from the relevant agent immediately upon discovery (file:line + how to fix)
- Notify **both** agents for boundary issues
- Report to leader: verification report (distinguish passed / failed / unchecked items)
```

---

## Real-World Cases: Bugs Found in SatangSlide

All content in this guide is distilled from the following real bugs:

| Bug | Boundary | Cause |
|-----|----------|-------|
| `projects?.filter is not a function` | API → hook | API returned `{projects:[]}`, hook expected array |
| All dashboard links returning 404 | File path → href | Missing `/dashboard/` prefix |
| Theme images not showing | API → component | `thumbnailUrl` vs `thumbnail_url` |
| Theme selection not saving | API → hook | select-theme API existed, no hook |
| Generation page stuck waiting forever | State transition → code | Missing `template_approved` transition code |
| `data.failedIndices` crash | Immediate response → frontend | Background result accessed from immediate response |
| View slides after completion returning 404 | File path → href | `/projects/` → `/dashboard/projects/` |
