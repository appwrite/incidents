## **Date and Time**

- **Incident Start:** 2024-05-21 20:00 UTC
- **Incident End:** 2024-05-21 22:00 UTC
- **Report Prepared By:** Steven Nguyen

## Summary

On May 21st, we received increased reports about function builds being stuck or taking too long. For example:

- [Cloud function deployments from cli takes more than 20 min to complete.](https://discord.com/channels/564160730845151244/1242553150171517029)
- [General channel](https://discord.com/channels/564160730845151244/564160731327758347/1242554284810305567)
- [Cloud Function Stuck Processing After Update](https://discord.com/channels/564160730845151244/1242450827763908649)

After digging into the build queue, we discovered slow builds were blocking other function builds. After preventing the slow builds from running, the queue recovered.

## Incident Details

- **Initial Detection:** Users reported increased build times
- **Affected Components:** Function builds were blocked and took a while to build
- **User Impact:** Users were unable to build and deploy their functions

## Root Cause Analysis

- **Preliminary Findings:** Initial analysis showed the build queue was not decreasing to 0 in a timely manner

- **Investigation:**
    - The queue was ~20, but the unusual part was it didn't just go down, but went back up again
    - Looking at the logs, we noticed some builds were consistently taking 30 minutes to complete
      - This seemed to line up with the release the day before that increased the build timeout to 30 minutes.
    - To get details on the deployments, we reviewed the job in the queue and identified builds that:
      - triggered every 15 minutes
      - failed every time

- **Root Cause:**
    - It looks like the failing function builds and the release earlier in the day caused the build queue to be overrun by the failing builds forcing other builds to wait until the build timed out.

## Resolution and Recovery

- **Immediate Actions:** No actions were taken to mitigate the problem
- **Resolution:** We stopped additional builds from triggering on this suspicious function

## Lessons Learned

- **What Went Well:** [Incident](https://status.appwrite.online/incident/372758) was created to keep the community notified
- **What Could Be Improved:** We only knew about this and dug into it after more than a handful of the community raised concerns
- **Action Items:**
    - Add additional alerts so we detect the problem sooner
    - Make it easier for server admins to disable functions from building or executing
