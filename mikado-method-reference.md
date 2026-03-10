# The Mikado Method — Reference for LLM Agents

> Distilled from "The Mikado Method" by Ola Ellnestam & Daniel Brolund.
> This document provides the authoritative method description that the Mikado Agent must follow.

## Contents

1. [What Is the Mikado Method?](#1-what-is-the-mikado-method)
2. [The 9-Step Process](#2-the-9-step-process)
3. [Core Principles](#3-core-principles)
4. [The Mikado Graph](#4-the-mikado-graph)
5. [Working with Tests](#5-working-with-tests)
6. [Terminology](#6-terminology)
7. [Common Refactoring Patterns Encountered](#7-common-refactoring-patterns-encountered)
8. [Key Anti-Patterns to Avoid](#8-key-anti-patterns-to-avoid)
9. [Adaptation for Automated Agents](#9-adaptation-for-automated-agents)

---

## 1. What Is the Mikado Method?

The Mikado Method is a structured, non-destructive approach to making large-scale changes to codebases. It works by:

1. **Attempting** a desired change naively
2. **Observing** what breaks (compiler errors, test failures, runtime exceptions)
3. **Recording** the breaks as prerequisites in a dependency graph
4. **Reverting** the code to its clean state (the graph holds the knowledge, not the code)
5. **Recursing** into each prerequisite until leaf nodes are found (changes that succeed without breaking anything)
6. **Applying** fixes leaf-first, bottom-up, each one a small, safe, check-in-able change

The key insight: **reverting is not losing work**. The Mikado Graph captures everything learned. The code stays in a working state at all times.

---

## 2. The 9-Step Process

### Step 1: Draw the Mikado Goal

Write down the desired end state as a short, clear description. Draw it as a double circle (root node). The goal should be formulated in terms of business value when possible.

### Step 2: Implement the Goal Naively (The Naive Approach)

Make the change directly — don't overthink it. Just do the most obvious thing. The purpose is to let the code tell you what's wrong, not to get it right on the first try.

### Step 3: Find Any Errors

Run the build and tests. Collect all compiler errors, test failures, and runtime exceptions. These are the "breaks" — the things that prevent the goal from being achieved.

### Step 4: Come Up with Immediate Solutions to the Errors

For each break, identify the most straightforward fix. Don't solve deeply — just the immediate, obvious solution. **A single solution can sometimes take care of hundreds of errors with the same root cause — draw only one prerequisite.**

### Step 5: Draw the Immediate Solutions as New Prerequisites

Add each solution as a new node in the graph. Draw an arrow from the goal to each prerequisite (the goal depends on the prerequisite being done first).

### Step 6: Revert the Code to the Initial State

**CRITICAL**: If there were errors, revert ALL code changes. Go back to the clean, working state. The graph holds the knowledge — the code should remain untouched.

This is the hardest step psychologically. The temptation to keep partial changes is strong. Resist it. The Mikado Method's power comes from always having working code.

### Step 7: Repeat for Each Prerequisite

Pick one prerequisite at a time. Start from a clean state. Apply that prerequisite's change naively. See what breaks. Record new prerequisites. Revert. Recurse.

Each prerequisite becomes a sub-goal. Its breaks become its prerequisites. The graph grows deeper.

### Step 8: Check In If There Are No Errors

When a change succeeds (builds, tests pass, makes sense), it's a **leaf node**. Commit it. This is a safe, small, non-breaking change.

### Step 9: If the Mikado Goal Is Met, You're Done

Once all prerequisites are satisfied leaf-first and the root goal change succeeds, the refactoring is complete.

---

## 3. Core Principles

### 3.1 Always Revert on Failure

Never leave broken code in the repository. The graph is your memory. The code must always be in a working state.

### 3.2 The Graph IS the Knowledge

When you revert code, you haven't lost anything. The Mikado Graph captures:

- What you tried
- What broke
- Where it broke
- How to fix it
- The dependency order

### 3.3 One Root Cause, One Node

Many errors can share a single root cause. When renaming a class produces CS0246 in 30 files, that's ONE break: "update all references to OldClass". Don't create a node per error — create a node per logical break.

### 3.4 Work Leaf-First

Fixes are applied bottom-up. Start with leaves (nodes with no children), apply their fixes, verify they work, commit. Then move to their parents. Eventually reach the root.

### 3.5 Keep the Code Working

The system is always in a deliverable state. Every committed change is small, safe, and non-breaking. This is what makes the method safe for production codebases.

### 3.6 Small, Boring Changes

The Mikado Method transforms risky, large changes into boring, small ones. Each leaf-level fix is trivial in isolation. The complexity is managed by the graph structure, not by the code changes.

---

## 4. The Mikado Graph

### 4.1 Structure

- **Root node** (double circle): The Mikado Goal — the desired end state
- **Branch nodes** (squares): Prerequisites that themselves have prerequisites
- **Leaf nodes** (squares, no children): Changes that can be applied directly without breaking anything
- **Arrows**: Point from a goal/node to its prerequisites (dependency direction)

### 4.2 Graph Properties

- **Directed Acyclic Graph (DAG)**: Arrows go from goal → prerequisites. Cycles indicate a design problem.
- **Resolution order**: Topological sort, leaves first. Each leaf can be committed independently.
- **Concurrent goals**: Multiple goals can share prerequisites. Drawing them in the same graph reveals common hot spots.

### 4.3 Node Content

Each node should contain:

- A short, clear description of what needs to change
- Enough detail for someone to pick it up after a break
- The files/locations affected (if known)

### 4.4 Graph Patterns (from the book)

#### Group Common Prerequisites

When multiple nodes depend on the same prerequisite, create a grouping node to reduce visual clutter.

#### Extract Preparatory Prerequisites

Tasks that must be done before exploring (e.g., code cleanup, setting up test infrastructure) should be extracted to a separate graph or completed first.

#### Concurrent Goals with Common Prerequisites

When multiple goals share prerequisites, draw them in the same graph. Common hot spots become visible and can be prioritized.

#### Split Large Graphs

Break large graphs into subgraphs referenced by a placeholder node. Each subgraph should be self-contained with no arrows back to the parent graph.

#### Explore Options

When multiple solutions exist for a break, draw them as alternative branches from a split-dependency arrow. Explore each option far enough to understand consequences before choosing.

---

## 5. Working with Tests

### 5.1 Cover Then Modify

The recommended approach: first cover existing code with tests, then modify it. Adding tests is itself a Mikado-style process — the test may reveal that the code isn't testable, creating prerequisites (extract interface, inject dependency, etc.).

### 5.2 Tests as Break Detection

In statically typed languages, the compiler catches many breaks. In dynamically typed languages, tests are the primary break detection mechanism. The method works the same either way — the feedback source just differs.

### 5.3 What Counts as a Break

- **Compiler errors**: Missing types, wrong signatures, namespace issues
- **Test failures**: Assertion failures, runtime exceptions
- **Static analysis warnings**: When treated as errors (e.g., Roslyn analyzers)
- **Runtime exceptions**: Null references, missing dependencies

---

## 6. Terminology

| Term | Definition |
|------|-----------|
| **Mikado Goal** | The root node — the desired end-state change |
| **Prerequisite** | A change that must be completed before its parent can succeed |
| **Leaf** | A prerequisite with no children — can be applied directly |
| **Naive Approach** | Making a change directly to see what breaks, without planning ahead |
| **Break** | An error/failure caused by attempting a change |
| **Revert** | Undoing all code changes after observing breaks (graph keeps the knowledge) |
| **Check in** | Committing a successful leaf-level change |
| **Resolution order** | The bottom-up sequence for applying fixes (leaves first) |
| **Grouping node** | A node that groups related prerequisites to reduce graph clutter |
| **Decision node** | A node representing a choice between alternative solutions |
| **Unexplored node** | A prerequisite that hasn't been attempted yet |

---

## 7. Common Refactoring Patterns Encountered

These are the types of breaks and solutions the method commonly reveals:

### 7.1 Code Doing Too Many Things (SRP Violation)

**Break**: Can't move/reuse functionality because it's entangled with other responsibilities.
**Fix**: Extract responsibilities into separate classes/methods.

### 7.2 Code Unstable in Face of Change (OCP Violation)

**Break**: Adding a feature requires modifying existing code in many places (e.g., growing if-else chains).
**Fix**: Replace conditionals with polymorphism/strategy pattern.

### 7.3 Contract Violations (LSP Violation)

**Break**: Subclass doesn't behave like its base class, causing runtime errors.
**Fix**: Replace inheritance with composition; extract proper interfaces.

### 7.4 Bulky Interfaces (ISP Violation)

**Break**: Can't move code to new package because interface is too large.
**Fix**: Extract client-specific interfaces from the large interface.

### 7.5 Dependency Direction Problems (DIP Violation)

**Break**: High-level module depends on implementation details, making it hard to change.
**Fix**: Introduce abstractions; depend on interfaces, not implementations.

### 7.6 Circular Dependencies

**Break**: Modules depend on each other in cycles, preventing independent changes.
**Fix**: Break the cycle by introducing an abstraction/interface at one edge. Explore options (which edge to break) using the graph.

### 7.7 Scattered Code

**Break**: Related code is spread across multiple packages/projects.
**Fix**: Merge packages → reorganize → split into cohesive units. Or: create API/facade and gradually move code behind it.

### 7.8 Missing Seams

**Break**: Can't test or modify behavior without editing the code directly.
**Fix**: Extract method → make it overridable or inject dependency.

---

## 8. Key Anti-Patterns to Avoid

1. **Not reverting**: Keeping partial/broken changes defeats the method. Always revert.
2. **Too-granular nodes**: One node per compiler error instead of per logical break. Cluster errors by root cause.
3. **Not using the graph**: Working from memory instead of maintaining the graph. The graph is the single source of truth.
4. **Big-bang changes**: Trying to fix everything at once instead of leaf-first. Each commit should be small and safe.
5. **Ignoring the Naive Approach**: Over-planning instead of letting the code tell you what's wrong. The naive attempt is how you discover prerequisites.
6. **Keeping dead-end explorations**: If an option exploration shows a path is undesirable, prune that branch from the graph.

---

## 9. Adaptation for Automated Agents

When an LLM agent executes the Mikado Method:

1. **The Naive Approach maps to**: Apply the change in an isolated worktree, run build/test, collect errors programmatically.
2. **Reverting maps to**: Discarding the worktree and returning to the clean base branch. The agent never modifies the main working tree.
3. **Drawing the graph maps to**: Creating node files (one per logical break) and assembling a Mermaid mindmap.
4. **Clustering errors**: The agent must group compiler/test errors by root cause. 30 CS0246 errors for the same missing type = 1 node.
5. **Cycle detection**: Before creating a new node, check existing nodes for the same file + error signature. If found, mark as cycle.
6. **Parallel exploration**: Each prerequisite can be explored independently in its own worktree, enabling parallel fan-out.
7. **The graph is the deliverable**: The agent's primary output is the documented Mikado Graph with resolution order — not the fixed code.
