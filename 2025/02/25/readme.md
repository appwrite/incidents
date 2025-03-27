# Incorrectly handled audit logs

- **Incident Start:** 2025-02-23 17:57 UTC
- **Incident End:** 2025-02-26 01:50 UTC
- **Report Prepared By:** Steven Nguyen

## Summary

On February 23rd, 2025, at 17:57 UTC, an incident occurred where audit logs for different users appeared in the incorrect place. To reduce confusion and prevent showing other users' logs, audit logs were disabled.

After some investigation, we discovered that the issue was caused by the batched writes as documents were inserted into the incorrect project.

After we solved the issue, we re-enabled audit logs.

## Incident details

### Initial detection

<!-- How and when the issue was first detected. -->

We first detected the issue on February 23rd, 2025, at 17:57 UTC, when users reported seeing logs from other users.

### Affected components

<!-- List of affected services, features, or infrastructure components. -->

Only the audit logs were affected.

### User impact

<!-- Description of how the incident impacted end users, including any potential data loss or service unavailability. -->

Since we had to disable audit logs, users were unable to view any audit logs for their account or in their projects.

The audit logs that ended up in the wrong place couldn't be put back in the right place so they were essentially lost.

## Root cause analysis

### Preliminary findings

<!-- Initial insights into what caused the incident. -->

Our initial theory was that the audit logs were being stored in the wrong project due to a bug in the audit batch writing process.

### Investigation

<!-- Detailed explanation of the investigation process, tools used, and key findings. -->

By digging into the [code](https://github.com/appwrite/appwrite/blob/392b33ef269670035e2062475119d059b298d9fb/src/Appwrite/Platform/Workers/Audits.php#L117-L120), we were able to confirm that the batch of logs were being inserted into the project that triggered the worker last.

### Root cause

<!-- Concise statement of the primary cause of the incident. -->

Because inserts were made into the incorrect project, users saw incorrect data if the resource or user matched up with the audit log that was inserted. For example, there was a user event log for a user with the same ID as my user that was inserted into the console project so that's why I saw that user's audit log when looking at my own audit logs.

## Resolution and recovery

### Immediate Actions

<!-- Steps taken to mitigate the impact and restore service. -->

To prevent the wrong data from being returned, we [updated all the log endpoints to return nothing](https://github.com/appwrite/appwrite/pull/9398) for the duration of the incident.

### Resolution

<!-- How and when was the incident fully resolved? -->

To solve the problem, we [fixed the batching](https://github.com/appwrite/appwrite/pull/9403) so that it's batched by project and then re-enabled the log endpoints.

We also skipped over any logs for the period the issue occurred to prevent any incorrect data from being returned.

## Lessons learned

### What went well

<!-- Aspects of the incident response that were effective. -->

* The team acted quickly to identify the root cause and mitigate the impact of the problem.
* Status updates were provided regularly to keep developers updated.

### What can be improved

<!-- Areas where the response could have been better or faster. -->

* We need to be able to publish the incident reports faster.

### Action items

<!-- Specific steps to be taken to prevent recurrence, improve detection, or enhance response. -->
* Planned improvements to our logging system.

## Additional resources

* [Better Stack incident](https://status.appwrite.online/incident/517695)
* Related PRs:
  * https://github.com/appwrite/appwrite/pull/9398
  * https://github.com/appwrite/appwrite/pull/9403
