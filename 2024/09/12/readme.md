# Security advisory: Leaked internal key

## Description

On September 12th, 2024, a **key exposure** was detected in Appwrite Function logs. Appwrite team identified the key as a secret used for **internal communication** inside Cloud between Appwrite API and Appwrite Functions. This key could be used maliciously to see **deployment logs and execute functions**.

## Immediate action taken

Within **2.5 hours** we identified, fixed, tested and deployed a fix. The key **cannot be exposed anymore**, and **key was rotated** to prevent any further misuse.

## Impact

After serious inspection of logs, we found no evidence of malicious executions or compromise of logs.

## Action items

If your function doesn't have public execute permission and you see any indication of malicious function executions, please investigate possible harms to your application, and let us know if we can help in any way.

If any secrets are printed during your build step or during execution, we highly recommend rotating those keys.
