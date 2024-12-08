# MySQL volume size allocation

- **Incident Start:** 2024-10-04
- **Incident End:** 2024-10-04
- **Report Prepared By:** Shimon newman

## Summary
One  MySQL cluster reached its storage capacity. 
Cluster volumes got resized and the cluster got restarted.
During restart MySQL replication issued some errors.
Data was restored from backup and another restart was conducted.

### Initial detection
The issue was first identified via uptime monitor alert.

### Affected components
Single MySQL cluster, Cloud API.

### User impact
Users that have projects hosted on the specific MySQL cluster.

### Preliminary findings
Quick check against MySQL cluster volumes showed that the allocated storage for the volumes has reached its limit.

### Root cause
The allocated storage for MySQL volumes has reached its limit.

### Immediate Actions
MySQL volumes got resized.
Cluster was restarted.

### What went well
The team’s swift response successfully minimized the duration of user impact during the incident.

### What can be improved
Additional storage size allocation monitoring needs to be introduced.

