# DDoS attack

- **Incident Start:** 2024-10-18 14:10 UTC
- **Incident End:** 2024-10-19 12:04 UTC
- **Report Prepared By:** Steven Nguyen

## Summary

On the day of the incident, we saw alerts of appwrite.io and cloud.appwrite.io being unavailable. After some initial analysis, we realized we were being hit by a distributed denial of service (DDoS) attack. To mitigate the problem and get back online, we added protections to block the unwanted traffic.

## Incident details

### Initial detection

The issue was first identified through several monitoring alerts, including:

- website uptime monitor
- system monitoring
- load balancer health checks

### Affected components

The primarily affected components were the loadbalancers for appwrite.io and cloud.appwrite.io.

### User impact

Due to the failure of the load balancers, users attempting to access both the cloud platform and the main website encountered errors stating "This site can't be reached."

## Root cause analysis

### Preliminary findings

Analysis of the affected services and logs indicated a DDoS attack as the source of the disruption.

### Investigation

Our metrics indicated we were receiving higher than usual usage.

In addition, the logs for cloud.appwrite.io showed a huge influx of multiple IPs hitting cloud.appwrite.io following the same pattern.

Ultimately, this lead us to believe we were being hit by a DDoS attack.

### Root cause

Because of the DDoS attack, our services were overloaded and unable to handle the incoming requests leading to users receiving the "This site can't be reached" error.

## Resolution and recovery

### Immediate Actions

We attempted to increase the load balancer capacity to handle more connections, but the request volume increased accordingly, rendering this solution ineffective.

In addition, we investigated blocking with firewall rules, but due to the distributed nature of the attack, it was impractical to block every IP address manually.

### Resolution

To effectively mitigate the attack, we added a web application firewall (WAF) in front of our systems. Using the WAF we were able to add rules to block excessive and/or suspicious requests.

Furthermore, we hardened our systems to prevent bad actors from circumventing the WAF by only allowing requests through the WAF.

## Lessons learned

### What went well

* Monitoring systems provided timely alerts, enabling quick identification of the problem.
* Teams across Engineering, Customer Success, and Developer Relations collaborated effectively to resolve the issue.

### What can be improved

* Some services failed to function after the DNS migration due to missing or misconfigured DNS entries.

### Action items

* Improve monitoring on some of our other sites.

### Additional resources

* [Better Stack incident](https://status.appwrite.online/incident/446920)