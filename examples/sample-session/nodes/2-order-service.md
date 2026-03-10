# Node 2 — OrderService uses PaymentProcessor directly

## Identity

- **ID:** 2
- **Parent:** 0 (Extract Payment Service)
- **Depth:** 1
- **Status:** in-progress

## Context

- **File(s):** src/Services/OrderService.cs
- **Line(s):** 23-45
- **Worktree branch:** mikado/extract-payment-2

## Break Description

After extracting `IPaymentService` (node 0), `OrderService` still directly
references `PaymentProcessor` for payment calculations. This class needs to
use the new `IPaymentService` interface instead.

## Error Output

```text
src/Services/OrderService.cs(28,20): error CS0246: The type or namespace name 'PaymentProcessor' could not be found
src/Services/OrderService.cs(34,8): error CS1061: 'object' does not contain a definition for 'CalculateTotal'
```

## Proposed Fix

Update `OrderService` to accept `IPaymentService` via constructor injection.
Replace `PaymentProcessor.CalculateTotal()` calls with `IPaymentService.CalculateTotal()`.

## Children

- 2.1 — OrderTotalCalc missing IPaymentService
