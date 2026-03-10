# Node 3 — InvoiceHandler casting to PaymentProcessor

## Identity

- **ID:** 3
- **Parent:** 0 (Extract Payment Service)
- **Depth:** 1
- **Status:** leaf

## Context

- **File(s):** src/Billing/InvoiceHandler.cs
- **Line(s):** 56-62
- **Worktree branch:** mikado/extract-payment-3

## Break Description

After extracting `IPaymentService` (node 0), `InvoiceHandler` casts the payment
service to `PaymentProcessor` at runtime to access internal methods. This cast
fails because the concrete type has been replaced with the interface.

## Error Output

```text
src/Billing/InvoiceHandler.cs(58,28): error CS0246: The type or namespace name 'PaymentProcessor' could not be found
```

## Proposed Fix

Remove the `(PaymentProcessor)` cast in `InvoiceHandler`. If the internal method
(`GetTransactionLog()`) is needed, add it to the `IPaymentService` interface.
Otherwise, remove the cast and use the interface methods directly.

## Children

None (leaf node)
