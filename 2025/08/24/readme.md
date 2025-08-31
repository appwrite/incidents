# Short title

- **Incident Start:** 2025-08-24 07:26 UTC
- **Incident End:** 2025-08-26 05:10 UTC
- **Report Prepared By:** Divyansha

## Summary

Between 24–26 August 2025, some Appwrite Cloud users experienced intermittent issues with email delivery, SMS and email OTPs, and realtime events. The issue was caused by instability in our messaging system. The problem was resolved by stabilizing the messaging infrastructure, and all services are now fully operational.

## Incident details

### Initial detection

The issue was first detected when users reported missing or delayed emails and OTPs.

### Affected components

Email delivery (invitations, password resets, notifications)
OTP delivery via email and SMS
Realtime events

### User impact

Some customers experienced delayed or failed OTPs and emails, which affected their ability to onboard new users or use realtime features.

## Root cause analysis

### Preliminary findings

We noticed few errors in the messaging system responsible for handling message delivery.

### Investigation

The issue was linked to instability in the messaging system.

### Root cause

The root cause was overload in the underlying messaging queues, which led to dropped or delayed messages and intermittent realtime failures.

## Resolution and recovery

### Immediate Actions

We restarted affected services across regions to temporarily restore delivery.

### Resolution

We stabilized the messaging infrastructure by migrating critical queues to a more reliable system and disabling task contributing to overload.

## Lessons learned

### What went well

Quick detection of failures and temporary restarts reduced user impact.
Migration to a more reliable system helped restore stability quickly.

### What can be improved

Better visibility into system dependencies is needed to speed up diagnosis.

### Action items

Improve monitoring and alerting for earlier detection of overloads.

## Additional resources

### Supporting documentation

https://status.appwrite.online/incident/711585