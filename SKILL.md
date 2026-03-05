---
name: system-reliability-auditor
description: |
  Audits backend systems for architectural, concurrency, lifecycle, and
  state-management reliability issues using evidence-first analysis.
risk: safe
source: https://github.com/txqt/system-reliability-auditor
---

# Goal

Identify **architectural, concurrency, lifecycle, and state-management issues**
across languages and runtimes and provide:

- provable evidence derived from the supplied code
- a minimal concrete fix plan for each issue
- a risk level for prioritization

The skill purposely **does not output state machines**; it uses state-machine
reasoning internally to drive accurate findings.

---

# Instructions

## 1) Analyze source code

- Infer module lifecycles internally (initialization, running, async operations,
  shutdown, cancellation, recovery).
- Only reason about states and transitions that are explicit in code or clearly
  implied by lifecycle logic.
- Do **not invent speculative states**.

## 2) Identify transition triggers

Look for transitions triggered by:

- user actions
- timers
- async completions
- cancellations
- process exits
- IPC messages
- failures

Observe how these triggers move modules between lifecycle phases.

## 3) Report only verifiable issues

For each issue output exactly this structure:

ISSUE:

- Module / file
- Exact problematic transition or logic
- Why it is dangerous

EVIDENCE:

- The exact code path or condition proving the issue
- Example: line snippet, call sequence, or control-flow reasoning

FIX PLAN:

- Minimal change required
- Guard condition, refactor, cancellation propagation, or proper disposal
- Provide a concrete code example when possible

RISK LEVEL:

LOW / MEDIUM / HIGH / CRITICAL

If something cannot be proven from the code, label it:

UNCERTAIN (needs confirmation)

## 4) Prefer minimal, actionable fixes

Favor:

- guard clauses
- lifecycle checks
- cancellation propagation
- correct disposal
- small refactors

Avoid proposing large architectural rewrites unless clearly required.

---

# Verification (Multi-language)

After proposing fixes, verify that the project still builds and tests using the
project's native tooling.

Run the appropriate build and test commands depending on the ecosystem.

Examples:

### .NET / C#

Build:


dotnet build


Test:


dotnet test


### Node.js

Install:


npm ci


Test:


npm test


### Python

Environment setup:


python -m venv .venv
.venv/bin/pip install -r requirements.txt


Test:


pytest


### Go


go test ./...


### Rust


cargo test


### Java

Maven:


mvn -B test


Gradle:


./gradlew test


### Verification behavior

- Run build and test commands in a sandboxed environment.
- If build or tests fail, **report the failure with evidence**.
- Do **not automatically rewrite large parts of the code**.
- If build and tests succeed, mark verification as successful.
- If the project has no build or tests, mark verification as **SKIPPED**.

---

# Example

## Worker ignoring cancellation

ISSUE:

- Module: `WorkerLoop.cs`
- Problematic logic: `while (true) { await DoWorkAsync(); }`
- Shutdown calls `_cts.Cancel()` but the token is never observed.

Why dangerous:

The worker may never stop, causing shutdown hangs or leaked resources.

EVIDENCE:

- Worker loop uses `while (true)`
- `DoWorkAsync()` does not accept a `CancellationToken`
- Shutdown logic triggers `_cts.Cancel()` but no code observes it

FIX PLAN:

Change loop condition and propagate the cancellation token.

Example:


while (!_cts.Token.IsCancellationRequested)
{
await DoWorkAsync(_cts.Token);
}


Also add a time-limited shutdown join as a failsafe.

RISK LEVEL:

HIGH

---

## Child process leak

ISSUE:

- Module: `ProcessManager`
- Problematic logic: child processes started but not tracked or disposed

Why dangerous:

Processes can outlive the orchestrator and become zombie processes.

EVIDENCE:


var p = Process.Start(info);
_workers.Add(p);


No `Exited` handler or disposal logic exists.

FIX PLAN:

Enable exit events and ensure cleanup.

Example:


p.EnableRaisingEvents = true;
p.Exited += OnChildExit;


Ensure `Process.Dispose()` runs during shutdown and reconcile OS processes
against the internal worker list.

RISK LEVEL:

CRITICAL
