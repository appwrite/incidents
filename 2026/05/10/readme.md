# Unsuccessful Function and Site Invocations

- **Incident Start:** 2026-05-10 18:05 UTC
- **Incident End:** 2026-05-10 19:25 UTC
- **Report Prepared By:** @levivannoort

## Summary

At 18:05 UTC, the function and site invocation success rate dropped significantly in the `fra` region. The issue was caused by an infrastructure scaling operation that removed too much capacity simultaneously, preventing new runtime workloads from starting. Scaling up the affected infrastructure restored the success rate. No other regions were affected.

## Incident details

### Initial detection

User reports of failed function invocations were received via support channels. Monitoring dashboards confirmed a significant drop in the runtime success rate, isolated to the `fra` region.

### Affected components

- Function and site runtime scheduling in the `fra` region.

### User impact

- Functions and sites created or invoked after the start of the incident had a high chance of failing in the `fra` region.
- Existing workloads already running were largely unaffected.
- Other regions were not impacted.

## Root cause analysis

### Root cause

A scheduled infrastructure compaction job removed capacity from two resource pools simultaneously rather than accounting for their combined impact. This left insufficient room for new runtime workloads to start, causing them to remain in a pending state.

## Resolution and recovery

### Immediate actions

- Identified the capacity shortfall through cluster monitoring.
- Increased the minimum capacity on the affected infrastructure to restore service.

### Resolution

After scaling up and allowing time for the infrastructure to become fully ready, the success rate recovered. The compaction job was temporarily disabled to prevent recurrence while further investigation was conducted.

## Lessons learned

### What went well

- Quick identification of the affected region and root cause direction.
- Immediate corrective action taken to restore capacity.

### What can be improved

- Certain runtime metrics were not retained, making it harder to pinpoint the exact failure mode.
- The compaction job operated on each resource pool independently without awareness of the combined impact.

### Action items

- [ ] Reduce the compaction frequency to limit the impact of capacity changes on runtime success rates.
- [ ] Update compaction logic to operate holistically rather than independently per resource pool.
- [ ] Set up broader alerting for function and site success rate drops.
