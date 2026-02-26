# Sites and functions errors

- **Incident Start:** 2026-02-24 14:15 UTC
- **Incident End:** 2026-02-24 22:30 UTC
- **Report Prepared By:** Steven Nguyen and Luke Silver

## Summary

<!-- A brief overview of the incident, including the affected services and the impact on users and the system. -->

At around 2026-02-24 14:15 UTC we had increased errors for functions and sites.

## Incident details

### Initial detection

<!-- How and when the issue was first detected. -->

We noticed errors in our Edge observability dashboard.

### Affected components

<!-- List of affected services, features, or infrastructure components. -->

- Sites
- Functions

### User impact

<!-- Description of how the incident impacted end users, including any potential data loss or service unavailability. -->

Due to the incident, functions and sites had errors.

## Root cause analysis

### Preliminary findings

<!-- Initial insights into what caused the incident. -->

Looking at our logs, we noticed several DNS-related errors and suspected an issue with our orchestration platform and/or our upstream cloud provider.

### Investigation

<!-- Detailed explanation of the investigation process, tools used, and key findings. -->

Digging into the different nodes, it seemed like the problem was specific to certain nodes, while some were fine.

Looking at errors for edge, we noticed several DNS resolution errors for hostnames that should not have issues such as internal and well-established. 

We ruled out regression from releases as the timeline of the release did not match up with when the incident started.

### Root cause

<!-- Concise statement of the primary cause of the incident. -->

The issue was caused by networking issue on one of upstream provider's hypervisor hosts that affected connectivity for some virtual machines. Since one of our worker nodes was also hosted on the same hypervisor, this caused the overall issues within our cluster.

## Resolution and recovery

### Immediate Actions

<!-- Steps taken to mitigate the impact and restore service. -->

We initially tried to restart the runtimes to resolve the problem. While things improved, we still had a significant percentage of failures.

To help mitigate the issue, we re-routed sites and function domain executions to Toronto.

Since it seemed like the issue was related to DNS, we also tried to scale up our internal DNS server.

### Resolution

<!-- How and when was the incident fully resolved? -->

To resolve the issue, we created a local DNS caching agent and patched the Edge API to use it. Then, we triggered a rollout of Edge. This enabled us to have a node-level cache of DNS entries rather than relying on the host DNS resolver.

## Lessons learned

### What went well

<!-- Aspects of the incident response that were effective. -->

- First attempt of Edge failover went well.
- Observability allowed us to clearly measure the impact of the issue and validate recovery.

### What can be improved

<!-- Areas where the response could have been better or faster. -->

- Detection was slow due to alert noise.
- Unable to failover API-invoked functions.

### Action items

<!-- Specific steps to be taken to prevent recurrence, improve detection, or enhance response. -->

* Automate re-routing traffic when a region goes down
* Ability to re-route API-invoked functions traffic
* Alerting for K8s resource creation time
* Deployment of node-level cache of DNS entries to reduce dependency of cross-cluster traffic

## Additional resources

### Supporting documentation

<!-- Links to relevant logs, graphs, or incident-related documents. -->

* [BetterStack incident](https://status.appwrite.online/incident/832277)
