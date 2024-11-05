# Problem Generating webp Images Incident Report

- **Incident Start:** 2024-09-28 12:00 UTC
- **Incident End:** 2024-09-28 14:45 UTC
- **Report Prepared By:** Christy Jacob

## Summary

The incident involved issues with our image processing service, particularly with generating and loading images in the webp format. Images that previously worked stopped being displayed when transformed to webp, while other formats such as jpg, jpeg, and png continued to function correctly. This issue affected our customers, causing disruptions in displaying media content.

## Incident Details

### Initial Detection

The issue was first detected on 2024-09-28 at 12:00 UTC, following reports from customers. The issue was specifically noticed when images transformed into the webp format failed to load, resulting in blank or missing image displays.

### Affected Components

- **Services:** Storage Service (specifically the image preview endpoint)
- **Features:** Image preview generation (webp format)

### User Impact

Users experienced problems loading images when transforming into the webp format. In some cases, images failed to display entirely, while others loaded inconsistently across different browsers and image sizes. Images that were already cached continued to load, thereby reducing the severity of the incident and resulting in a partial service degradation.

## Root Cause Analysis

### Preliminary Findings

The issue was linked to a missing system library required for generating webp images. Initial investigation suggested that the problem might be related to a recent system update, affecting the image transformation process.

### Investigation

The engineering team conducted a thorough investigation, including:
- Error monitoring and logging analysis
- Manual testing across different browsers
- Container-level debugging
- Examination of image processing components

### Root Cause

The primary cause of the incident was the absence of a required system library in the environment. This library is necessary for the generation of webp images. The issue stemmed from a recent system update, which resulted in the library being excluded from the environment.

## Resolution and Recovery

### Immediate Actions

- Once the issue was identified, the missing component was quickly restored
- Tests were conducted in development and staging environments
- The engineering team verified the fix was working as intended
- The solution was deployed to production

### Resolution

The issue was fully resolved at 14:45 UTC on 2024-09-28, after the deployment was complete and all services were updated with the necessary components. Subsequent image transformations worked correctly.

## Lessons Learned

### What Went Well

- Quick collaboration between team members during a weekend
- Rapid identification and resolution of the issue

### What Can Be Improved

- **Incident detection:** Better monitoring could have helped identify the issue faster
- **Better Exception Handling:** Improved error reporting needed for similar issues
- **Library dependencies:** Enhanced checks needed for system dependencies
- **Testing:** Additional image transformation testing required

### Action Items

- Add more comprehensive e2e tests for image transformation
- Improve error reporting for the image processing service
- Enhance dependency verification processes
- Review system update procedures