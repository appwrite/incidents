# Database cluster volumes size allocation

- **Incident Start:** 2024-10-04
- **Incident End:** 2024-10-04
- **Report Prepared By:** Shimon Newman

## Summary
One  database cluster reached its storage capacity.
Cluster volumes got resized and the cluster got restarted.
During restart database replication issued some errors.
Data was restored from backup and another restart was conducted.

### Initial detection
The issue was first identified via uptime monitor alert.

### Affected components
One of the databases cluster, Cloud API.

### User impact
Users that have projects hosted on the specific database cluster.

### Preliminary findings
Quick check against database cluster volumes showed that the allocated storage for the volumes has reached its limit.

### Root cause
The allocated storage for database volumes has reached its limit.

### Immediate Actions
Database cluster volumes got resized.
Database cluster was restarted.

### What went well
The team’s swift response successfully minimized the duration of user impact during the incident.

### What can be improved
Additional storage size allocation monitoring needs to be introduced.
