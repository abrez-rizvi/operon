# Operon Specification

## 1. Tool Definition

**Operon** is an agent-agnostic CLI control plane and shared repository standard that syncs engineering playbooks, codebase knowledge, and deterministic quality gates across software teams using different AI coding tools.

Operon is built on the **Open Knowledge Format (OKF v0.3)**—a typed, frontmatter-driven plain-text schema that turns static documentation and dynamic session state into a fast, queryable local store for AI agents and host machines.

---

## 2. The Motive

Software teams rarely standardize on a single AI coding tool. Engineers switch between Cursor, Claude Code, GitHub Copilot, Codex, Windsurf, and custom CLI agents. This divergence introduces three recurring systemic failure modes:

1. **Behavioral Divergence & Review Fatigue:** Every engineer prompts their assistant differently. Without shared constraints, agents introduce conflicting abstractions, import redundant packages for trivial tasks, and bypass codebase conventions. Senior engineers burn review cycles policing repetitive, unverified AI slop.
2. **Context Firehose & Prompt Tax:** Teams attempt to fix divergence by dumping sprawling markdown rulebooks into system prompts (`.cursorrules`, `CLAUDE.md`). This wastes thousands of tokens on every interaction, increases response latency, and causes instruction drift midway through long sessions.
3. **Tribal Amnesia:** Hard-won architectural boundaries, undocumented API quirks, and PR corrections remain trapped in Slack threads or individual developer heads. The next teammate's agent makes the exact same mistake days later.

**Operon ensures every agent on a team follows the same engineering playbooks, queries the same codebase memory, and passes the same deterministic host verification checks.**

---

## 3. Storage & Architecture Model

Operon is built on the **Open Knowledge Format (OKF v0.3)** and supports two deployment bindings:

```text
Storage Model (Flexible Binding):
├── Central Profile (Default): ~/.operon/projects/<project-hash>/
│   ├── sessions/ (session.md, archived checkpoints)
│   ├── telemetry.jsonl
│   └── config.json
├── Team Repository (Git-Backed): An independent git repo for shared skills, knowledge, and tools
└── In-Repo Mode (Optional): operon/ directly in the product root
```

* **No Git Pollution by Default:** Operon tracks local project state and ephemeral session logs in a local user profile by default.
* **Shared Team Repo:** Teams version and distribute their shared skills, knowledge, and tools through a centralized Git repo synced via `operon init`.
* **Zero Database / Daemon Overhead:** All data lives in human-readable, git-blameable Markdown/YAML files parsed on demand via the host CLI in `<5ms`. No background servers, no SQLite, no vector embeddings.

---

## 4. The OKF v0.3 Core Architecture

Operon structures all agent capability, repository memory, and operational state into five standardized OKF concepts:

```text
operon/
├── skills/       # OKF: Skill (Procedural playbooks & behavioral state machines)
├── knowledge/    # OKF: Knowledge (Repository memory & atomic invariant ledgers)
├── tools/        # Deterministic Executables (Host scripts with exit codes)
├── workflows/    # OKF: Workflow (User-defined execution state machines)
└── docs/         # OKF: Reserved Catalog & Architecture Topology
```

### Concept 1: Skills (`type: Skill`)
* **What it is:** Structured procedural playbooks defining *how* an agent performs engineering tasks (e.g., test-driven development, lean diff verification, root-cause debugging).
* **Metadata Schema:**
  ```yaml
  ---
  type: Skill
  name: test-driven-development
  title: Seam-Based Test-Driven Development
  description: Enforces red-to-green tracer bullets and anti-mock discipline.
  tags: [testing, tdd, reliability]
  tools:
    test: "npm test"
  ---
  ```
* **AST Heading Slicing:** The CLI allows agents to query specific sections (`operon fetch <skill>:<heading>`) rather than ingesting the full document, returning targeted phase instructions on demand.

### Concept 2: Knowledge (`type: Knowledge`)
Operon structures repository knowledge into two distinct epistemic forms:

1. **Consolidated Text (Narrative Specs & ADRs):**
   * High-context architecture documents, threat models, and system design records.
   * Section-addressable markdown (`## Context`, `## Decision`, `## Consequences`).
   * Queried on demand during planning and discovery phases (`operon fetch <doc>:<section>`).
2. **Formatted Entries (Atomic Invariant Ledgers):**
   * Machine-parseable micro-records (YAML/JSON tuples with `id`, `trigger`, `rule`, `fix`).
   * Evaluated by the host CLI for automated mechanical linting (`operon check-invariants`).
   * Injected into prompts as razor-sharp constraint checklists.

```yaml
---
type: Knowledge
name: auth-invariants
title: Authentication & Session Token Invariants
timestamp: 1793285912          # Unix epoch for temporal freshness & decay tracking
author: "team-lead"
paths: ["src/auth/**", "src/middleware/**"]   # Spatial scoping: loads ONLY when matching files are touched
tags: [security, auth, tokens]
tier: invariant                 # [invariant (hard gate) | architecture (guideline)]
---

entries:
  - id: AUTH-01
    name: "Constant-Time Signature Comparison"
    trigger: "verifySignature|compareTokens"
    rule: "Never use standard string equality (==) for token verification."
    reason: "Standard string equality leaks timing information."
    fix: "Use crypto.timingSafeEqual() or platform constant-time comparison."

  - id: AUTH-02
    name: "Revocation Check Gate"
    trigger: "validateSession"
    rule: "Always verify session revocation status against the distributed cache before trust."
    reason: "Tokens remain valid until TTL even if user session is revoked."
    fix: "Query the token revocation blocklist before payload decoding."
```

* **Spatial Scoping (`paths`):** Knowledge binds to physical filesystem glob patterns. The agent receives constraints *only* for the files it is actively modifying, eliminating context firehosing.
* **Temporal Provenance (`timestamp`):** Unix epoch anchors prevent zombie knowledge. Teams can configure policies to audit rules that haven't been reviewed or modified within defined timeframes.

### Concept 3: Deterministic Tools (Polyglot Mechanical Gates)
* **What it is:** Host-executed scripts running on the host CPU in 0 prompt tokens that enforce hard boundaries using exit codes (`0` = pass, `1` = fail).
* **Runtimes:** PowerShell (`.ps1`) and POSIX Bash (`.sh`) for cross-platform execution; Python (`.py`) for AST parsing and OKF validation.
* **Core Gates:**
  * **Blast Radius & Package Manifest Audit:** Enforces expected net additions and blocks unapproved external package additions.
  * **Policy-Driven File Ceilings:** Evaluates file line counts against configurable project policies to prevent monolithic file accretion.
  * **Sanitary Cleanup:** Scans for and removes temporary debugging markers, scratch logging, or unpurged test flags.

### Concept 4: Continuous Sessions (`type: Session`)
Operon resolves the "End-of-Session Paradox" (agents cannot predict which prompt will be the last before rate limits, context compactions, or user exits) by applying the **Write-Ahead Log (WAL)** principle:

* **Continuous Delta Checkpointing:** The agent runs a lightweight command (`operon checkpoint "..."`) upon completing vertical milestones, pivoting context, or every 2–3 code-modifying turns. The host CLI updates `session.md` in 1ms on the host CPU without requiring the agent to read or rewrite the full file.
* **OKF Session Schema:**
  ```yaml
  ---
  type: Session
  id: "b6e08687"
  author: "dev-name"
  timestamp: 1793289745
  branch: "feature/auth-refresh"
  status: in-flight              # [in-flight | paused | closed]
  workflow: custom-pipeline
  phase: 2
  ---
  ```
* **Cross-Session Retrieval:**
  * `operon session --latest`: Loads the most recent chronological session.
  * `operon session --team`: Resolves the latest session by a teammate on the active branch.
  * `operon fetch session:in-flight`: Ingests *only* the active in-flight context.

### Concept 5: Workflows (User-Defined Execution Pipelines)
* **What it is:** Phased execution pipelines linking skills and deterministic tools for common engineering workflows defined by the team.
* **Autonomy Spectrum:**
  1. *Pre-dialed Pipeline:* Agent advances through structured, user-defined phases (e.g., Scope ➔ Implement ➔ Verify).
  2. *Atomic Surgical:* Agent executes individual skills or tools on demand for single-turn tasks (`operon verify`).
  3. *Script-as-a-Plan:* The agent writes an ephemeral composite script to batch multiple queries and tool checks in a single turn, eliminating round-trip latency.

### Concept 6: The Flight Recorder (Telemetry Ledger)
* **What it is:** An append-only machine log stored at `.operon/telemetry.jsonl`.
* **Zero Token Cost:** Logs execution metadata (timestamp, user, intent, skills accessed, tools executed, exit codes, net LOC delta) on the host CPU outside the conversation context.
* **Team Visibility:** Provides leads and reviewers with objective data on agent behavior and tool performance during PR reviews and post-mortems.

### Concept 7: Harness Connectors
* **What it is:** Minimal, non-invasive trampolines (<40 tokens) injected into editor and assistant configurations:
  * Claude Code (`CLAUDE.md`)
  * Cursor (`.cursorrules` / `.cursor/rules/operon.mdc`)
  * Codex / OpenCode (`AGENTS.md`)
  * Gemini / Antigravity (`GEMINI.md`)
  * GitHub Copilot (`.github/copilot-instructions.md`)

---

## 5. CLI Command Reference

### Setup & Conformance

| Command | Scope | Description |
| :--- | :--- | :--- |
| `operon setup` | Global | Detects installed AI harnesses, registers `operon` on PATH, and injects minimal trampolines into global harness configs. |
| `operon init [name]` | Project | Binds the current directory as an Operon project, creating local config and session tracking. |
| `operon validate [path]` | Conformance | Verifies 100% OKF v0.3 schema conformance across all skills, knowledge, and workflows. Exits `1` on invalid frontmatter. |
| `operon tokens [path]` | Audit | Audits token overhead across discovery, active body, and reference tiers to ensure budget compliance. |

### Development Runtime Loop

| Command | Stage | Description |
| :--- | :--- | :--- |
| `operon boot "<task>"` | Entry | Triggered by prompt. Classifies intent, pulls active in-flight session context, checks git status, and emits the Phase 1 directive. |
| `operon fetch <entity>[:section]` | In-Flight | Slices specific skill sections or knowledge headers in targeted token chunks. |
| `operon checkpoint "<status>"` | Continuous | Appends micro-deltas to `session.md` every 2–3 turns or on state transitions (Write-Ahead Logging). |
| `operon session [--latest\|--team]` | Continuity | Resolves chronological session states across teammates or branches without full-file ingestion. |
| `operon check-invariants` | Verification | Lints uncommitted git diffs against formatted atomic knowledge entries. Exits `1` on rule violations. |
| `operon verify` | Exit Gate | Evaluates net LOC delta, enforces project file ceiling policies, and checks for unapproved package additions. |
| `operon clean [-Purge]` | Cleanup | Detects and cleans temporary debugging markers or scratch instrumentation from the codebase. |
| `operon remember "<entry>"` | Knowledge | Formulates and appends a new atomic invariant or quirk to project knowledge. |
| `operon handoff` | Session Wrap | Concludes active session state and marks status as paused or wrapped. |

---

## 6. Execution Rules & Operating Invariants

1. **Adaptive Scoping (Trivial Bypass):** Simple tasks (e.g., cosmetic styling, typo fix, 1-line bug) skip multi-phase workflows. Agents bypass heavy pipelines for surgical changes.
2. **Continuous Delta Checkpointing:** Never rely solely on session-end triggers. Agents drop lightweight checkpoints via `operon checkpoint` upon completing vertical milestones, pivoting context, or every 2–3 code-modifying turns.
3. **Policy-Driven File Modularization:** File size limits and modularity rules are enforced mechanically via project policy configuration rather than subjective prompt advice.
4. **Spatial Knowledge Isolation:** Agents query knowledge scoped to the paths modified in the git diff (`paths`) rather than ingesting entire repository wikis.
5. **Deterministic Exit Gates:** All non-trivial code modifications must pass `operon verify` and `operon clean` before a task is considered complete.

---

## 7. Team Collaboration Scenarios

| Scenario | Without Operon | With Operon |
| :--- | :--- | :--- |
| **Cross-Tool Knowledge Sync** | Developer A discovers an obscure middleware auth token timing leak in Cursor. Developer B working in Claude Code introduces the exact same bug an hour later. | Developer A's agent logs the invariant via `operon remember`. When Developer B's agent touches matching paths, the invariant is sliced and injected automatically. |
| **Enforcing Quality Guardrails** | Teammates rely on manual PR reviews to catch temporary debug statements, unneeded packages, and runaway monolithic files. | `operon verify` runs locally on the host CPU and exits `1` if policy limits or dependency checks fail, forcing the agent to refactor before committing. |
| **Abrupt Session Termination** | An engineer is interrupted, laptop battery dies, or an agent hits a rate limit midway through a complex refactor. All in-flight context is lost. | Continuous delta checkpointing (`operon checkpoint`) has saved state to disk within the last 2 turns. Running `operon boot` resumes immediately. |
| **Teammate Branch Handoff** | An engineer checks out a teammate's PR branch and has to read dozens of commit diffs or ask on Slack to understand what is remaining. | The engineer runs `operon session --team`. The CLI returns a concise summary of completed milestones, in-flight work, and the exact next step. |
| **PR Verification & Audit** | Reviewers have no objective proof of whether an agent tested its changes or guessed solutions speculatively. | The reviewer checks `.operon/telemetry.jsonl` to see the exact deterministic tools executed, test durations, and rule conformance throughout the build. |
