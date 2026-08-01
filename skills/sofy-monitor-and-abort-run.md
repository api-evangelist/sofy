---
name: Monitor and abort a scheduled run
description: >-
  Enumerate the test run groups for a SOFY scheduled run, check their status,
  and abort an in-progress run when needed.
api: openapi/sofy-public-api-openapi.yml
operations:
  - fetchTestRunGroupIds
  - getScheduledRunStatus
  - abortScheduledRun
---

# Monitor and abort a scheduled run

Manage an in-flight SOFY scheduled run over `https://public.sofy.ai`. Every
request carries `x-sofy-auth-key` (see `authentication/sofy-authentication.yml`).

## Steps

1. **List test run groups** — `fetchTestRunGroupIds`
   `GET /scheduler-microservice/scheduled-runs/{scheduledRunGuid}/test-run-groups`.
   Returns each `testRunGroupId` with `executedAt` and `status` (`Queued`,
   `Running`, `Complete`, `Complete with errors`).

2. **Check detailed status** — `getScheduledRunStatus`
   `GET /scheduler-microservice/scheduled-runs/{scheduledRunGuid}/status/{testRunGroupId}`
   for the group you care about (returns `scenarioName`, `build`, `device`,
   `status`).

3. **Abort if needed** — `abortScheduledRun`
   `PUT /scheduler-microservice/public/test-run/abort/{scheduledRunGuid}?testRunGroupId={id}`.
   Omit `testRunGroupId` to abort **all** active executions under the scheduled
   run. Success returns a message confirming the run was aborted.

## Notes
- Only `Queued`/`Running` groups are abortable; already-`Complete` groups are
  terminal.
- Errors follow the `{ "error": { message, details, timestamp } }` envelope
  (`errors/sofy-problem-types.yml`).
