# System Reliability Auditor

Audit backend systems for **architectural, concurrency, lifecycle, and state-management issues**.

This skill analyzes code using **state-machine reasoning internally** to detect reliability bugs that normal reviews often miss.

Unlike typical AI reviews, it reports **only provable issues** backed by concrete code evidence.

---

# What This Skill Detects

The auditor focuses on **system reliability problems**, including:

### Concurrency
- race conditions
- async lifecycle conflicts
- cancellation bugs
- lock misuse
- thread-safety violations

### Lifecycle Management
- invalid state transitions
- partial initialization
- shutdown sequencing problems
- missing cleanup logic

### Resource Management
- leaked processes
- leaked streams or sockets
- un-disposed handles
- orphan background workers

### State Consistency
- stale cached state
- cross-module assumptions
- inconsistent lifecycle ownership

### Worker / Orchestrator Systems
- zombie processes
- heartbeat failures
- orphan workers
- IPC lifecycle races

---

# Design Philosophy

Most AI code reviews produce vague comments like:

> “This code might have concurrency issues.”

This skill instead enforces **evidence-based auditing**.

Each issue must include:


ISSUE
EVIDENCE
FIX PLAN
RISK LEVEL


If a problem cannot be proven from the code, it is marked:


UNCERTAIN (needs confirmation)


This prevents hallucinated issues.

---

# Example Output


ISSUE:
Module: ProcessManager
Problem: Child processes started but not tracked during shutdown.

EVIDENCE:
Process.Start() invoked without registering an exit handler.
No cleanup path disposes running processes.

FIX PLAN:
Subscribe to Process.Exited and ensure disposal during orchestrator shutdown.

RISK LEVEL:
CRITICAL


---

# Supported Languages

The auditor works with **any backend language**.

Build/test verification uses the project's native tooling.

Examples:

| Language | Build | Test |
|--------|--------|--------|
| .NET | `dotnet build` | `dotnet test` |
| Node.js | `npm install` | `npm test` |
| Python | install deps | `pytest` |
| Go | `go build` | `go test ./...` |
| Rust | `cargo build` | `cargo test` |
| Java | `mvn test` | `gradle test` |

If a project has no build/test commands, verification is skipped.

---

# Ideal Use Cases

This skill works best for **system-level codebases**, such as:

- backend services
- distributed workers
- orchestration engines
- async task runners
- process managers
- message queue consumers
- IPC systems

It is especially useful for reviewing:

- shutdown logic
- cancellation handling
- worker lifecycle management
- process spawning
- async state machines

---

# When To Use

Use this skill when you want to:

- audit a backend system
- review concurrency logic
- debug shutdown issues
- detect race conditions
- find lifecycle bugs
- check worker/orchestrator reliability

---

# How The Analysis Works

The auditor internally performs:

1. Lifecycle inference for each module
2. Transition analysis triggered by:
   - user actions
   - async completions
   - timers
   - failures
   - cancellations
3. Verification of valid state transitions
4. Resource lifecycle analysis
5. Cross-module consistency checks

Only issues **provably implied by code paths** are reported.

---

# What This Skill Does NOT Do

To maintain accuracy, the auditor intentionally avoids:

- generating full architectural documentation
- creating speculative system diagrams
- inventing states not present in code
- reporting unverified issues

---

# Limitations

Like any static analysis tool, this skill cannot detect:

- runtime-only race conditions without visible code paths
- infrastructure-level problems outside the repository
- production configuration errors

Those require runtime observability tools.

---

# Contributing

Suggestions for improving detection patterns are welcome.

Possible future improvements include:

- deadlock detection heuristics
- async starvation detection
- advanced IPC lifecycle validation
- distributed worker analysis

---

# License

MIT
