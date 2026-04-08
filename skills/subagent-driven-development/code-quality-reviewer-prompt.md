# Code Quality Reviewer Prompt Template

Use this template when dispatching a code quality reviewer subagent.

**Purpose:** Verify implementation is well-built (clean, tested, maintainable)

**Only dispatch after spec compliance review passes.**

```
Task tool (superpowers:code-reviewer):
  Use template at requesting-code-review/code-reviewer.md

  WHAT_WAS_IMPLEMENTED: [from implementer's report]
  PLAN_OR_REQUIREMENTS: Task N from [plan-file]
  BASE_SHA: [commit before task]
  HEAD_SHA: [current commit]
  DESCRIPTION: [task summary]
```

**In addition to standard code quality concerns, the reviewer should check:**
- Does each file have one clear responsibility with a well-defined interface?
- Are units decomposed so they can be understood and tested independently?
- Is the implementation following the file structure from the plan?
- Did this implementation create new files that are already large, or significantly grow existing files? (Don't flag pre-existing file sizes — focus on what this change contributed.)
- Is the implementation contract-locked to `openapi_spec.json` (no schema drift)?
- Are real API outputs present and internally consistent for import -> endpoint auto-mock -> context?
  - `POST /api/workspaces/{ws_id}/api-mock/projects/{task_id}/swagger/import`
    - Request format must be `multipart/form-data` (`raw_content`/`file`/`source_url`)
  - `GET /api/workspaces/{ws_id}/api-mock/projects/{task_id}/jobs/{job_id}`
  - `GET /api/workspaces/{ws_id}/api-mock/projects/{task_id}/endpoints`
  - `POST /api/workspaces/{ws_id}/api-mock/projects/{task_id}/endpoints/{endpoint_id}/auto-mock`
  - `GET /api/workspaces/{ws_id}/api-mock/endpoints/{endpoint_id}/mock-cases`
  - `GET /api/workspaces/{ws_id}/api-mock/projects/{task_id}/context`
- Are API-call trace lines present and consistent with JSON outputs (`method`, `path`, `status_code`, key response fields)?
- Are contract artifact paths absolute (not relative)?
- Is runtime preflight verification present (`API_BASE_URL`, `ACCESS_TOKEN`, required contract files)?
- Did preflight checks prevent execution when required runtime inputs or contract paths were missing?
- Is there no repository-scan dependency for API MOCK configuration discovery?
- If runtime context was missing, is failure explicit as `PLATFORM_CONTEXT_MISSING` (not converted into a human config questionnaire)?
- Does context prove full endpoint coverage (`endpoints_without_mock_cases == 0` when `endpoint_count > 0`) before parallel lanes start?
- For frontend tasks: is API base URL derived from `MOCK_BASE_URL` with `API_MOCK_BASE_URL` fallback?
- For Vite frontend tasks: is proxy target derived from mock baseline configuration (not hardcoded unrelated backend host)?
- If frontend had no test framework, did the change add one and wire tests into runnable scripts?
- For Vue/Vite frontend tasks: do tests include both API-layer assertion and UI behavior assertion, with runnable command evidence and explicit test config (`vitest.config.*` or equivalent)?
- For backend tasks: do controller tests clearly trace to mock cases or contract examples?
- Are automated tests present for both lanes and meaningful for changed behavior?

**Code reviewer returns:** Strengths, Issues (Critical/Important/Minor), Assessment
