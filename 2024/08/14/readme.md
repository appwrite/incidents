# Wrongly sent billing alerts

### Date and Time:
- **Incident Start**: 2024-08-14 17:00 UTC
- **Incident End**: 2024-08-14 19:00 UTC
- **Report Prepared By**: **Damodar Lohani**

### Summary
On August 14th, 2024, we noticed that we sent more billing alerts than usual. This happened immediately after new cloud version was released. After digging more into database we noticed that we had some issue with aggregation of usage.

### Incident Details:
- **Initial Detection**: We noticed the billing alerts emails being sent
- **Affected Components**: Billing aggregation and alerts were wrong
- **User Impact**: Users got wrong alerts and some of their projects got blocked

### Root Cause Analysis:
- **Preliminary Findings**: Initial analysis showed that the aggregation had increased values than previous data before the deployment.
- **Investigation**: 
  - We looked at the aggregation data for the teams that got the alerts, compared it with the values before the deployment
  - We found that the bandwidth value were significantly different
  - Upon investigating the changes happened in the new deployment we identified a typo that caused the value to be different
- **Root cause:**
  - The typo in the latest deployment caused the bandwidth value to significantly increase and teams got the billing alerts

### Resolution and Recovery:
- **Immediate Actions**: Immediately we reverted back the deployment to previous deployment that was working as expected
- **Resolution**: We identified and fixed the problematic code and deployed new version, than re-ran the billing calculations so that all the blocked teams were restored to their original state

### Lessons Learned:
- **What Went Well**: Immediate monitoring and identification of the issue
- **What Could Be Improved**: We only knew after the alerts were sent.
- **Action Items**: 
  - Add additional alerts when values significantly differ than the previous cycle or previous iteration of the billing calculations
