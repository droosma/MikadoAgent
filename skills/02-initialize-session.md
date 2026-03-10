# Phase 2: Initialize Session

## Purpose

Create the session directory, verify the baseline is clean, and set up the root node for the Mikado Graph.

## Input

Approved refactor definition from Phase 1 (goal text, starting files, language profile).

## Process

### 1. Load Language Profile

Read `languages/{lang}.md` to get:

- Build command
- Test command
- Static analysis command
- Error parsing patterns
- Solution/project discovery rules

### 2. Discover Solution/Project

Using the language profile's discovery rules:

- Find the primary `.sln` file (C#) or equivalent
- Identify all projects in the solution
- Record the solution path in session metadata

### 3. Create Session Directory

Generate session ID: `{yyyyMMdd}-{seq}-{name}`

- Date: current date
- Seq: next sequential number for that date (check existing dirs)
- Name: slugified goal (e.g., `extract-payment-service`)

Create directory structure:

```
doc/refactor/{session-id}/
├── session.md
└── nodes/
```

### 4. Run Baseline Verification

Execute build + test commands from the language profile against the **main working tree** (not a worktree):

```bash
# Example for C#
dotnet restore
dotnet build --no-restore
dotnet test --no-restore --no-build
```

**If baseline fails → ABORT**: Print message "Cannot start Mikado session — baseline build/tests do not pass. Fix existing issues first." and stop.

### 5. Record Base Branch

```bash
git rev-parse --abbrev-ref HEAD
```

Store this as the base branch in `session.md`. All worktrees will branch from this.

### 6. Ensure .gitignore

Add `.worktrees/` to `.gitignore` if not already present.

### 7. Create Session File

Write `session.md`:

```markdown
# Session: {Goal Title}

## Metadata
- **Created:** {ISO timestamp}
- **Session ID:** {session-id}
- **Base branch:** {branch name}
- **Language profile:** {lang}
- **Solution:** {solution path}
- **Status:** in-progress

## Refactor Goal
{Goal description from Phase 1}

## User Intent
{Why the refactor is needed, from Phase 1}

## Progress
- **Total nodes:** 1
- **Leaves (fixable):** 0
- **Cycles:** 0
- **Unexplored:** 0
- **Generations completed:** 0

## Generation Log
| Gen | Breaks Explored | New Breaks Found | Notes |
|-----|----------------|-----------------|-------|
```

### 8. Create Root Node File

Write `nodes/0-{goal-slug}.md`:

```markdown
# Node 0 — {Goal Title}

## Identity
- **ID:** 0
- **Parent:** (none — root)
- **Depth:** 0
- **Status:** unexplored

## Context
- **File(s):** {starting files from Phase 1}
- **Worktree branch:** mikado/{session-name}-0

## Goal Description
{Full goal description}

## Proposed Change
{What change to attempt — the naive approach from Phase 1}

## Children
(to be discovered)
```

### 9. Initialize Mikado Graph

Write `mikado-graph.md`:

````markdown
# Mikado Graph — {Goal Title}

```mermaid
mindmap
  root(({Goal Title}))
```

## Legend
- **Root (circle):** The target refactor goal
- **Branch (square):** A break that caused further breaks
- **Leaf (square, no children):** A break that can be fixed directly

## Resolution Order
(to be computed after exploration)
````

## Output

- Session directory with `session.md`, `mikado-graph.md`, and `nodes/0-*.md`
- Verified clean baseline (build + tests pass)
- Ready for Phase 3: Attempt Change on root node (ID: 0)
