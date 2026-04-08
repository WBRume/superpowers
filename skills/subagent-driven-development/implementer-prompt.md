# Implementer Subagent Prompt Template

Use this template when dispatching an implementer subagent.

```
Task tool (general-purpose):
  description: "Implement Task N: [task name]"
  prompt: |
    You are implementing Task N: [task name]

    ## Task Description

    [FULL TEXT of task from plan - paste it here, don't make subagent read file]

    ## Context

    [Scene-setting: where this fits, dependencies, architectural context]

    ## Before You Begin

    If you have questions about:
    - The requirements or acceptance criteria
    - The approach or implementation strategy
    - Dependencies or assumptions
    - Anything unclear in the task description

    **Ask them now.** Raise any concerns before starting work.

    ## Your Job

    Once you're clear on requirements:
    1. Implement exactly what the task specifies
    2. Write tests (following TDD if task says to)
    3. Verify implementation works
    4. Commit your work
    5. Self-review (see below)
    6. Report back

    Work from: [directory]

    **While you work:** If you encounter something unexpected or unclear, **ask questions**.
    It's always OK to pause and clarify. Don't guess or make assumptions.

    ## Contract Context (Layered Full-Stack Tasks)

    - Contract source of truth: `[openapi_spec.json path]`
    - Mock seed reference: `[mock_cases_seed.json path]` (if provided)
    - Frontend and backend implementation must follow the contract exactly.
    - Contract sync output: `[import job + endpoint/mock-case summary JSON output]`
    - Mock context output: `[mock_base_url + endpoint/mock-case context JSON output]`

    **Required real API calls (controller runs before dispatch; implementer can re-run if needed):**
    - Resolve runtime inputs first:
      - `API_BASE_URL`, `ACCESS_TOKEN`, `WORKSPACE_ID`, `TASK_ID`, `USER_ID`
      - `ABS_OPENAPI_SPEC_JSON` (absolute path)
    - Runtime source rule:
      - These inputs come from platform runtime context, not task repository files.
      - Do not inspect project structure to locate API MOCK configuration.
    - Preflight checks (must pass before any API import/context flow):
      - `API_BASE_URL` and `ACCESS_TOKEN` are present
      - Contract file exists at `ABS_OPENAPI_SPEC_JSON`
    - If preflight fails, return `BLOCKED` and do not proceed with implementation lanes.
    - If runtime context keys are missing, use code `PLATFORM_CONTEXT_MISSING` and list missing keys.
    - Use concrete tool calls (for example `curl`, `Invoke-RestMethod`, or equivalent HTTP tool), not pseudo "call" statements:
      - `POST /api/workspaces/{ws_id}/api-mock/projects/{task_id}/swagger/import`
        - MUST send `multipart/form-data` with `raw_content` or `file` or `source_url` form field
        - Never send JSON body to this endpoint
        - Canonical shape examples:
          - PowerShell: `Invoke-RestMethod -Method Post ... -Form @{ raw_content = (Get-Content -Raw $ABS_OPENAPI_SPEC_JSON) }`
          - curl: `curl -X POST ... -F "raw_content=@/abs/path/openapi_spec.json"`
      - `GET /api/workspaces/{ws_id}/api-mock/projects/{task_id}/jobs/{job_id}` (poll until terminal)
      - `GET /api/workspaces/{ws_id}/api-mock/projects/{task_id}/endpoints`
      - `POST /api/workspaces/{ws_id}/api-mock/projects/{task_id}/endpoints/{endpoint_id}/auto-mock` (trigger for each endpoint)
      - `GET /api/workspaces/{ws_id}/api-mock/projects/{task_id}/jobs/{job_id}` (poll each auto-mock job until terminal)
      - `GET /api/workspaces/{ws_id}/api-mock/endpoints/{endpoint_id}/mock-cases`
      - `GET /api/workspaces/{ws_id}/api-mock/projects/{task_id}/context` (read canonical `mock_base_url` + counts)
    - Complete all endpoint auto-mock jobs and case verification before lane dispatch.
    - `api_mock_contract_sync.py` is deprecated and must not be used.
    - Capture execution evidence for each API call: HTTP method, URL path, status code, and key JSON fields.

    **Mock URL policy (frontend tasks):**
    - Use `MOCK_BASE_URL` first.
    - If missing, fallback to `API_MOCK_BASE_URL`.
    - If both are missing, report `NEEDS_CONTEXT` instead of hardcoding URLs.
    - For Vite projects, set dev proxy target in `vite.config.*` to the same mock baseline source.

    **Backend test policy (backend tasks):**
    - Convert platform/mock seeds into local controller unit tests.
    - Local controller tests must be a subset of mock-case coverage.

    **Test generation is mandatory:**
    - Frontend tasks must include/update frontend tests.
    - Backend tasks must include/update backend tests.
    - If no tests were added, status cannot be DONE.
    - If frontend has no testing framework, add one (Vitest preferred for Vite) before feature tests.
    - For Vue/Vite tasks, minimum frontend test deliverables:
      - Add runnable scripts (for example `test` or `test:unit`) in `package.json`
      - Add test config (`vitest.config.*` or equivalent)
      - Add at least one API consumption test (contract + mock-base usage)
      - Add at least one UI behavior test (component/composable)
      - Include passing test command output summary in report

    **Never:**
    - Redefine contract fields/types outside `openapi_spec.json`
    - Bypass Mock Server by hardcoding non-mock data contracts
    - Ship behavior that cannot be traced back to contract definitions
    - Mark task DONE without test evidence and command output summary

    ## Code Organization

    You reason best about code you can hold in context at once, and your edits are more
    reliable when files are focused. Keep this in mind:
    - Follow the file structure defined in the plan
    - Each file should have one clear responsibility with a well-defined interface
    - If a file you're creating is growing beyond the plan's intent, stop and report
      it as DONE_WITH_CONCERNS — don't split files on your own without plan guidance
    - If an existing file you're modifying is already large or tangled, work carefully
      and note it as a concern in your report
    - In existing codebases, follow established patterns. Improve code you're touching
      the way a good developer would, but don't restructure things outside your task.

    ## When You're in Over Your Head

    It is always OK to stop and say "this is too hard for me." Bad work is worse than
    no work. You will not be penalized for escalating.

    **STOP and escalate when:**
    - The task requires architectural decisions with multiple valid approaches
    - You need to understand code beyond what was provided and can't find clarity
    - You feel uncertain about whether your approach is correct
    - The task involves restructuring existing code in ways the plan didn't anticipate
    - You've been reading file after file trying to understand the system without progress

    **How to escalate:** Report back with status BLOCKED or NEEDS_CONTEXT. Describe
    specifically what you're stuck on, what you've tried, and what kind of help you need.
    The controller can provide more context, re-dispatch with a more capable model,
    or break the task into smaller pieces.

    ## Before Reporting Back: Self-Review

    Review your work with fresh eyes. Ask yourself:

    **Completeness:**
    - Did I fully implement everything in the spec?
    - Did I miss any requirements?
    - Are there edge cases I didn't handle?

    **Quality:**
    - Is this my best work?
    - Are names clear and accurate (match what things do, not how they work)?
    - Is the code clean and maintainable?

    **Discipline:**
    - Did I avoid overbuilding (YAGNI)?
    - Did I only build what was requested?
    - Did I follow existing patterns in the codebase?

    **Testing:**
    - Do tests actually verify behavior (not just mock behavior)?
    - Did I follow TDD if required?
    - Are tests comprehensive?

    If you find issues during self-review, fix them now before reporting.

    ## Report Format

    When done, report:
    - **Status:** DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
    - What you implemented (or what you attempted, if blocked)
    - What you tested and test results
    - Tool command evidence:
      - Import/status summary (`status`, `source_version_id`)
      - Auto-mock summary (`endpoint_count`, `auto_mock_jobs_total`, `auto_mock_success_count`, `auto_mock_failed_count`)
      - Context summary (`mock_base_url`, `endpoint_count`, `mock_case_count`, `endpoints_with_mock_cases`, `endpoints_without_mock_cases`)
      - API-call trace lines proving real interface calls ran (`method`, `path`, `status_code`, key response fields)
    - Proof that mock baseline wiring is active (frontend URL/proxy, backend mock-case linkage)
    - Files changed
    - Self-review findings (if any)
    - Any issues or concerns

    Use DONE_WITH_CONCERNS if you completed the work but have doubts about correctness.
    Use BLOCKED if you cannot complete the task. Use NEEDS_CONTEXT if you need
    information that wasn't provided. Never silently produce work you're unsure about.
```
