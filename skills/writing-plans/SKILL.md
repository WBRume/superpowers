---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** This should be run in a dedicated worktree (created by brainstorming skill).

**Save plans to:**
- Single-plan mode (default): `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- Full-stack layered mode (required for frontend+backend work):
  `docs/superpowers/plans/YYYY-MM-DD-<feature-name>/api_plan.md`
  `docs/superpowers/plans/YYYY-MM-DD-<feature-name>/frontend_plan.md`
  `docs/superpowers/plans/YYYY-MM-DD-<feature-name>/backend_plan.md`
- (User preferences for plan location override these defaults)

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## Layered Planning Mode (Full-Stack Required)

When a request includes both frontend and backend implementation, you MUST use layered planning mode. Do not produce one mixed implementation plan.

**Hard dependency order (never violate):**
1. Write `api_plan.md` first
2. Write `frontend_plan.md` based on the finalized API plan
3. Write `backend_plan.md` based on the finalized API plan

**Strict rule:** Never start `frontend_plan.md` or `backend_plan.md` before `api_plan.md` is complete and stable.

**Layer content requirements:**
- `api_plan.md` defines all endpoints, HTTP methods, paths, request fields, response fields, types, error responses, and contract examples.
- `frontend_plan.md` defines component tree, state/data flow, and mock-consumption behavior bound to the contract (`MOCK_BASE_URL`, fallback `API_MOCK_BASE_URL`).
  If the frontend stack is Vite, the plan must include updating `vite.config.*` proxy target to the mock baseline source.
- `backend_plan.md` defines data models, business logic, controller boundaries, and unit tests that enforce the same contract.
- Both `frontend_plan.md` and `backend_plan.md` must include explicit automated test tasks and pass criteria.
- For Vue/Vite frontend plans, test tasks must explicitly include:
  - framework bootstrap when missing (`vitest` + `@vue/test-utils` + `jsdom`)
  - at least one API-layer test and one UI behavior test
  - runnable test command and expected PASS output

**Contract consistency rules:**
- Frontend and backend plans may only reference fields/types defined in `api_plan.md`.
- If contract changes, update `api_plan.md` first, then update both downstream plans.
- Layered implementation plans must include explicit real API interface steps for:
  - `POST /api/workspaces/{ws_id}/api-mock/projects/{task_id}/swagger/import`
    - request format must be `multipart/form-data` with `raw_content` or `file` or `source_url` (no JSON body)
    - plan should show canonical invocation shape (`Invoke-RestMethod -Form` or `curl -F`)
  - `GET /api/workspaces/{ws_id}/api-mock/projects/{task_id}/jobs/{job_id}` (poll until terminal)
  - `GET /api/workspaces/{ws_id}/api-mock/projects/{task_id}/endpoints`
  - `POST /api/workspaces/{ws_id}/api-mock/projects/{task_id}/endpoints/{endpoint_id}/auto-mock` (generate endpoint cases)
  - `GET /api/workspaces/{ws_id}/api-mock/endpoints/{endpoint_id}/mock-cases` (verify coverage)
  - `GET /api/workspaces/{ws_id}/api-mock/projects/{task_id}/context` (read canonical `mock_base_url`)
  - Use concrete tool invocations (`curl`, `Invoke-RestMethod`, or equivalent), not pseudo "call" text
  - Contract artifact file inputs must be absolute paths
  - `api_mock_contract_sync.py` is deprecated and must not appear in plan steps
  - Plan must state that API MOCK runtime context (`API_BASE_URL`, `ACCESS_TOKEN`, `WORKSPACE_ID`, `TASK_ID`) comes from platform injection, not task-repo config
  - Plan must forbid searching task repository files for API MOCK configuration
  - Plan must include preflight checks and explicit stop condition when runtime inputs (`API_BASE_URL`, `ACCESS_TOKEN`) or contract files are missing
  - Execution evidence must include HTTP method, URL path, status code, and key JSON fields

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

**Layered mode headers (full-stack only):**

`api_plan.md`
```markdown
# [Feature Name] API Plan

> **For agentic workers:** This contract is the source of truth for downstream frontend/backend plans.

**Goal:** [Contract scope this API plan defines]

**Contract Version:** [v1 / date / revision tag]

---
```

`frontend_plan.md`
```markdown
# [Feature Name] Frontend Plan

> **Dependency:** This plan depends on `api_plan.md`. Do not redefine API schemas here.

**Goal:** [UI/UX and client behavior scope]

**Contract Input:** [Path to api_plan.md]

---
```

`backend_plan.md`
```markdown
# [Feature Name] Backend Plan

> **Dependency:** This plan depends on `api_plan.md`. Do not redefine API schemas here.

**Goal:** [Server implementation scope]

**Contract Input:** [Path to api_plan.md]

---
```

## Task Structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## No Placeholders

Every step must contain the actual content an engineer needs. These are **plan failures** — never write them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" (without actual test code)
- "Similar to Task N" (repeat the code — the engineer may be reading tasks out of order)
- Steps that describe what to do without showing how (code blocks required for code steps)
- References to types, functions, or methods not defined in any task

## Remember
- Exact file paths always
- Complete code in every step — if a step changes code, show the code
- Exact commands with expected output
- DRY, YAGNI, TDD, frequent commits

## Self-Review

After writing the complete plan, look at the spec with fresh eyes and check the plan against it. This is a checklist you run yourself — not a subagent dispatch.

**1. Spec coverage:** Skim each section/requirement in the spec. Can you point to a task that implements it? List any gaps.

**2. Placeholder scan:** Search your plan for red flags — any of the patterns from the "No Placeholders" section above. Fix them.

**3. Type consistency:** Do the types, method signatures, and property names you used in later tasks match what you defined in earlier tasks? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

If you find issues, fix them inline. No need to re-review — just fix and move on. If you find a spec requirement with no task, add the task.

## Execution Handoff

After saving the plan:

**If this is layered full-stack mode:**

Use this handoff exactly:

**"Layered plans complete and saved to `docs/superpowers/plans/<feature-folder>/` with `api_plan.md`, `frontend_plan.md`, and `backend_plan.md`.**

**Next step is REQUIRED: execute with `superpowers:subagent-driven-development` so the API contract gate (OpenAPI generation + platform sync lock) runs before frontend/backend implementation."**

Do not offer inline `executing-plans` for layered full-stack mode.

**If this is single-plan mode, offer execution choice:**

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Two execution options:**

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?"**

**If Subagent-Driven chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development
- Fresh subagent per task + two-stage review

**If Inline Execution chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:executing-plans
- Batch execution with checkpoints for review
