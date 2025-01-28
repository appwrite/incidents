# Unknown attribute: "specification"

## **Date and time**

- **Incident Start:** 2024-08-10 15:16 UTC
- **Incident End:** 2024-08-10 18:29 UTC
- **Report Prepared By:** Steven Nguyen

## Summary

On August 10th, we received reports about functions failing to build with the following error:

```
Invalid document structure: Unknown attribute: "specification"
```

After digging into the code and database, we discovered the `specification` column was in the functions tables, but it wasn't in all of the metadata.

After removing the `specification` column from the functions table, function creation and builds worked again.

Any functions created during the incident will be in a malformed state and will need to be manually deleted. There will be an error, but it should still delete successfully.

## Incident details

- **Initial detection:** Developers reported build errors on Discord
  - [Build broken](https://discord.com/channels/564160730845151244/1271849547814928464)
  - [Function Deployment not working](https://discord.com/channels/564160730845151244/1271868837343531038)
- **Affected components:** Function creation and deployments resulted in errors
- **User impact:** Some developers were unable to create a new functions or deploy existing functions

## Root cause analysis

After troubleshooting with developers who faced the problem, we were table to narrow down which projects had the problem so we could dig into the database and see what was wrong.

After investigating the tables, we discovered the new `specification` attribute was missing from the `_metadata` entry.

We added the `specification` attribute in a recent zero-downtime patch, but there was a bug where the `_metadata` record wasn't updated properly. Since the `specification` attribute was in the table, it was returned when the function was fetched from the database, but since it wasn't in the `_metadata` table, the subsequent [update function document](https://github.com/appwrite/appwrite/blob/e31bae7644e113153c552f184e89131479c43722/app/controllers/api/functions.php#L278) resulted in the "Invalid document structure" error.

## Resolution and recovery

- **Immediate actions:** No actions were taken to mitigate the problem
- **Resolution:** To prevent the error from occurring, we:
  1. Removed the attribute from the metadata of the 1 project that had it to make all projects consistent.
  2. Dropped the column from the functions tables so that the attribute wouldn't be in the resulting document.
  3. Flushed the cache to ensure the old `_metadata` and functions with the `specification` attribute were removed.

## Lessons learned

- **What went well:** [Incident](https://status.appwrite.online/incident/411901) was created to keep the community notified
- **What could be improved:**
  - We only knew about this and dug into it after more than a handful of the community raised concerns
  - Better validation, logging, and testing of patch scripts
- **Action items:**
  - Properly add the specification attribute

## Additional resources

- [Better Stack incident](https://status.appwrite.online/incident/411901)
