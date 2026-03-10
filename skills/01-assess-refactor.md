# Phase 1: Assess Refactor

## Purpose

Interview the user about their desired refactoring, clarify the goal, and assess feasibility before committing to a Mikado session.

## Input

User describes a refactoring they want to make.

## Process

### 1. Clarify the Goal

Ask targeted questions to understand:

- **What** should the end state look like? (e.g., "PaymentService extracted behind IPaymentService interface")
- **Why** is this change needed? (business value, testability, modularity)
- **Where** in the codebase does this change start? (specific files, classes, namespaces)

Formulate the goal in clear terms. Prefer business-value framing when possible:

- Good: "Extract payment processing to enable unit testing and pluggable providers"
- Avoid: "Refactor PaymentController" (too vague)

### 2. Assess Scope

Examine the codebase around the target area:

- Identify the primary solution/project files
- Count approximate files and dependencies that may be affected
- Check for existing test coverage
- Look for obvious dependency chains

### 3. Evaluate Feasibility

Consider:

- Does the codebase build cleanly right now? (prerequisite for Phase 2)
- Is the refactoring goal achievable without changing external interfaces?
- Are there obvious blockers (e.g., no tests, circular dependencies)?

### 4. Produce Refactor Summary

Write a summary containing:

```
Goal: [one-sentence description]
Scope: [files/projects affected]
Starting point: [specific file(s) and what to change]
Expected break areas: [educated guess of what will break]
Risk level: low | medium | high
Naive attempt: [the single most direct code change that expresses the intent]
```

The **naive attempt** should be the smallest concrete change that triggers the refactoring’s cascading impact. For example:

- Good: "Create `IPaymentService` interface and change `OrderController` constructor to accept it"
- Too broad: "Extract payment service" (not specific enough to attempt naively)
- Too detailed: "Create interface, move methods, update DI, update all callers" (that’s the whole solution, not a naive attempt)

### 5. Get User Confirmation

Present the summary and get explicit go/no-go before proceeding to Phase 2.

## Output

- Go/no-go decision
- Refactor goal definition (goal text, starting files, language profile to use)
- Passed to Phase 2: Initialize Session

## Decision Rules

- If baseline build/tests don't pass → recommend fixing first, no-go
- If scope is trivially small (1-2 files, no cascading breaks expected) → suggest doing it directly without the Mikado Method
- If the goal is unclear after clarification → ask more questions, don't guess
- If the goal is too broad (multiple independent refactorings bundled together, e.g., "extract 10 services from the monolith") → recommend splitting into separate Mikado sessions, one per logical goal. Each session should have a single, focused Mikado Goal.
- If the naive attempt isn't obvious → help the user find the single smallest code change that expresses the intent. If you can't identify one, the goal likely needs to be narrowed.
