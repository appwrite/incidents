# SDK Type Mismatch: $sequence Field Returned as String Instead of Integer

- **Incident Start:** 2026-02-25 06:31 UTC
- **Incident End:** 2026-02-25 08:37 UTC
- **Report Prepared By:** Jake Barnby

## Summary

A deployment to Appwrite Cloud introduced a change that caused the API to return the `$sequence` field as a JSON string (e.g. `"42"`) instead of a JSON integer (e.g. `42`). The Android, Apple, and Flutter SDKs perform strict type assignments in their `Document.fromMap` methods with no type coercion, so when a string arrived where an `int` was expected, Dart threw `type 'String' is not a subtype of type 'int'`. This crash affected every `listDocuments` and `getDocument` call, cascading into provider failures across all apps using these SDKs.

The incident occurred in two windows due to a second, uncoordinated deployment. Both times the fix was reverting to an earlier release.

## Incident details

### Initial detection

The issue was first detected at 06:31 UTC when users reported crashes in apps using the Android, Apple, and Flutter SDKs.

### Affected components

- Appwrite Cloud API response serialization
- Android SDK
- Apple SDK
- Flutter SDK

### User impact

All apps using the Android, Apple, and Flutter SDKs experienced crashes on any `listDocuments` or `getDocument` call. This effectively broke all document read operations for affected SDK users.

## Root cause analysis

### Preliminary findings

The `$sequence` field in API responses was being returned as a JSON string instead of a JSON integer, causing type errors in SDK deserialization.

### Investigation

A recent merge introduced support for switching the `$sequence` field between string and integer types depending on the underlying database in use. A bug in the switching logic meant the type defaulted to string regardless of the database. Additionally, the changes introduced a gap in the test suite where type assertions were not correctly validating the `$sequence` type based on whether int or string was supported by the database.

### Root cause

The switching logic for the `$sequence` field type did not work correctly, causing the field to always be serialized as a string. The accompanying test modifications for the new feature failed to preserve existing assertions that would have caught the incorrect type.

## Resolution and recovery

### Immediate Actions

At 06:31 UTC, the issue was identified and the deployment was reverted to an earlier release, restoring correct behavior by 07:32 UTC.

### Reoccurrence

At 08:07 UTC, a second team member, unaware of the ongoing incident, deployed again with the same changes, reintroducing the issue. The deployment was reverted again, and the incident was fully resolved by 08:37 UTC.

## Lessons learned

### What went well

1. Quick identification of the root cause and fast rollback both times

### What can be improved

1. Better internal communication during active incidents to prevent uncoordinated deployments
2. More comprehensive type-level testing for API response fields, especially at boundaries where types can break down in loose languages (e.g. numeric strings)
3. When adapting existing tests for new features, all pre-existing assertions must be verified to still be intact

### Action items

1. Add tests that explicitly validate response field types (int vs string) for fields like `$sequence` across all supported database backends
2. Establish an incident communication protocol to ensure all team members are aware of active incidents before deploying
3. Review test modification practices to ensure new feature tests do not remove or weaken existing type assertions
