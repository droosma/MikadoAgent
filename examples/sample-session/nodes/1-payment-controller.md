# Node 1 — PaymentController depends on concrete class

## Identity

- **ID:** 1
- **Parent:** 0 (Extract Payment Service)
- **Depth:** 1
- **Status:** in-progress

## Context

- **File(s):** src/Controllers/PaymentController.cs
- **Line(s):** 15-32
- **Worktree branch:** mikado/extract-payment-1

## Break Description

After extracting `IPaymentService` (node 0), `PaymentController` still instantiates
`PaymentProcessor` directly in its constructor instead of accepting `IPaymentService`.
Multiple related types in the payment namespace also reference the concrete class.

## Error Output

```text
src/Controllers/PaymentController.cs(18,24): error CS0246: The type or namespace name 'PaymentProcessor' could not be found
src/Controllers/PaymentController.cs(25,12): error CS1061: 'IPaymentService' does not contain a definition for 'ProcessDirectly'
```

## Proposed Fix

Update `PaymentController` constructor to accept `IPaymentService` via DI instead
of creating `PaymentProcessor` directly. Replace `ProcessDirectly()` call with
the interface method `ProcessPayment()`.

## Children

- 1.1 — PaymentDTO in wrong namespace
- 1.2 — PaymentValidator references PaymentProcessor
