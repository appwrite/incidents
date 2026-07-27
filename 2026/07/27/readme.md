# Database connection failure

- **Incident Start:** 2026-07-27 ~8:43 UTC
- **Incident End:** 2026-07-27 ~9:10 UTC
- **Report Prepared By:** Shimon Newman

## Summary

On July 27, projects sharing one of our database clusters experienced intermittent query timeouts and elevated response times over an approximately 30-minute window. The malfunction was caused by a database cluster connection failure. We quickly switched traffic to a standby cluster and service was restored. Projects on other database clusters were unaffected.

## Incident details

### Initial detection

The issue was identified through internal monitoring.

### Affected components

- A single database cluster and the projects hosted on it.
- No other database clusters were affected.

### User impact

- Projects sharing the affected cluster saw intermittent query timeouts and increased response times between ~8:43 UTC and ~9:10 UTC.
- The impact was at the database layer only; projects hosted on other clusters had zero impact.
- No data was lost.

## Root cause analysis

### Root cause

One of our database clusters experienced a connection failure that prevented it from serving queries reliably. As connections to the primary cluster became unavailable, projects hosted on that cluster began seeing intermittent query timeouts and elevated response times. The failure was isolated to this single cluster; no other database clusters were affected.

## Resolution and recovery

### Immediate actions

- Detected elevated error rates and connection failures on the affected database cluster through internal monitoring.
- Routed traffic away from the affected cluster and switched to a standby cluster.
- Monitored the standby cluster until query latency and error rates returned to normal.

### Resolution

User-visible impact ended at approximately 9:10 UTC on July 27 once traffic was successfully served from the standby cluster and database operations returned to normal. No data was lost during the failover.

## Lessons learned

### What went well

- Internal monitoring detected the connection failure quickly.
- Failover to the standby cluster was executed promptly, limiting the duration of user impact to approximately 30 minutes.
- The blast radius remained confined to a single database cluster; projects on other database clusters saw no impact.

### What can be improved

- Investigate the underlying cause of the database cluster connection failure to reduce the likelihood of recurrence.
- Review and document the failover procedure to ensure it can be executed consistently under pressure.

### Action items

- Determine the root infrastructure cause of the database cluster connection failure and apply any necessary fixes to the primary cluster.
- Review health checks and alerting thresholds to ensure connection failures are surfaced as early as possible.
- Validate standby-cluster failover paths and runbooks across all database clusters.
