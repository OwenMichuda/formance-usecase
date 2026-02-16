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
| Platform | craftory:refunds | A pool for instance refunds, to be settled later with vendor

This structure is designed to separate Cash Flow (movement of money) from Revenue State (accounting status). By keeping distinct accounts for each stage of the transaction lifecycle, we create a ledger that is self-auditing and compliant with revenue recognition standards.

### Customer Granularity (:gift_card, :wallet)

By splitting customer funds at the source, we can natively answer questions like "How much revenue came from Gift Cards vs Credit Cards?" without needing complex metadata queries. The ledger structure itself enforces the data quality, and allows for easy expansion for different types of payment methods.

### Invoice State Management (:pending, :settled)

The separation of :pending and :settled creates a hard gate between Deferred Revenue (Liability) and Recognized Revenue.

* Pending: Holds funds as a liability while the service is unfulfilled.
* Settled: Acts as the immutable "trigger" event. Moving funds here provides a precise timestamp for when revenue was legally recognized, distinct from when the cash was received.

### Vendor Cash Flow (:earnings, :settlements)

We separate "Net Revenue" (:earnings) from "Payouts" (:settlements) to prevent overdrafts.

* Earnings: Shows the vendor's total accrual (net wealth).
* Settlements: Shows what is actually queued for bank transfer. This prevents the "Empty Wallet" problem where a vendor withdraws funds that might later be needed for a refund.

### Platform Revenue & Liability (:fees, :taxes, :refunds)


* Fees: Represents Craftory's actual earnings (EBITDA).
* Taxes: Represents money Craftory holds but owes to the government.
    Mixing these would inflate revenue figures and complicate tax audits.
* Refunds: The dedicated refunds pool ensures that Craftory can instantly refund a customer even if the specific Vendor has withdrawn their balance. This prioritizes platform trust over vendor liquidity issues, treating the deficit as a debt to be collected later.

## Q&A

**Q: How do you ensure a transaction is not created twice?**

We utilize idempotency keys. When submitting a transaction, we can include a unique `reference` field. If the ledger receives a sceond request with the same `reference`, it will reject the transaction as a duplicate.

Alternatively, we can utilize the `Idempotency-Key` header to achieve similar functionality. `Idempotency-Key` should be favored when silent duplicate handling is desired, whereas `reference` should be favored when there is a unique entity in the system the transaction should match.

**Q: What are the different ways to create accounts w/ negative balance?**

By default, Formance accounts cannot go negative. We can allow for negative accounts with either:
1. **Unbounded Overdraft**: `allowing unbounded overdraft` allows the account to go infinitely negative. 
2. **Bounded Overdraft**: `allowing overdraft up to [monetary]` allows the account to go negative up to a specific limit.

**Q: What is your understanding of a transaction's effectiveVolume?**

`effectiveVolume` is the mechanism for handling backdated transactions correctly. Unlike "Regular Volumes" which represent the state of the ledger now, effectiveVolume represents the balance of an account as it was at the specific date/time the transaction was effectively posted, even if that transaction was inserted later.

If we insert a backdated transaction (e.g., posting a correction for last month today), the ledger must "rewrite history" for that account. It recalculates the `effectiveVolume` for every subsequent transaction to ensure the running balance remains historically accurate. This allows us to trust historical reporting without locking the ledger, although it comes with a performance cost for deep backdating.
