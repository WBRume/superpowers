# Plan Document Reviewer Prompt Template

Use this template when dispatching a plan document reviewer subagent.

**Purpose:** Verify the plan is complete, matches the spec, has proper task decomposition, and preserves contract dependencies.

**Dispatch after:** The complete plan is written.

```
Task tool (general-purpose):
  description: "Review plan document"
  prompt: |
    You are a plan document reviewer. Verify this plan set is complete and ready for implementation.

    **Plan to review:** [PLAN_FILE_PATH or PLAN_DIRECTORY_PATH]
    **Layered plans (if full-stack):**
    - API plan: [API_PLAN_FILE_PATH]
    - Frontend plan: [FRONTEND_PLAN_FILE_PATH]
    - Backend plan: [BACKEND_PLAN_FILE_PATH]
    **Spec for reference:** [SPEC_FILE_PATH]

    ## What to Check

    | Category | What to Look For |
    |----------|------------------|
    | Completeness | TODOs, placeholders, incomplete tasks, missing steps |
    | Spec Alignment | Plan covers spec requirements, no major scope creep |
    | Task Decomposition | Tasks have clear boundaries, steps are actionable |
    | Contract Dependency | Frontend/backend plans only use API fields/types from `api_plan.md` |
    | Buildability | Could an engineer follow this plan without getting stuck? |

    ## Layered Mode Gate (full-stack only)

    If this is layered mode, verify:
    1. `api_plan.md` exists and is complete before downstream plans
    2. `frontend_plan.md` and `backend_plan.md` both explicitly depend on `api_plan.md`
    3. No endpoint/field/type appears in frontend/backend plans unless defined in `api_plan.md`
    4. The execution handoff requires `subagent-driven-development` for contract gating
    5. Frontend/backend plans both contain automated test tasks (not manual-only verification)
    6. If frontend is Vite, plan includes mock-baseline proxy target update in `vite.config.*`
    7. Plan includes explicit real API-call steps for import/poll/endpoints/auto-mock/mock-cases/context:
       - `POST /api/workspaces/{ws_id}/api-mock/projects/{task_id}/swagger/import`
         with `multipart/form-data` (`raw_content`/`file`/`source_url`, not JSON body)
       - `GET /api/workspaces/{ws_id}/api-mock/projects/{task_id}/jobs/{job_id}`
       - `GET /api/workspaces/{ws_id}/api-mock/projects/{task_id}/endpoints`
       - `POST /api/workspaces/{ws_id}/api-mock/projects/{task_id}/endpoints/{endpoint_id}/auto-mock`
       - `GET /api/workspaces/{ws_id}/api-mock/endpoints/{endpoint_id}/mock-cases`
       - `GET /api/workspaces/{ws_id}/api-mock/projects/{task_id}/context`
    8. Plan requires absolute-path contract artifact inputs and API-call evidence capture (`method/path/status_code` + key JSON fields)
    9. Plan includes preflight checks with a hard stop (`BLOCKED`) when runtime inputs or required files are missing
    10. Vue/Vite frontend plan includes concrete test framework bootstrap (if missing) and at least one API-layer + one UI behavior test task

    ## Calibration

    **Only flag issues that would cause real problems during implementation.**
    An implementer building the wrong thing or getting stuck is an issue.
    Minor wording, stylistic preferences, and "nice to have" suggestions are not.

    Approve unless there are serious gaps - missing requirements from the spec,
    contradictory steps, placeholder content, or tasks so vague they can't be acted on.

    ## Output Format

    ## Plan Review

    **Status:** Approved | Issues Found

    **Issues (if any):**
    - [Task X, Step Y]: [specific issue] - [why it matters for implementation]

    **Recommendations (advisory, do not block approval):**
    - [suggestions for improvement]
```

**Reviewer returns:** Status, Issues (if any), Recommendations
