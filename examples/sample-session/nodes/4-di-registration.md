# Node 4 — DI registration missing for IPaymentService

## Identity

- **ID:** 4
- **Parent:** 0 (Extract Payment Service)
- **Depth:** 1
- **Status:** leaf

## Context

- **File(s):** src/Program.cs
- **Line(s):** 24
- **Worktree branch:** mikado/extract-payment-4

## Break Description

After extracting `IPaymentService` (node 0), the application fails at startup
because `IPaymentService` is not registered in the DI container. All controllers
and services requesting `IPaymentService` via constructor injection get a
runtime `InvalidOperationException`.

## Error Output

```text
System.InvalidOperationException: Unable to resolve service for type 'Payment.IPaymentService' while attempting to activate 'Controllers.PaymentController'.
```

## Proposed Fix

Add DI registration in `Program.cs`:

```csharp
builder.Services.AddScoped<IPaymentService, PaymentService>();
```

## Children

None (leaf node)
