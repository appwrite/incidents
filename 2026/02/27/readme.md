# FRA Region Cache Failure — Migration Script Cache Wipe Loop

- **Incident Start:** 2026-02-27 14:15 UTC
- **Incident End:** 2026-02-27 15:15 UTC
- **Report Prepared By:** Luke Silver

## Summary

A database migration patch script running against the FRA region issued cache wipe commands to the cache master in a loop (~1 command every 3 seconds). This repeatedly wiped all cached items, triggering a cascading failure: cache-miss thundering herd, database overload, Swoole worker pool exhaustion, and API throughput collapse. The FRA region experienced a ~70% drop in request throughput for approximately 30 minutes, returning 5xx errors to end users.

## Incident details

### Initial detection

The incident was detected manually. At 14:36 UTC, a core member noticed elevated cache errors and opened an urgent thread in Discord, tagging the team. There were no automated alerts that fired on cache health degradation. An alert for the stats-usage queue fired at 14:54 UTC, roughly 40 minutes after impact began, but this monitored a secondary symptom rather than cache health directly.

Notably, the underlying cache error had been appearing in error logs for 13 days at low volume from previous migration runs in other regions. It was never escalated.

### Affected components

- **Cache master**: repeatedly flushed, transaction queue backed up
- **API service**: worker pool exhaustion, throughput collapse
- **Database**: 3-4x QPS spike from cache-miss thundering herd
- **Rate-limiting middleware**: hard dependency on cache, caused request failures
- **Stats-usage queue**: backlog exceeded

### User impact

- user-facing 5xx errors at the edge
- users affected by cache read errors
- ~30 minutes of severe degradation in FRA where API throughput dropped by 70%

## Root cause analysis

### Preliminary findings

The team initially observed API errors and restarted API containers (14:32 UTC). Cache replica sync errors were spotted in logs. The issue appeared limited to FRA. A core member noted cache errors had been present in logs "for days" at low levels. Another core member flagged the migration as a possible factor.

### Investigation

The team joined a debugging call at 14:53 UTC. Key findings in sequence:

1. **Cache dashboard:** Total cached items dropped from to near zero. Connected clients fell by ~70%. Cache uptime showed 10 weeks — the process never restarted, ruling out a crash.

2. **API dashboard:** Total request rate dropped ~70%. Swoole idle workers hit zero (full worker pool exhaustion). The API workers autoscaled up by 90% but could not compensate since the bottleneck was DB capacity, not API worker count.

3. **Database dashboard:** QPS on a particular database primary spiked 3-4x. DB connection use times increased 3-7x across multiple pools. The read replica absorbed minimal additional load.

4. **Error logs:** The error stack trace showed failures originating in the rate-limiting middleware, confirming cache failure was killing requests at the abuse-check layer.

5. **Cache logs:** Revealed cache wipe commands queued in a continuous loop. The transaction queue was backed up with pending and armed transactions, confirming the cache was both empty and blocked.

6. **Correlation:** A core member had started the FRA migration with multiple workers ~10 minutes before impact. The migration script wipes the cache to invalidate stale cache entries after schema changes. With multiple workers, this created a sustained loop of full cache wipes.

### Root cause

The migration patch script wipes the entire cache as its cache invalidation strategy. When deployed with multiple worker replicas against FRA, each worker independently wiped the cache, creating a sustained loop (~1 every 3 seconds) that continuously wiped the entire cache.

Since the wipe is a blocking operation in the cache. The repeated calls backed up the transaction queue and starved normal operations. With cache both empty and intermittently unresponsive, the failure cascaded through multiple layers:

1. All reads fell through to the database, causing a 3-4x QPS spike on the primary.
2. Write operations failed because cache invalidation after DB writes is fail-closed — cache exceptions on the purge path kill the write even after the DB commit succeeds.
3. Rate-limiting checks failed because the adapter has a hard dependency on cache and sits in the critical request path. Cache unreachable = request rejected.
4. Swoole workers blocked on cache retries (2 retries x 1s timeout + 1s fixed delay = 3+ seconds per failed cache operation) and slow DB queries, exhausting the worker pool.

**Contributing factors:**

- No circuit breaker around cache operations. The codebase has no circuit breaker or bulkhead pattern. Every request continues attempting cache operations under failure, accumulating latency rather than failing fast.
- Static retry policy (2 retries, 1s fixed delay, no jitter) amplifies thundering herd instead of shedding load.
- Read paths are fail-open, but write and rate-limit paths are fail-closed. This asymmetry means cache failure selectively breaks writes and rate-limiting while reads degrade more gracefully.
- Rate-limiting uses only the first cache config, tightly coupling every request's critical path to a single cache endpoint.
- No cache health alerting. Detection was manual, ~15 minutes after impact began.
- Multiple days of ignored warning signs. The same error had been firing at low volume from prior migration runs in other regions.
- FRA is the largest region. The same script ran successfully in NYC and others, but FRA's scale amplified the flaw into a full outage.

## Resolution and recovery

### Immediate actions

| Time (UTC) | Action |
|------------|--------|
| 14:32 | A core member restarted API containers: partial, temporary relief |
| 14:36 | Another core member opened incident thread, tagged team as urgent |
| 14:53 | Team joined debugging call |
| ~15:09 | Root cause identified via cache logs (cache wipe loop) |
| ~15:15 | Migration patch worker scaled to zero, stopping the cache wipe loop |

### Resolution

The incident was resolved by scaling the migration patch worker deployment to zero, which immediately stopped the cache wipe loop. DB QPS dropped back to baseline within minutes. The cache began refilling organically from application traffic. API throughput recovered to normal levels within 15 minutes. As of the end of the observation window, cache items had recovered to ~23% of the pre-incident baseline, with the remainder expected to refill over time as keys are accessed.

## Lessons learned

### What went well

- The team mobilized quickly once the incident was flagged: from thread to call to root cause in under 40 minutes.
- Cache logs were clear and immediately pointed to the cache wipe loop once examined.
- The DB absorbed a 3-4x load spike without crashing.
- HPA autoscaling kicked in correctly on the databases deployment.
- A maintenance window had been posted to the status page for the migration.

### What can be improved

- **Detection was manual and slow.** No automated alert fired on cache health. The issue was discovered when a core member checked logs. Automated alerting should fire within minutes.
- **Low-volume cache errors persisted for days without escalation.** Previous migration runs in other regions produced the same error at lower volume. A recurring error pattern across regions warrants triage before running the same operation in the largest region.
- **Cache wiping the entire keyspace per invocation.** With multiple workers issuing it concurrently, the cache cannot retain any data. Scoped invalidation (by key prefix or version) would limit blast radius to the migrated dataset.
- **The system's current cache dependency model does not tolerate cache unavailability at FRA's scale.** At the current req/s, cache failure cascades through write paths, rate-limiting, and worker pools faster than manual intervention can respond. Client-side resilience (circuit breakers, fail-open write paths, adaptive retries) would allow the system to absorb cache disruptions with degraded latency rather than full failure.
- **Successful runs in smaller regions validated functionality but not resilience.** NYC and other regions completed the same migration without major incident. The lower traffic volume in those regions masked the cascading failure dynamics that manifested at FRA scale.

### Action items

**P0 — Immediate**

- [ ] **Eliminate cache wipes from the migration script.** Redesign cache key schema so migrations invalidate only affected keys. For the remaining FRA migration, use a safe invalidation approach.
- [ ] **Make cache invalidation fail-open on write paths.** Wrap cache purge/invalidation calls in try/catch. Log and emit metrics on failure, but do not fail the write. Alternatively, move invalidation to an async outbox pattern.
- [ ] **Make rate-limiting fail-open on cache failure.** When cache is unreachable, rate-limit checks should allow requests through (or use an in-memory fallback) rather than throwing and killing the request.
- [ ] **Add cache health alerts.** Alert on: cache item count dropping >50% in 5 minutes, connected clients dropping >30%, cache error rate exceeding threshold, cache wipe commands in logs.

**P1 — Short-term**

- [ ] **Implement circuit breaker per cache shard.** Add a circuit breaker with half-open probes around cache operations. When open, bypass cache entirely rather than accumulating timeout latency.
- [ ] **Replace fixed retry with exponential backoff + jitter.** Current policy (2 retries, 1s fixed delay) amplifies load under failure.
- [ ] **Audit all cache timeout and reconnect paths.** Ensure bounded timeouts are enforced in every code path that touches cache — no unbounded waits in production.
- [ ] **Add per-operation cache observability.** Emit metrics for `cache.error_rate`, `reconnect_count`, `breaker_state`, and `p99_cache_latency`.
- [ ] **Propagate cache invalidation via cache workers.** Use the existing cache worker pipeline to propagate invalidations to all regions rather than wiping the entire cache.
- [ ] **Review DB read replica routing under cache failure.** During the incident, the DB replica stayed at the same QPS while the primary increased. Cache-miss fallback queries should be routable to read replicas.

**P2 — Long-term**

- [ ] **Chaos testing.** Simulate cache failure in FRA and validate graceful degradation. Target: degraded latency but no 5xx errors.
- [ ] **Automated cache reconnection testing.** Integration tests validating cache recovery behavior under failure conditions.

## Additional resources

### Supporting documentation

- [BetterUptime incident](https://status.appwrite.online/incident/835263)
- [Planned maintenance window](https://status.appwrite.online/maintenance/834362)