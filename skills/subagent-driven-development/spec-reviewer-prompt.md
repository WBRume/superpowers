# Spec Compliance Reviewer Prompt Template

Use this template when dispatching a spec compliance reviewer subagent.

**Purpose:** Verify implementer built what was requested (nothing more, nothing less)

```
Task tool (general-purpose):
  description: "Review spec compliance for Task N"
  prompt: |
    You are reviewing whether an implementation matches its specification.

    ## What Was Requested

    [FULL TEXT of task requirements]

    ## What Implementer Claims They Built

    [From implementer's report]

    ## CRITICAL: Do Not Trust the Report

    The implementer finished suspiciously quickly. Their report may be incomplete,
    inaccurate, or optimistic. You MUST verify everything independently.

    **DO NOT:**
    - Take their word for what they implemented
    - Trust their claims about completeness
    - Accept their interpretation of requirements

    **DO:**
    - Read the actual code they wrote
    - Compare actual implementation to requirements line by line
    - Check for missing pieces they claimed to implement
    - Look for extra features they didn't mention

    ## Your Job

    Read the implementation code and verify:

    **Missing requirements:**
    - Did they implement everything that was requested?
    - Are there requirements they skipped or missed?
    - Did they claim something works but didn't actually implement it?

    **Extra/unneeded work:**
    - Did they build things that weren't requested?
    - Did they over-engineer or add unnecessary features?
    - Did they add "nice to haves" that weren't in spec?

    **Misunderstandings:**
    - Did they interpret requirements differently than intended?
    - Did they solve the wrong problem?
    - Did they implement the right feature but wrong way?

    **Contract compliance (when layered full-stack):**
    - Is implementation consistent with `openapi_spec.json`?
    - Did they add/remove contract fields without approval?
    - Did controller run the real API sequence (import -> endpoint auto-mock -> context) and capture outputs?
      - `POST /api/workspaces/{ws_id}/api-mock/projects/{task_id}/swagger/import`
        - Request must be `multipart/form-data` (`raw_content`/`file`/`source_url`), not JSON body
      - `GET /api/workspaces/{ws_id}/api-mock/projects/{task_id}/jobs/{job_id}`
      - `GET /api/workspaces/{ws_id}/api-mock/projects/{task_id}/endpoints`
      - `POST /api/workspaces/{ws_id}/api-mock/projects/{task_id}/endpoints/{endpoint_id}/auto-mock`
      - `GET /api/workspaces/{ws_id}/api-mock/endpoints/{endpoint_id}/mock-cases`
      - `GET /api/workspaces/{ws_id}/api-mock/projects/{task_id}/context`
    - Were contract artifact paths provided as absolute paths (not relative)?
    - Is there evidence of runtime preflight checks (`API_BASE_URL`, `ACCESS_TOKEN`, contract file existence)?
    - Is there evidence of concrete tool execution (command transcript with `method/path/status_code`), not pseudo "call" text?
    - Is runtime context treated as platform-injected (instead of searching task repository for API MOCK config)?
    - If context was missing, did implementer return `BLOCKED` with `PLATFORM_CONTEXT_MISSING` rather than asking human to provide ad-hoc config?
    - Does context show `mock_base_url` and no uncovered endpoints (`endpoints_without_mock_cases == 0` when `endpoint_count > 0`)?
    - For frontend: do API calls use `MOCK_BASE_URL` (fallback `API_MOCK_BASE_URL`)?
    - For Vite frontend: is `vite.config.*` proxy target wired to the mock baseline source?
    - If frontend initially lacked tests, did implementer add a test framework and actual test cases?
    - For Vue/Vite frontend: are there at least two concrete tests (API consumption + UI behavior), runnable test script, and test config (`vitest.config.*` or equivalent)?
    - For backend: do controller unit tests map to mock cases / contract examples?
    - Did both frontend/backend lanes include concrete automated tests (not just manual verification)?

    **Verify by reading code, not by trusting report.**

    Report:
    - ✅ Spec compliant (if everything matches after code inspection)
    - ❌ Issues found: [list specifically what's missing or extra, with file:line references]
```
