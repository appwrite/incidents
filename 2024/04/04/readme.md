## **Date and Time**

- **Incident Start:** 2024-04-04 22:00 UTC
- **Incident End:** 2024-04-05 03:00 UTC
- **Report Prepared By:** Evan LeAir + Steven Nguyen

## Summary

On April 4th at approximately 22:00 UTC, we received notification of increased error rates on our cloud platform related to function timeouts. During the initial investigation, we discovered that disk space was almost full due to logs from our intrusion detection system (IDS) tooling. To remedy, the IDS was disabled and disk usage was freed. As a result of the service being disabled, load balancers were unable to communicate with each other and the container orchestration system lost its leader. Once service was restored and stable, the IDS was re-enabled with appropriate log retention and rotation configurations.

## Incident Details

- **Initial Detection:** Users reported errors and we noticed real-time services not connecting with attempts to constantly reconnect
- **Affected Components:** Some load balancers disconnected and eventually caused all cloud services to not function
- **User Impact:** Users were unable to reliably connect to the console, view their projects, or access the API endpoints. This resulted in high error rates, increased latency, and network gateway timeouts.

## Root Cause Analysis

- **Preliminary Findings:** Initial analysis showed that our IDS tooling was consuming disk space on the load balancers
- **Investigation:**
    - Load balancer disk space was almost maxed out due to IDS logs
    - Disk space was freed by cleaning the container system, compressing logs, and truncating large files
    - After servers recovered, certificates weren't syncing across all servers due to NFS mount issues
    - IDS was temporarily disabled to prevent logs from filling up disk space again, but this caused load balancers to lose network connectivity
    - Re-enabling IDS restored network connectivity and allowed the container orchestration system to elect new leaders
- **Root Cause:**
    - Load balancers running out of disk space due to IDS logs caused initial instability, leading to unresponsive reverse proxy containers
    - Disabling IDS to prevent disk space issues caused loss of network connectivity between load balancers, disrupting the container orchestration system
    - Certificates and configs were not properly mounted into the reverse proxy containers

## Resolution and Recovery

- **Immediate Actions:** No actions were successful in mitigating the problem
- **Resolution:** Service was fully recovered by freeing up disk space on the load balancers and re-enabling the IDS service, which restored network connectivity and allowed the container orchestration system to elect new leaders.

## Lessons Learned

- **What Went Well:** Several team members collaborated to help resolve the issue
- **What Could Be Improved:**
    - Response Time:
        - Improve detection mechanisms to accurately monitor system infrastructure
        - Improve alerting for CPU, memory, and disk usage
        - Enhance configuration management to better track manually installed services
- **Action Items:**
    - Document the process for reinitializing the container orchestration system
    - Improve DR strategy in the event of orchestration leader loss to ensure accurate failover
    - Improve system/infrastructure resource monitoring and alerting (disk space)
    - Rotate IDS logs to a separate volume