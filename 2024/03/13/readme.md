# Security advisory: command injection vulnerability

## Description

On March 12th, 2024, an Appwrite core member discovered a **command injection vulnerability**. This vulnerability allowed **developers with access to manage Appwrite Functions in a project** to execute arbitrary commands in the executor container, the component responsible for coordinating the build and execution of functions.

## Immediate action taken

We patched the vulnerability in **less than 18 hours** and started an investigation of **all functions on Appwrite Cloud**.

## Impact

We verified **no one exploited the vulnerability** on Appwrite Cloud. As such, we are confident **no data was leaked**.

## Action items

If you are self-hosting Appwrite versions **1.4.0** - **1.4.13** or **1.5.0** - **1.5.3**, please [update](https://appwrite.io/docs/advanced/self-hosting/update) to **1.4.14** or **1.5.4**.
