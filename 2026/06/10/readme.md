# Delayed propagation of site and function deployments

- **Incident Start:** 2026-06-10 ~16:00 UTC
- **Incident End:** 2026-06-12 ~11:10 UTC
- **Report Prepared By:** Chirag Aggarwal

## Summary

Between June 10 and June 12, activating a new site or function deployment could intermittently fail to refresh the content served on the deployment's domain. Affected users continued to see a previous version of their site or function response, and redeploying did not always resolve it. The issue was caused by a regression in our CDN cache invalidation pipeline. A global CDN cache flush restored correct behavior for all affected deployments, and fixes have been merged to prevent recurrence.

## Incident details

### Initial detection

The issue was identified through user reports of deployments not going live after activation.

### Affected components

- CDN cache invalidation for site and function deployments across all regions.

### User impact

- Newly activated deployments could continue serving the previous deployment's content, depending on region and traffic patterns.
- Sites receiving traffic during the activation window were most likely to be affected.
- No data was lost, and function executions were unaffected.

## Root cause analysis

### Root cause

A change to the cache invalidation system updated the cache keys used for purging cached content, while the corresponding change that applies those keys to newly cached content did not reach production due to a merge-ordering issue. As a result, cache purges no longer matched cached objects, and stale content could persist on CDN nodes after a new deployment was activated.

## Resolution and recovery

### Immediate Actions

- Restored the direct cache purge path for deployment domains.
- Performed a complete flush of the CDN cache, immediately restoring correct content for all affected domains.
- Merged the fix that applies the new cache keys to all newly cached content.

### Resolution

User impact ended on June 12 at ~11:10 UTC with the global cache flush. The remaining fix is being rolled out to all edge regions, followed by a final cache flush.

## Lessons learned

### Action items

- Roll out the cache key fix to all edge regions and perform a final cache flush.
- Add automated end-to-end monitoring that continuously verifies deployment propagation across regions.
- Add retries and dead-letter alerting to the cache invalidation pipeline.
- Strengthen our release process for changes that span multiple repositories with deployment-order dependencies.
