# Phase 6: Finalize Graph

## Purpose

Assemble the final Mikado Graph, compute the resolution order, clean up all worktrees, and present the complete results to the user.

## Input

- Complete session with all nodes explored (no `unexplored` nodes remain)
- All `nodes/*.md` files
- `session.md`

## Process

### 1. Final Graph Rebuild

Scan all `nodes/*.md` files one last time. Build the complete tree structure from ID and Parent fields.

Render the final `mikado-graph.md`:

````markdown
# Mikado Graph — {Goal Title}

```mermaid
mindmap
  root(({Goal Title}))
    1[Break 1]
      1.1[Sub-break A]
      1.2[Sub-break B]
    2[Break 2]
      2.1[Sub-break C]
    3[Break 3 — leaf]
    4[Break 4 — leaf]
```

## Legend
- **Root (double circle):** The target refactor goal
- **Branch (square):** A break that has its own prerequisites
- **Leaf (square, no children):** A break that can be fixed directly
- **Cycle:** (marked with dashed line if present) A break that references an already-known break

## Resolution Order
Apply fixes leaf-first, bottom-up:
1. 1.1 — Sub-break A
2. 1.2 — Sub-break B
3. 2.1 — Sub-break C
4. 3 — Break 3
5. 4 — Break 4
6. 1 — Break 1
7. 2 — Break 2
8. 0 — {Goal Title} (root goal)

## Quick Reference
For each step, see the corresponding node file in `nodes/` for:
- What to change
- Which files to edit
- The proposed fix
````

### 2. Compute Resolution Order

Topological sort of the graph, leaves first:

1. Start with all leaf nodes (no children)
2. After all leaves of a parent are resolved, the parent becomes resolvable
3. Continue until the root is reached

For each entry in the resolution order, include:

- Node ID
- Short description
- Status (leaf / branch)
- Number of files affected

### 3. Handle Cycles

If any cycle nodes exist:

- List them separately under a "Cycles Detected" section
- Explain that cycles indicate the same break appearing in multiple paths
- The cycle node does not need to be fixed separately — fixing the original node handles it

### 4. Clean Up All Worktrees

```bash
# Remove all Mikado worktrees
git worktree list --porcelain | grep -A1 "worktree.*\.worktrees/{session-name}" | ...
# For each:
git worktree remove --force {path}
git branch -D mikado/{session-name}-{nodeId-path}
```

Remove the `.worktrees/{session-name}-*` directories.

### 5. Update Session File

Set `session.md` Status to `completed`. Update final counts:

```markdown
## Progress
- **Total nodes:** {final count}
- **Leaves (fixable):** {count}
- **Cycles:** {count}
- **Unexplored:** 0
- **Generations completed:** {count}
- **Status:** completed
```

### 6. Present Summary to User

```
═══════════════════════════════════════════════════
  Mikado Session Complete: {Goal Title}
═══════════════════════════════════════════════════

  Total breaks discovered:  {count}
  Leaf nodes (directly fixable): {count}
  Cycle detections: {count}
  Generations explored: {count}

  Resolution order: {count} steps, leaf-first

  Session files:
    doc/refactor/{session-id}/mikado-graph.md
    doc/refactor/{session-id}/session.md
    doc/refactor/{session-id}/nodes/*.md

  Next steps:
    Apply fixes in resolution order (leaf-first).
    Each node file contains the proposed fix.
    Start with step 1 and work through to the root.
═══════════════════════════════════════════════════
```

## Output

- Final `mikado-graph.md` with complete mindmap and resolution order
- Updated `session.md` with completion status
- All worktrees cleaned up
- Summary presented to user

## Post-Session

The session directory remains in the repository as documentation:

- `mikado-graph.md` serves as a visual plan for the refactoring
- `nodes/*.md` files contain detailed fix instructions per step
- `session.md` has the overall session metadata

The user (or a future agent phase) applies fixes leaf-first following the resolution order.
