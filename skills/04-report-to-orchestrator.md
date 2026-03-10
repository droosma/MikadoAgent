# Phase 4: Report to Orchestrator

## Purpose

Sub-agents report their exploration results back to the orchestrator. Sub-agents do NOT update `mikado-graph.md` directly — they only create node files.

This phase defines the handoff protocol between a sub-agent (exploring one break) and the orchestrator (managing the overall graph).

## Input

- Completed Phase 3 + Phase 3b results for a specific node
- Node files created by the sub-agent

## Process

### 1. Compile Report

The sub-agent produces a structured report:

```markdown
## Sub-Agent Report: Node {parentId}

### Exploration Result
- **Parent node:** {parentId} — {parent description}
- **Change attempted:** {what was changed}
- **Result:** breaks_found | leaf | error

### New Nodes Created
| ID | Description | Status | Files Affected |
|----|-------------|--------|----------------|
| {id} | {description} | unexplored | {files} |
| {id} | {description} | cycle | {files} |

### Cycle Detections
- Node {id} is a cycle back to node {existingId}: {reason}

### Summary
- New breaks: {count}
- Cycles: {count}
- Total new nodes: {count}
```

### 2. Return to Orchestrator

The sub-agent terminates and returns the report. It does NOT:

- Write to `mikado-graph.md`
- Update `session.md`
- Modify any node file it didn't create
- Touch any other sub-agent's worktree

### 3. Orchestrator Receives Report

The orchestrator (Phase 5) collects reports from all sub-agents in the current generation and:

- Validates that all reported node files exist
- Checks for any conflicts (shouldn't happen since IDs are unique)
- Queues newly created `unexplored` nodes for the next generation
- Rebuilds `mikado-graph.md` from all `nodes/*.md` files

## Output

- Structured report to orchestrator with list of new node IDs and statuses
- No shared files modified — only the sub-agent's own node files

## Key Rules

### Sub-agents Are Independent

Each sub-agent works in its own worktree and creates its own node files. There are no shared mutable resources between sub-agents.

### The Orchestrator Owns the Graph

Only the orchestrator writes to `mikado-graph.md` and `session.md`. This eliminates all concurrency conflicts.

### Node Files Are the Source of Truth

`mikado-graph.md` is a **derived artifact** — always regenerable from `nodes/*.md` files. If a sub-agent crashes, the orchestrator can rebuild the graph from whatever node files exist.
