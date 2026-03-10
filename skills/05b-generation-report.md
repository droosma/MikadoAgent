# Phase 5b: Generation Report

## Purpose

After each generation of exploration completes, present a status update to the user before proceeding to the next generation.

## Input

- Completed generation number
- Updated `session.md` and `mikado-graph.md`
- All node files from the current and previous generations

## Process

### 1. Summarize the Generation

```
Generation {N}: Explored {count} breaks, discovered {new_count} new breaks, {leaf_count} leaves found
```

### 2. List New Breaks

For each break discovered in this generation:

```
| ID | Description | Files | Status |
|----|-------------|-------|--------|
| 1.1 | PaymentDTO namespace conflict | PaymentDTO.cs | unexplored |
| 1.2 | Validator references concrete class | PaymentValidator.cs | unexplored |
| 2.1 | OrderTotalCalc missing dependency | OrderTotalCalc.cs | leaf |
```

### 3. Show Running Totals

```
Total nodes:    {count}
Leaves:         {count} (fixable directly)
Cycles:         {count}
Unexplored:     {count} (queued for next generation)
Generations:    {completed}/{estimated}
```

### 4. Display Current Mindmap

Show the current state of `mikado-graph.md` — the Mermaid mindmap rendered so far.

### 5. Continue

Proceed automatically to the next generation (Phase 5) unless:

- The user explicitly asks to pause
- All nodes are explored (→ Phase 6)

## Output

- Status report shown to user
- Automatic continuation to next generation or Phase 6
