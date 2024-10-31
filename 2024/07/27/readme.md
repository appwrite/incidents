## **Date and Time**

- **Incident Start**: 2024-07-27 01:00 UTC
- **Incident End**: 2024-07-28 03:00 UTC
- **Report Prepared By**: Damodar Lohani

## Summary

On July 27th, we noticed some payments did not match the amount of the invoice. Also, some payments were for organizations whose invoice should have been settled by credits alone. Upon further investigation we found two issues. The first issue was with credits calculation and the second was some invoices were processed twice.

## Incident Details

- **Initial Detection:** Routine monitoring of our payments led us to notice the wrong payment
- **Affected Components:** Payment processing component
- **User Impact:** Some organizations were charged when they should not have been

## Root Cause Analysis

- **Preliminary Findings:** Initial analysis revealed a cache failure during payment processing caused invoices to be marked as failed even though it was already processed successfully. As such, the invoice was processed again the next day, where the error in calculation led to a charge even though they should not have been charged.
- **Root Cause:**
    - cache failure
    - wrong calculation

## Resolution and Recovery

- **Immediate Actions:** Refunded the payment immediately and notified the affected user via email. Fixed the problematic calculation and deployed.
- **Resolution:** Apart from the calculation fix, we are also working on improving error handling after a payment is processed to prevent the same invoice to be processed more than once.

## Lessons Learned

- **What Went Well:** Regular monitoring of payments and invoices
- **What Could Be Improved:**
    - Improved monitoring: Improve and automate the monitoring of invoices and payments
    - Improved error handling in payment processing
