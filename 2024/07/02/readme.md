
# Database Cluster Failure

### Date and Time:
**Incident Start**: 2024-07-02 06:00 UTC  
**Incident End**: 2024-07-02 11:00 UTC  
**Report Prepared By**: **Shimon Newman**

### Summary
One of MySQL databases cluster got restarted due to high memory consumption.  
During the MySQL restart on the cluster replica instances, data corruption was detected on both replica nodes.  
Data restoration was conducted from a backup.

### Incident Details:
**Initial Detection**: A database monitoring heartbeat notification was missed.  
**Affected Components**: One of MySQL databases cluster.  
**User Impact**: The service was down while the primary node was restarted (about 1.5 hours).

### Root Cause Analysis:
**Preliminary Findings**: High memory consumption on all MySQL nodes on one of MySQL databases cluster.  
**Investigation**: A rare situation occurred where all nodes of one of MySQL databases cluster restarted simultaneously due to high memory consumption.  
Service restarts were performed on all database nodes.  
As the cluster finished loading, the replicas issued errors related to data corruption.

### Resolution and Recovery:
- **Immediate Actions**: Data restoration was immediately performed on both replicas from backups.

### Lessons Learned:
- **What Went Well**: Immediate response ensured that replicas were restored to their previous state.
- **What Could Be Improved**: Improved monitoring of high memory usage.
- **Action Items**:
    - Add memory consumption alerts.
