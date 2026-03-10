# Phase 3c: Cross-Solution Scan (Optional)

## Purpose

For multi-solution repositories (multiple `.sln` files), check if changes in the current exploration would impact other solutions.

This is **informational only** — the agent reports potential cross-solution impact but does NOT automatically explore other solutions.

## Input

- Completed node exploration for the primary solution
- List of all files changed across the current generation's nodes
- Session metadata (primary solution path)

## When to Run

- Only for repos with multiple `.sln` files
- After the primary solution tree for a generation is fully explored
- Skip if the repo has only one `.sln` file

## Process

### 1. Discover Other Solutions

```bash
find . -name "*.sln" -not -path "./.worktrees/*" | sort
```

Exclude the primary solution (already being explored).

### 2. Identify Changed Files/Types

Collect from all node files created in this generation:

- Files listed in the `File(s)` field
- Types/symbols mentioned in break descriptions
- Public API changes (interface changes, removed methods, renamed types)

### 3. Scan for References

For each other `.sln` file:

- Parse its project references to find which `.csproj` files it includes
- Check if any of those projects reference the changed files/types
- Use grep/search to find usages of changed symbols

### 4. Report Findings

For each affected solution, report:

```markdown
## Cross-Solution Impact: {SolutionName}.sln

### Potentially Affected Files
- `src/OtherProject/ServiceClient.cs` — references `PaymentProcessor` (renamed in node 1)
- `tests/IntegrationTests/PaymentFlow.cs` — uses `IPaymentService` (modified in node 1.2)

### Recommendation
These files may require updates when the Mikado resolution is applied.
Consider running a separate Mikado session for this solution if changes are significant.
```

## Output

- List of potentially affected solutions with file references
- Informational only — no node files created, no automatic fan-out

## Key Rules

- This phase is **optional** and **read-only** with respect to the Mikado Graph
- Do NOT create nodes for other solutions
- Do NOT automatically fan out into other solutions
- Only report findings to the user for awareness
- The user must explicitly request exploration of other solutions
