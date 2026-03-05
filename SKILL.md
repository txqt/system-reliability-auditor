---
name: system-reliability-auditor
description: |
  Audits backend code for architectural, concurrency, lifecycle, and
  state-management reliability issues using evidence-first reasoning.

  Detects and reports provable problems such as:
  - race conditions
  - cancellation-handling bugs
  - resource leaks (processes, streams, handles)
  - invalid lifecycle transitions and partial initialization
  - stale cached-state divergence and cross-module assumptions

  Use when the user asks to:
  - "audit this code"
  - "find race conditions"
  - "review lifecycle or shutdown logic"
  - "debug worker/orchestrator reliability"
  - "check process manager, IPC or async code"
---

# Goal

Identify **architectural, concurrency, lifecycle, and state-management issues**
across languages and runtimes and provide:

- provable evidence derived from the supplied code
- a minimal concrete fix plan for each issue
- a risk level for prioritization

The skill purposely **does not output state machines**; it uses state-machine reasoning internally to drive accurate findings.

---

# Instructions

## 1) Analyze source code
- Infer module lifecycles internally (initialization, running, async ops, shutdown, cancellation, recovery).
- Only reason about states and transitions that are explicit in code or clearly implied by lifecycle logic.
- Do NOT invent speculative states.

## 2) Identify transition triggers
- user actions, timers, async completions, cancellations, process exit, IPC messages, failures.
- Note how these triggers move modules between lifecycle phases.

## 3) Report only verifiable issues
For each issue output exactly this structure (one issue per block):

ISSUE:
- Module / file
- Exact problematic transition or logic
- Why it is dangerous

EVIDENCE:
- The exact code path or condition proving the issue (e.g., line snippet, call sequence, control flow)

FIX PLAN:
- Minimal change required (guard condition, refactor, cancellation propagation, disposal)
- Concrete code example when possible

RISK LEVEL:
LOW / MEDIUM / HIGH / CRITICAL

If something cannot be proven from the code, label it:

UNCERTAIN (needs confirmation)

## 4) Prefer minimal, actionable fixes
- Guard clauses, lifecycle checks, cancellation propagation, small refactors, proper disposal.
- Avoid large architectural rewrites unless evidence shows they are required.

---

# Verification (multi-language)

After proposing fixes, verify that the project still builds and tests (if applicable) using **the project's native tooling**. Use the appropriate adapter (see adapters list below). Run the build/test commands and report if they succeed.

Examples of toolchains (adapter examples):

- .NET / C#
  - Build: `dotnet build`
  - Test: `dotnet test`
- Node.js
  - Install: `npm ci` (or `yarn install`)
  - Test: `npm test` (or `yarn test`)
- Python
  - Setup: `python -m venv .venv && .venv/bin/pip install -r requirements.txt`
  - Test: `pytest`
- Go
  - Build/Test: `go test ./...`
- Rust
  - Build/Test: `cargo test`
- Java (Maven / Gradle)
  - Maven: `mvn -B test`
  - Gradle: `./gradlew test`

**Verification behavior**
- Run the chosen build/test commands in a sandboxed environment.
- If build/test fails, do NOT automatically patch code — report failure and include failure evidence.
- If build/test passes, mark verification succeeded.
- If project has no build/test, mark verification as SKIPPED and continue.

---

# Examples (short)

## Example: worker ignoring cancellation

ISSUE:
- Module: `WorkerLoop.cs`
- Problematic logic: `while (true) { await DoWorkAsync(); }` where shutdown calls `_cts.Cancel()` but token never observed.
- Why dangerous: Worker may never stop; shutdown can hang or leak resources.

EVIDENCE:
- Worker loop uses `while (true)` and `DoWorkAsync()` is invoked without a CancellationToken parameter.
- Shutdown code calls `_cts.Cancel()` but no code path reads `_cts.Token.IsCancellationRequested`.

FIX PLAN:
- Minimal fix: change loop to `while (!_cts.Token.IsCancellationRequested)` and/or pass token into `DoWorkAsync(CancellationToken)` and observe it inside.
- Add a time-limited join during shutdown as a failsafe.

RISK LEVEL:
HIGH

## Example: child process leak

ISSUE:
- Module: `ProcessManager`
- Problematic logic: `var p = Process.Start(info); _workers.Add(p);` without subscribing to `p.Exited` or disposing `p`.
- Why dangerous: Process objects (and OS child processes) can survive orchestrator exit, creating zombies.

EVIDENCE:
- Process.Start used; no `EnableRaisingEvents`, no `Exited` handler, no disposal on shutdown.

FIX PLAN:
- Set `p.EnableRaisingEvents = true; p.Exited += OnChildExit;`
- Ensure `Process.Dispose()` in cleanup path.
- Add health check to reconcile running OS processes vs internal `_workers` list.

RISK LEVEL:
CRITICAL

---

# Adapter / Plugin plan

To keep core analysis language-agnostic we separate verification into small adapters. Each adapter only needs a tiny manifest:

`adapters/dotnet-adapter.md`
- commands: `dotnet build`, `dotnet test`
- env: .NET SDK required

`adapters/node-adapter.md`
- commands: `npm ci`, `npm test`
- env: Node.js + npm

`adapters/python-adapter.md`
- commands: `python -m venv .venv && .venv/bin/pip install -r requirements.txt`, `pytest`
- env: python3, pip

Adapters are lightweight text files mapping a repo to its verification commands. New adapters can be added easily.

---

# Output expectations

- For each code sample analyzed, produce zero or more ISSUE blocks.
- If no provable issues found, output:

`No reliability issues detected from available code paths.`

- Always include `<!-- Generated by System Reliability Auditor v1.0 -->` at the end of outputs.

---

<!-- Generated by System Reliability Auditor v1.0 -->
