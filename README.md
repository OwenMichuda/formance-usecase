## Executive Summary

Craftory is expanding from a simple e-commerce marketplace to an embedded finance provider for artisans. This document outlines a ledger model built on Formance that ensures multi-source payment support, automated revenue recognition, and handling of cancellations and credit reallocations.

## Account Design

| Category | Account Template | Purpose |
| --------- | ---------------- | ------- |
| Customer | customers:{customer_id}:gift_card | Tracks prepaid gift card balances specific to a customer |
| Customer | customers:{customer_id}:wallet | Represents the customer's main payment source (e.g., credit card, cash balance) |
| Invoice | invoices:{invoice_id}:pending | Escrow/Liability. Holds funds from customer payment until the order is fulfilled |
| Invoice | invoices:{invoice_id}:settled | Revenue Recognition. Temporary holding for funds immediately after fulfillment/shipping, before distribution |
| Vendor | vendors:{id}:earnings | Money vendors have earned after fees |
| Vendor | vendors:{id}:settlements | Funds moved here when they are ready to be paid out to the vendor's real-world bank account |
| Platform | craftory:fees | Collects Craftory's commission/take rate from every transaction |
| Platform | craftory:taxes | Collects sales tax/VAT to be remitted to the government |

## Transaction Lifecycle

## Edge Cases

## Q&A

**Q: How do you ensure a transaction is not created twice?**

Idempotency

**Q: What are the different ways to create accounts w/ negative balance?**

Overdraft MAKE SURE TO MENTION ALTERNATIVE DESIGN IMPLEMENTATION WITH NEGATIVE BALANCE INVOICES

**Q: What is your understanding of a transaction's effectiveVolume?**

Idempotency