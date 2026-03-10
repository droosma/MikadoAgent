# Phase 3b: Analyze Breaks

## Purpose

Transform raw compiler/test errors into logical break nodes. Many errors share a single root cause — this phase clusters them and creates one node per logical break.

This is the **"Come up with immediate solutions"** + **"Draw prerequisites"** step of the Mikado Method.

## Input

- Raw error list from Phase 3 (file, line, error code, message)
- Parent node ID (the node whose change produced these errors)
- Session directory path

## Process

### 1. Group by Root Cause

Raw output often contains many errors that are symptoms of a single logical break. Cluster by:

**Separate compile errors from test failures before clustering.** Compiler errors are deterministic — the same code always produces the same error. Test failures may be environment-dependent, order-dependent, or flaky. Cluster each category independently, then merge only if a test failure clearly shares the same root cause as a compile error (e.g., a missing type causes both CS0246 and a test that references that type).

**Same error code + same missing symbol/type/member:**

```
src/Orders/OrderController.cs(23,15): error CS0246: 'PaymentProcessor' not found
src/Billing/InvoiceService.cs(45,8): error CS0246: 'PaymentProcessor' not found
src/Reports/ReportGenerator.cs(12,20): error CS0246: 'PaymentProcessor' not found
```

→ ONE break: "Update all references to `PaymentProcessor`" (3 files affected)

**Same test failure pattern:**

```
FAILED OrderTests.ProcessPayment_ShouldDeductAmount
FAILED OrderTests.ProcessPayment_ShouldNotifyCustomer
FAILED OrderTests.ProcessPayment_ShouldLogTransaction
```

→ ONE break if all fail for the same reason (e.g., missing dependency injection binding)

### 2. Identify Distinct Breaks

After clustering, each group becomes a distinct logical break. For each:

- Summarize **what broke** (one sentence)
- List **which files** are affected
- Describe the **obvious fix** (the immediate solution — don't overthink)

A single solution can sometimes take care of hundreds of errors with the same root cause. **Draw only one prerequisite per logical break.**

### 3. Check for Cycles

For each logical break, compare against ALL existing `nodes/*.md` files:

- Same file(s) affected + same error signature = **cycle**
- If a cycle is detected:
  - Create the node file but set Status to `cycle`
  - Add a `Cycle Reference` field linking to the existing node
  - Do NOT recurse into this node

### 4. Create Node Files

For each distinct break (that isn't a cycle), create `nodes/{id}-{slug}.md`:

**ID assignment:** `{parentId}.{sequence}` where sequence starts at 1.

- Parent `0` → children are `1`, `2`, `3`, ...
- Parent `1` → children are `1.1`, `1.2`, `1.3`, ...
- Parent `1.2` → children are `1.2.1`, `1.2.2`, ...

**Slug:** Short kebab-case description (e.g., `payment-validator-ns-conflict`)

```markdown
# Node {id} — {Short Description}

## Identity
- **ID:** {id}
- **Parent:** {parentId} ({parent description})
- **Depth:** {depth}
- **Status:** unexplored

## Context
- **File(s):** {affected files, comma-separated}
- **Line(s):** {relevant line ranges}
- **Worktree branch:** mikado/{session-name}-{nodeId-path}

## Break Description
{One paragraph: what broke and why. Reference the parent change that caused it.}

## Error Output
```text
{Representative error output — not all 30 lines, just enough to show the pattern}
```

## Proposed Fix

{The immediate, obvious solution. Be specific: what to change, where, and how.
E.g., "Change `PaymentValidator` constructor to accept `IPaymentService` instead
of `PaymentProcessor`. Update the validation call on line 52."}

## Children

(to be discovered)

```

### 5. Count and Summarize
Produce a summary for the orchestrator:
```

Parent node: {parentId}
Errors analyzed: {total raw errors}
Logical breaks found: {count}
Cycles detected: {count}
New node IDs: {list of IDs created}

```

## Output
- Node files created in `nodes/` directory
- Summary report: list of new node IDs, their statuses, cycle detections
- Passed to Phase 4: Report to Orchestrator

## Key Principles

### Cluster Aggressively
When in doubt, merge errors into fewer breaks. One well-described break node is better than ten near-duplicate nodes. You can always split a node later if its fix reveals it was actually multiple breaks.

### The Fix Should Be Obvious
If the fix isn't obvious, describe what you know and mark it clearly. The Mikado Method works because each prerequisite's fix is a direct consequence of the parent change — it should be straightforward.

### Don't Fix, Document
You are not fixing the code here. You are recording what broke, why, and what the fix would be. The actual fix happens later, leaf-first, in the resolution phase.

### Explore Options (Alternative Branches)

When a break has multiple plausible fixes (e.g., "inject `IPaymentService` via constructor" vs. "use a service locator"), and it’s not obvious which is better:

1. Create **one node per option**, using the same parent and sequential IDs.
2. Prefix the node description with `[Option A]` / `[Option B]`.
3. Add a `## Alternatives` section to each option node referencing the other(s).
4. Both are explored in Phase 5 like any other node.
5. In Phase 6 (Finalize), flag the alternatives to the user: "Nodes X and Y are alternative approaches to the same break — choose one and prune the other from the resolution order."

This is rare. Most breaks have one obvious fix. Only create alternatives when the trade-off is genuinely unclear and the user would benefit from seeing both paths explored.
