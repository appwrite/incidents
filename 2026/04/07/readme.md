# Auth Session Failures Following Platform Update

- **Incident Start:** 2026-04-07 11:05 UTC
- **Incident End:** 2026-04-07 18:53 UTC
- **Report Prepared By:** Chirag Aggarwal

## Summary

Following a platform update that migrated the dependency injection container and updated the underlying HTTP layer, a subset of users using React Native, Flutter, and Swift SDKs experienced intermittent authentication failures when logging in via email/password. The update was deployed progressively across regions starting at 11:05 UTC. User reports surfaced at 17:56 UTC, and the issue was resolved by reverting to the previous stable release at 18:53 UTC.

The root cause was a cookie handling bug where URL-encoded session cookie values were not properly decoded before processing. Mobile clients following the RFC 6265 cookie standard reflected encoded values verbatim, causing valid sessions to be silently rejected. The changes were quickly reverted to restore service, and a permanent fix was developed and merged the following day.

## Incident details

### Initial detection

At 17:56 UTC, user reports indicated authentication failures - specifically 401 errors when creating sessions in mobile applications. The team investigated telemetry dashboards, which showed no obvious error spikes, making initial diagnosis challenging.

### Affected components

- Email/password session creation flow.
- Session cookie handling in the HTTP layer.
- Mobile SDK authentication (React Native, Flutter, Swift).

### User impact

- Users experienced 401 errors when logging in via email/password on mobile applications using RFC 6265-compliant clients (Flutter, Swift, React Native).
- Sessions were created successfully, but subsequent requests failed because the session cookie value was corrupted during server-side parsing.
- The issue affected users whose account IDs produced certain characters in the base64-encoded session cookie.

## Root cause analysis

### Preliminary findings

Initial investigation focused on identifying which of the recent platform changes was responsible, as multiple updates had been deployed in close succession.

### Investigation

The initial investigation explored a cache ordering issue in the session creation handler, which appeared plausible but did not resolve the reported failures. Continued investigation identified the actual root cause - a cookie handling bug in the HTTP framework layer.

The server URL-encoded base64 session values when setting cookies. Mobile clients following the RFC 6265 standard stored and reflected these encoded values verbatim. However, the server-side cookie parser did not URL-decode the values before processing, causing the base64 decoder to receive corrupted input and silently reject valid sessions.

Existing end-to-end tests did not catch this because the test client inadvertently URL-decoded cookie values, compensating for the server-side bug.

### Root cause

A cookie decoding bug in the HTTP framework layer. URL-encoded session cookie values were not decoded before processing, causing RFC 6265-compliant mobile clients to have their valid sessions rejected. The recent platform update changed the underlying HTTP library dependency, exposing this pre-existing bug.

## Resolution and recovery

### Immediate actions

- Investigated telemetry and error dashboards to assess scope.
- Identified the recent platform update as the likely trigger.
- Reverted to the previous stable release at 18:53 UTC, resolving the user-facing issue.

### Resolution

After the initial fix attempt did not resolve the issue, the actual fix was developed and merged on April 8. The HTTP framework was updated to use proper URL-decoded cookie values, and the end-to-end test client was updated to match real client behavior to prevent future test-masking. A cookie roundtrip assertion was also added to verify correct behavior.

## Lessons learned

### What went well

- Progressive region-by-region deployment limited initial exposure.
- Quick response to user reports and prompt revert to stable release.
- Thorough root cause investigation and permanent fix with regression tests produced the next day.
- Team correctly identified the initial fix was insufficient and continued investigating until the real root cause was found.

### What can be improved

- Telemetry did not clearly surface the authentication failures during initial investigation.
- Multiple concurrent releases complicated triage.
- The test client inadvertently masked the bug by compensating for the server-side cookie handling issue.
- The initial investigation led to a plausible but incorrect fix before the actual root cause was identified.

### Action items

- [x] Update E2E test client to use RFC 6265-compliant cookie handling, matching real client behavior.
- [ ] Improve test coverage around mobile framework cookie handling to better reflect React Native, Flutter, and Swift client behavior.

## Additional resources

### Supporting documentation

- [Better Stack incident](https://status.appwrite.online/incident/866991)
- [Fix PR](https://github.com/appwrite/appwrite/pull/11826)
