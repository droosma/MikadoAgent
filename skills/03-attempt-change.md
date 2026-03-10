# Phase 3: Attempt Change

## Purpose

Apply the change described in a node's file within an isolated git worktree, then run build/test/analysis to collect all resulting errors.

This is the **Naive Approach** from the Mikado Method: make the change directly and let the code tell you what's wrong.

## Input

- Node ID and its node file (`nodes/{id}-{slug}.md`)
- Session metadata from `session.md` (base branch, language profile, solution path)

## Process

### 1. Create Git Worktree

```bash
git worktree add .worktrees/{session-name}-{nodeId-path} -b mikado/{session-name}-{nodeId-path} {base-branch}
```

Where `{nodeId-path}` replaces dots with dashes: node `1.5.10` → `1-5-10`.

**Crash recovery**: If the worktree already exists (from a previous interrupted run):

```bash
git worktree remove --force .worktrees/{session-name}-{nodeId-path}
git branch -D mikado/{session-name}-{nodeId-path}
```

Then recreate from clean base branch.

### 2. Restore Dependencies

In the worktree, restore dependencies before building. This is required for fresh worktrees.

```bash
cd .worktrees/{session-name}-{nodeId-path}
dotnet restore  # or equivalent from language profile
```

### 3. Apply the Change (The Naive Approach)

Read the node file's **Proposed Change** / **Proposed Fix** section. Apply that change to the code in the worktree.

This is where the LLM does actual code editing:

- For the **root node (0)**: Apply the refactoring goal directly (e.g., extract a class, move a method, rename a type)
- For **break nodes**: Apply the fix described in the node file (e.g., update callers to use the new interface, add a missing using statement)

The change should be **straightforward** — each break's fix is a direct consequence of its parent change. Don't overthink it. If the fix is unclear, make the most obvious change and let the build tell you what else breaks.

### 4. Run Build

Execute the build command from the language profile:

```bash
dotnet build --no-restore
```

Capture all output (stdout + stderr).

### 5. Run Tests

Execute the test command from the language profile:

```bash
dotnet test --no-restore --no-build
```

If the language profile supports scoping tests to affected projects (via project references), scope them. Otherwise, run the full test suite.

Capture all output.

**Flaky test detection:** If only 1-3 tests fail, re-run just the failing tests once. If a test passes on re-run, exclude it from the error list — it’s flaky, not a break. If it fails consistently on both runs, it’s a real break. Log flaky tests in the node file under a `## Flaky Tests` section for the user’s awareness.

### 6. Run Static Analysis

Execute the static analysis command from the language profile (if configured):

```bash
dotnet build /p:TreatWarningsAsErrors=true /p:EnforceCodeStyleInBuild=true
```

Capture all output.

### 7. Parse Errors

Using the language profile's error parsing patterns, extract structured errors:

```
{file}({line},{col}): error {code}: {message}
```

Collect into a list:

```json
[
  { "file": "src/Payment/PaymentValidator.cs", "line": 47, "col": 12, "code": "CS0246", "message": "The type or namespace name 'PaymentProcessor' could not be found" },
  ...
]
```

### 8. Handle Error Volume

If the error count exceeds a reasonable context limit (e.g., 200+ errors):

- **Paginate**: Process errors in batches
- **Warn the user**: "This change produced {N} errors — the change may be too large. Consider breaking the refactor into smaller steps."
- Still proceed with analysis, but flag the scale concern

### 9. Determine Result

**If NO errors** (build succeeds, tests pass, analysis clean):

- This node is a **leaf** — the change can be applied directly
- Update node file: set Status to `leaf`
- The worktree can be cleaned up

**If errors exist**:

- Pass the raw error list to Phase 3b: Analyze Breaks
- The worktree will be discarded (the graph holds the knowledge)

## Output

- Raw list of errors (file, line, code, message) — passed to Phase 3b
- OR: empty list → node marked as `leaf`

## Key Principle: Don't Fix Forward

If the change broke things, do NOT try to fix them in this worktree. Collect the errors and move to Phase 3b. The worktree will be discarded. This is the **revert** step of the Mikado Method — the graph captures what you learned.
