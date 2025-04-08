# Schedules stopped working

- **Incident Start:** 2025-03-25 20:09 UTC
- **Incident End:** 2025-03-25 21:38 UTC
- **Report Prepared By:** Steven Nguyen

## Summary

On March 25, 2025 at 20:09 UTC, we responded to an incident in which cron-scheduled functions were not executing.

After an initial investigation, we discovered the database query in schedule workers filtered out new resources due to region changes made in preparation for future releases. This caused delayed executions, (CRON) scheduled exeuctions, scheduled (push/email/SMS) messages, and backups to not work for some of the projects.

## Incident details

### Initial detection

We were notified when a [community member reported](https://discord.com/channels/564160730845151244/1354120595100401684) a new function wasn't running based on the cron scheduled configured despite older functions they had working as expected.

### Affected components

Although the problem was first raised for a cron-scheduled function, we realized the problem occurred for everything using a schedule worker including scheduled messages, backups, and cron-scheduled functions.

### User impact

Only newly created scheduled messages, backup policies, and functions were affected.

## Root cause analysis

### Preliminary findings

By looking at the database and the ScheduleBase code, we found the region in the database was `fra`, but the code was looking for schedules with a region of `default`. As such, it found no schedules to keep track of and execute.

### Investigation

Looking into the database, we saw the region for newly created schedules was `fra`.

However, the [code](https://github.com/appwrite/appwrite/blob/4804bdf110631661c2c907411cd3a0c8acc4fc96/src/Appwrite/Platform/Tasks/ScheduleBase.php#L106-L110) looked for schedules filtered based on region:

```php
$results = $dbForPlatform->find('schedules', \array_merge($paginationQueries, [
    Query::equal('region', [System::getEnv('_APP_REGION', 'default')]),
    Query::equal('resourceType', [static::getSupportedResource()]),
    Query::equal('active', [true]),
]));
```

and the `_APP_REGION`, at the time, was set to `default`.

### Root cause

Since the worker was looking for schedules with a region of `default`, but the region was set to `fra`, it found no schedules to keep track of and execute.

## Resolution and recovery

### Immediate Actions

To fix the issue, we [updated](https://github.com/appwrite/appwrite/pull/9577) all instances where we checked the region to include default if it wasn't in the list of regions.

In addition, we updated the `_APP_REGION` to `fra`.

### Resolution

Once the PRs were merged and deployed, the worker was able to find the schedules and execute them successfully.

## Lessons learned

### What went well

* quick analysis of the issue to find the root cause
* quick resolution of the issue

### What can be improved

* quicker (ideally, automated) discovery of the problem as the user created the [post](https://discord.com/channels/564160730845151244/1354120595100401684) at 15:51 UTC, but we didn't see it as a big issue until 20:08 UTC.
* better testing to catch the problem earlier

### Action items

* clean up region in all stats, projects, schedules, and rules collections to replace `default` with `fra`
* review code to ensure we're not using default region and we're using region correctly

## Additional resources

* [Better Stack incident](https://status.appwrite.online/incident/534464)
* [PR to fix schedules](https://github.com/appwrite/appwrite/pull/9577)
* [community thread](https://discord.com/channels/564160730845151244/1354120595100401684)
