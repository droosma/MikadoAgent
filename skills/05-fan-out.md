# Phase 5: Fan Out

## Purpose

Spawn parallel sub-agents to explore each break from the current generation. Each sub-agent works in its own worktree, applies the fix described in its node, and runs the full Phase 3 → 3b → 4 pipeline.

This is the **recursive** heart of the Mikado Method: each prerequisite becomes a sub-goal, explored independently.

## Input

- List of `unexplored` node IDs from Phase 3b (or from resume scan)
- Session metadata (base branch, language profile, session name)

## Process

### 1. Identify Exploration Targets

Scan `nodes/*.md` for all nodes with `Status: unexplored`.

### 2. Spawn Sub-Agents (Parallel)

For each unexplored node, spawn a sub-agent that executes:

```
Phase 3: Attempt Change (apply fix in new worktree)
    ↓
Phase 3b: Analyze Breaks (cluster errors, create child nodes)
    ↓
Phase 4: Report to Orchestrator (return list of new node IDs)
```

Each sub-agent:

- Gets its own git worktree branching from the base branch
- Applies the fix described in its node file
- Runs build/test/analysis
- Creates child node files if breaks are found
- Reports back with new node IDs

### 3. Collect Reports

Wait for all sub-agents in this generation to complete. Collect their reports.

### 4. Update Node Statuses

For each explored node:

- If it was a leaf (no breaks) → status already set to `leaf` by Phase 3
- If it had breaks → status is `in-progress` (children exist)

Update the parent node file's Status field:

- Has children → keep as `in-progress`
- Was successful → `leaf`

### 5. Rebuild Mikado Graph

Scan all `nodes/*.md` files and reconstruct the mindmap:

```python
# Pseudocode
for each node file:
    parse ID, Parent, Status, Description
    add to tree structure

render as Mermaid mindmap:
    root((Goal))
        1[Break 1]
            1.1[Sub-break]
        2[Break 2]
        ...
```

Write updated `mikado-graph.md`.

### 6. Update Session Progress

Update `session.md` with:

- Total nodes count
- Leaves count
- Cycles count
- Unexplored count
- New generation log entry

### 7. Check Termination

If there are still `unexplored` nodes → proceed to Phase 5b (generation report), then recurse (next generation of Phase 5).

If NO `unexplored` nodes remain → proceed to Phase 6 (finalize).

### 8. Clean Up Worktrees

After a generation completes, clean up worktrees for explored nodes:

```bash
git worktree remove .worktrees/{session-name}-{nodeId-path}
git branch -D mikado/{session-name}-{nodeId-path}
```

Only clean up worktrees for nodes that are `leaf` or `in-progress` (fully explored). Keep worktrees for `unexplored` nodes if they exist (shouldn't normally).

## Output

- Updated `mikado-graph.md` (rebuilt from all node files)
- Updated `session.md` (progress counts, generation log)
- Either: proceed to next generation (more unexplored nodes)
- Or: proceed to Phase 6 (all nodes explored)

## Concurrency Model

### What's Shared (Read-Only)

- `session.md` — sub-agents read base branch, language profile, solution path
- Existing `nodes/*.md` files — sub-agents read for cycle detection

### What's Written (No Conflicts)

- New `nodes/*.md` files — each sub-agent creates uniquely-named files (IDs are unique)
- Each sub-agent's git worktree — completely isolated

### What the Orchestrator Writes (After Generation)

- `mikado-graph.md` — rebuilt from all node files
- `session.md` — updated progress counts and generation log
- These are only written AFTER all sub-agents complete, never concurrently

## Resume Support

To resume an interrupted session:

1. Scan `nodes/*.md` for nodes with `Status: unexplored`
2. Re-enter Phase 5 with those nodes as the exploration targets
3. The graph is rebuilt from existing node files automatically
