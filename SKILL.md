---
name: system-reliability-auditor
description: "Use when auditing backend code for race conditions, lifecycle bugs, resource leaks, and concurrency reliability issues."
---

# System Reliability Auditor

## Overview

This skill analyzes backend code to identify **reliability issues related to concurrency, lifecycle management, and resource handling**.

It focuses on **provable bugs** such as race conditions, cancellation issues, resource leaks, and invalid lifecycle transitions.

The analysis uses **internal state-machine reasoning**, but outputs only concrete issues, evidence, and minimal fix plans.

---

## When to Use This Skill

Use this skill when:

- auditing backend services for reliability issues
- debugging race conditions or async bugs
- reviewing worker / orchestrator lifecycle logic
- checking shutdown or cancellation handling
- inspecting IPC, process managers, or async loops
- verifying safe resource cleanup (streams, processes, handles)

---

## Step-by-Step Guide

### 1. Analyze the Source Code

Inspect the provided code and identify modules responsible for:

- lifecycle management
- async processing
- worker orchestration
- resource allocation
- process management

Infer lifecycle phases internally such as:

- initialization
- running
- async operations
- cancellation
- shutdown

Only reason about states **clearly implied by the code**.

---

### 2. Identify State Transitions

Look for transitions triggered by:

- user actions
- timers
- async task completion
- process exit
- cancellation tokens
- IPC events
- failure paths

Track how modules move between lifecycle phases.

---

### 3. Detect Reliability Issues

Check for common reliability problems:

- race conditions
- cancellation tokens ignored
- resource leaks
- invalid lifecycle transitions
- partial initialization states
- stale cached state divergence
- cross-module state assumptions

Only report issues that can be **proven from the code**.

---

### 4. Produce Structured Findings

For each issue output:

ISSUE:

- Module / file
- Exact problematic logic
- Why it is dangerous

EVIDENCE:

- Code path proving the issue

FIX PLAN:

- Minimal fix required

RISK LEVEL:

LOW / MEDIUM / HIGH / CRITICAL

If evidence is incomplete, label:

UNCERTAIN (needs confirmation)

---

### 5. Verify Build and Tests

After suggesting fixes, attempt to verify the project using its native toolchain.

Examples:

**.NET**
```
dotnet build
dotnet test
```

**Node.js**
```
npm ci
npm test
```

**Python**
```
pytest
```

**Go**
```
go test ./...
```

**Rust**
```
cargo test
```

If the project has no tests, mark verification as **SKIPPED**.

---

## Examples

### Example: Audit backend worker loop

Prompt:
> Audit the following worker implementation for concurrency and lifecycle reliability issues.
> [PASTE CODE HERE]

Expected output format:
> ISSUE:
> WorkerLoop.cs – infinite loop ignoring cancellation token
> 
> EVIDENCE:
> while(true) loop with no CancellationToken checks
> 
> FIX PLAN:
> Use while(!token.IsCancellationRequested) and pass token to async calls
> 
> RISK LEVEL:
> HIGH

---

### Example: Audit process manager

Prompt:
> Review this process manager for resource leaks and lifecycle issues.
> [PASTE CODE HERE]

Expected output:
> ISSUE:
> ProcessManager.cs – child process not disposed
> 
> EVIDENCE:
> Process.Start used without Exited handler or Dispose()
> 
> FIX PLAN:
> EnableRaisingEvents = true and add cleanup during shutdown
> 
> RISK LEVEL:
> CRITICAL

---

## Best Practices

- ✅ Focus on **provable issues**
- ✅ Prefer **minimal targeted fixes**
- ✅ Use lifecycle reasoning internally
- ✅ Verify fixes using the project build system

- ❌ Do not invent speculative states
- ❌ Do not output large architecture diagrams
- ❌ Do not propose massive rewrites without evidence
