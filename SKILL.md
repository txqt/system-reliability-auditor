---
name: system-reliability-auditor
description: "Use when auditing backend code for race conditions, lifecycle bugs, resource leaks, or concurrency reliability issues."
risk: none
source: https://github.com/txqt/system-reliability-auditor
---

# System Reliability Auditor

## Overview

This skill audits backend systems to detect **reliability issues related to concurrency, lifecycle management, and resource handling**.

It focuses on **provable problems** such as race conditions, cancellation bugs, resource leaks, and invalid lifecycle transitions.

The analysis uses internal state-machine reasoning but only outputs **concrete issues, evidence, and minimal fix plans**.

---

## When to Use This Skill

Use this skill when:

- auditing backend services for reliability issues
- debugging race conditions or async bugs
- reviewing worker or orchestrator lifecycle logic
- inspecting IPC, process managers, or background workers
- checking cancellation handling and shutdown behavior
- verifying safe cleanup of processes, streams, or handles

---

## Step-by-Step Guide

### 1. Analyze the Code

Inspect the provided code and identify modules responsible for:

- lifecycle management
- async tasks
- worker orchestration
- resource allocation
- process management

Infer lifecycle phases internally such as:

- initialization
- running
- async execution
- cancellation
- shutdown

Only reason about states clearly implied by the code.

---

### 2. Identify Transition Triggers

Look for transitions triggered by:

- user actions
- timers
- async task completion
- cancellation tokens
- IPC events
- process exit
- failure paths

Track how modules move between lifecycle phases.

---

### 3. Detect Reliability Issues

Check for common reliability problems:

- race conditions
- ignored cancellation tokens
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

If something cannot be proven from the code, label it:

UNCERTAIN (needs confirmation)

---

### 5. Verify Build and Tests

After suggesting fixes, attempt to verify the project using its native tooling.

Examples:

.NET

```bash
dotnet build
dotnet test
```

Node.js

```bash
npm ci
npm test
```

Python

```bash
pytest
```

Go

```bash
go test ./...
```

Rust

```bash
cargo test
```

If the project has no tests, mark verification as **SKIPPED**.

---

## Examples

### Example: Audit a worker loop

Prompt:

> Audit this worker implementation for lifecycle and concurrency reliability issues.
> 
> [PASTE CODE HERE]

Expected output format:

> ISSUE:
> WorkerLoop.cs – infinite loop ignoring cancellation token
> 
> EVIDENCE:
> while(true) loop without observing CancellationToken
> 
> FIX PLAN:
> Use while(!token.IsCancellationRequested) and propagate token to async calls
> 
> RISK LEVEL:
> HIGH

---

### Example: Audit a process manager

Prompt:

> Review this process manager implementation for resource leaks and lifecycle bugs.
> 
> [PASTE CODE HERE]

Expected output:

> ISSUE:
> ProcessManager.cs – child process not disposed
> 
> EVIDENCE:
> Process.Start used without Exited handler or Dispose()
> 
> FIX PLAN:
> EnableRaisingEvents = true and dispose process during shutdown
> 
> RISK LEVEL:
> CRITICAL

---

## Best Practices

- ✅ Focus only on **provable issues**
- ✅ Prefer **minimal targeted fixes**
- ✅ Use lifecycle reasoning internally
- ✅ Verify fixes using project build tools

- ❌ Do not invent speculative states
- ❌ Do not output large architecture diagrams
- ❌ Do not propose large rewrites without evidence

---

## Limitations

This skill has several limitations:

- It cannot prove runtime race conditions without execution traces.
- It depends on the code provided and may miss issues in unseen modules.
- Build/test verification only works if the repository contains working build scripts.
- Dynamic runtime behaviors (reflection, dynamic loading, external services) may not be fully analyzable.
