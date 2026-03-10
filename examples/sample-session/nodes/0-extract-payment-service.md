# Node 0 — Extract Payment Service

## Identity

- **ID:** 0
- **Parent:** (none — root)
- **Depth:** 0
- **Status:** in-progress

## Context

- **File(s):** src/Controllers/OrderController.cs
- **Worktree branch:** mikado/extract-payment-0

## Goal Description

Extract payment processing logic from `OrderController` into a dedicated
`IPaymentService` interface and `PaymentService` implementation to enable
unit testing and support multiple payment providers.

## Proposed Change

1. Create `IPaymentService` interface in `src/Payment/IPaymentService.cs`
2. Create `PaymentService` class implementing `IPaymentService` in `src/Payment/PaymentService.cs`
3. Move payment logic from `OrderController.ProcessPayment()` into `PaymentService`
4. Update `OrderController` to accept `IPaymentService` via constructor injection

## Children

- 1 — PaymentController depends on concrete class
- 2 — OrderService uses PaymentProcessor directly
- 3 — InvoiceHandler casting to PaymentProcessor
- 4 — DI registration missing for IPaymentService
