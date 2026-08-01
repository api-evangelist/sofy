---
name: Upload a build and run scheduled tests
description: >-
  Upload an application build to SOFY, trigger a pre-configured scheduled test
  run against it on real devices, and poll the run to completion.
api: openapi/sofy-public-api-openapi.yml
operations:
  - uploadApplication
  - triggerScheduledRun
  - getScheduledRunStatus
---

# Upload a build and run scheduled tests

Use the SOFY Public API (`https://public.sofy.ai`) to push a new build into a
CI/CD test cycle. Every request carries the header `x-sofy-auth-key` with the
API access key generated under Account Settings > API Key
(see `authentication/sofy-authentication.yml`).

## Steps

1. **Upload the build** — `uploadApplication`
   `POST /parser-microservice/build-upload?userFriendlyBuildName={name}` with the
   APK/IPA as multipart `applicationFile`. Capture `data.appHash` from the 200
   response — you need it to run tests. A missing `x-sofy-auth-key` returns
   `400 Missing required headers`; wrong file type returns `500 Files with
   extensions other than APK or IPA not supported`
   (see `errors/sofy-problem-types.yml`).

2. **Trigger the scheduled run** — `triggerScheduledRun`
   `POST /scheduler-microservice/scheduled-runs/{scheduledGUID}/execute?appHash={appHash}&deviceSerials={serials}`.
   No request body. On success you get `testGroupsRunId` and the message
   "Test runs are scheduled and in queue."

3. **Poll status** — `getScheduledRunStatus`
   `GET /scheduler-microservice/scheduled-runs/{scheduledRunGuid}/status/{testRunGroupId}`.
   Repeat until `status` is a terminal value (`Passed`, `Failed`, `Stopped`, or
   `Not Executed`); `Running` means keep polling.

## Notes
- The API is unversioned and documents no idempotency-key mechanism
  (`conventions/sofy-conventions.yml`), so avoid duplicate uploads — a repeat
  build returns `400 Build already exists`.
- Completion can also be delivered via webhook (`asyncapi/sofy-webhooks.yml`)
  instead of polling.
