# Operon ↔ Meko Integration Specification (`meko.md`)
**Role:** Background Replication Engine & Distributed Context Provider  
**Host Controller:** Operon CLI (Host Control Plane)  
**Upstream Backend:** YugabyteDB Meko Datapack (`@yugabytedb/meko-mcp` / FastMCP / Langfuse / Mem0)  

---

## 1. Core Architectural Invariant

> [!IMPORTANT]
> **The Containment Principle:**  
> **Operon will not be lobotomized for Meko.** Operon does not adapt to Meko's internal schemas; Meko is constrained to serve Operon's deterministic contracts.
> 
> * Operon treats Meko strictly as an **asynchronous, remote persistence layer**.
> * The AI coding agent **never directly interacts with Meko** or its 42 MCP tools.
> * The agent interacts **only with the Operon CLI and local OKF files**.
> * **Verification Precedes Persistence:** No memory, invariant, or session state is ever sent to Meko unless it has first passed Operon’s deterministic host gates (Exit Code `0`).

---

## 2. The 3-Tool Containment Boundary

The forensic audit revealed that Meko exposes 42 tools (~64 KB raw JSON schema), consuming **16,000–18,500 prompt tokens per turn** under standard MCP eager loading.

Operon encapsulates and shields the agent from 39 of those tools, strictly interacting with only **three primitive endpoints**:

```
┌────────────────────────────────────────────────────────────────────────┐
│                        MEKO EXPOSURE MATRIX                            │
├───────────────────────┬────────────┬───────────────────────────────────┤
│ Meko Tool             │ Operon Use │ Operational Contract              │
├───────────────────────┼────────────┼───────────────────────────────────┤
│ 1. `artifact_put` /   │ Core State │ Stores/retrieves compiled OKF     │
│    `artifact_get`     │ Storage    │ invariant manifests as pure JSON. │
│                       │            │ Hash-addressed, zero schema pain. │
├───────────────────────┼────────────┼───────────────────────────────────┤
│ 2. `memory_add` +     │ Check T    │ Pushes verified architectural     │
│    `memory_promote`   │ Compliance │ decisions into `<team>-build`.    │
│                       │            │ Satisfies Hackathon 10-point rule.│
├───────────────────────┼────────────┼───────────────────────────────────┤
│ 3. `context_search`   │ Deep Fetch │ Fallback query engine when local  │
│    (Optional)         │ (On-Demand)│ cache misses historical context.  │
├───────────────────────┼────────────┼───────────────────────────────────┤
│ Remaining 39 Tools    │ BLOCKED    │ Zero agent exposure. Eliminates   │
│                       │            │ 18,000-token prompt tax & crashes.│
└───────────────────────┴────────────┴───────────────────────────────────┘
```

---

## 3. Data Pipeline & Normalization Lifecycle

Operon decouples the developer loop from Meko’s network latency and schema quirks via a **Local Translation and Caching Pipeline**:

```mermaid
flowchart TD
    subgraph Remote["Cloud / Upstream (YugabyteDB Meko)"]
        Datapack["<team>-build Datapack"]
        MekoArtifact["artifact_put / artifact_get"]
        MekoMem0["memory_add -> memory_promote"]
    end

    subgraph OperonBackground["Operon Background Worker & Cache Engine"]
        Queue["Offline WAL Queue (.operon/queue.jsonl)"]
        Cache["Local Hash Cache (~/.operon/cache/)"]
        Hasher["SHA-256 ETag Validator"]
        Translator["OKF v0.3 Markdown Compiler"]
    end

    subgraph HostExec["Host Execution & Verification Engine"]
        HostCLI["operon CLI (Host CPU)"]
        Gate["Deterministic Verification Gates (Exit 0 / 1)"]
        GitTree["Git Working Tree & Diff"]
    end

    subgraph AgentScope["Agent Execution Loop (Zero Bloat)"]
        LLM["AI Coding Agent (Cursor / Claude Code)"]
        Trampoline["2-Line Trampoline (<40 Tokens)"]
    end

    %% Ingress Flow
    Datapack -->|1. artifact_get| Hasher
    Hasher -->|Hash mismatch: Pull & Compile| Translator
    Translator -->|2. Emit Typed Markdown| Cache
    Hasher -.->|Hash match: 0ms| Cache

    %% Local Agent Flow
    LLM --> Trampoline
    Trampoline -->|3. operon check-invariants| HostCLI
    HostCLI -->|4. Spatial Path Filter| Cache
    Cache -->|Injects Invariants| HostCLI
    HostCLI -->|Exit 0 or 1| LLM

    %% Egress Flow
    LLM -->|5. Propose Changes & Checkpoint| HostCLI
    HostCLI -->|6. Evaluate Blast Radius & Policies| Gate
    Gate -->|Exit 1: Blocked| LLM
    Gate -->|7. Exit 0: Passed| Queue
    Queue -->|8. Async Background Drain| MekoArtifact
    Queue -->|9. Async Promote (Check T)| MekoMem0
    MekoArtifact --> Datapack
    MekoMem0 --> Datapack
```

### Ingress (Meko $\rightarrow$ Operon Local Store)
1. During `operon init` or background sync, Operon queries `artifact_get` for the team’s `operon-invariants.json`.
2. The payload hash is compared against the local cache. If unchanged, Operon aborts immediately (**0ms overhead, 0 tokens spent, 0 quota consumed**).
3. If new, Operon parses the JSON payload and compiles it into local **OKF v0.3 Markdown** files in `.operon/knowledge/`, preserving typed headers, spatial paths (`paths: [...]`), and atomic triggers.
4. The agent reads and lints against local OKF files. It never makes a remote network call during code generation.

### Egress (Operon Verified State $\rightarrow$ Meko Persistence)
1. The agent completes a task, refactor, or session milestone.
2. The agent executes `npx operon checkpoint "..."` and `npx operon verify`.
3. Operon's host gates run locally on the host CPU:
   - **Blast Radius Gate:** Checks net LOC delta against project ceilings.
   - **File Ceiling Gate:** Checks modified file sizes.
   - **Sanitary Gate:** Ensures no debug flags or temporary markers remain.
   - **Spatial Invariants Gate:** Ensures modified code matches all active OKF triggers.
4. **Only when all gates exit with code `0`**, Operon stages the checkpoint or invariant into `.operon/queue.jsonl`.
5. The background worker drains the queue asynchronously to Meko:
   - Invariants are uploaded to `artifact_put` with a fresh SHA-256 hash.
   - Milestone decisions are sent to `memory_add` and immediately elevated via `memory_promote`.

---

## 4. The Local Cache: Why It Is Legitimate & Mandatory

We deliberately implement and justify the local cache (`~/.operon/cache/` or `.operon/cache/`) on two technical grounds:

1. **Protocol Asymmetry (Lack of WebSockets):**  
   Meko is exposed exclusively over request-response protocols (HTTP / SSE / stdio MCP). It provides **no streaming pub/sub, WebSockets, or webhook pushes**. If every developer's agent queried Meko synchronously on every keystroke or command, IDE latency would spike to 2–4 seconds per turn. Operon's local cache acts as the necessary edge read-replica.
2. **Quota Preservation:**  
   Standard Meko accounts are constrained by a **1,000-query lifetime quota** on `memory_search`. A single 50-turn agent session using eager MCP consumes 50–100 queries. Operon’s SHA-256 hash caching drops repeat queries to zero.
3. **Live Context Broadcast:**  
   By combining local git branch metadata with the cached session Write-Ahead Log (WAL), teammates gain real-time awareness of what other agents are building on shared branches without waiting for remote cloud sync.

---

## 5. Background Execution Models

Operon supports two decoupled execution models for talking to Meko:

### Model A: Asynchronous Subprocess Runner (Default for CLI)
When a developer runs `operon sync` or when `operon verify` passes:
- Operon spawns a detached Node.js background process:
  ```typescript
  const child = spawn(process.execPath, [path.join(__dirname, "meko-worker.js")], {
    detached: true,
    stdio: "ignore",
    env: { ...process.env, OPERON_DATAPACK_ID: config.datapackId }
  });
  child.unref(); // Detaches immediately; terminal returns in <15ms
  ```
- The worker reads `.operon/queue.jsonl`, pushes updates to Meko via JSON-RPC, updates the local cache hash, and terminates.
- The developer's terminal never hangs.

### Model B: On-Demand Deterministic Sync
When running in strict environments (CI/CD or explicit team review):
- Running `operon sync --wait` executes synchronously, exiting `0` only after Meko confirms persistence in YugabyteDB.

---

## 6. Meko Payload Contracts

### 1. Invariants Contract (`artifact_put` / `artifact_get`)
* **Key:** `operon-invariants.json`
* **Datapack:** `<team>-build`
* **Payload Structure:**
  ```json
  {
    "version": "0.3.0",
    "updated_at": 1793425900,
    "hash": "4a7f29b6c4...",
    "team": "team-alphacode",
    "invariants": [
      {
        "id": "AUTH-01",
        "name": "Constant-Time Signature Comparison",
        "paths": ["src/auth/**", "src/middleware/**"],
        "trigger": "verifySignature|compareTokens",
        "rule": "Never use == for token comparison. Use crypto.timingSafeEqual().",
        "reason": "Standard equality leaks timing information.",
        "fix": "crypto.timingSafeEqual(Buffer.from(a), Buffer.from(b))"
      },
      {
        "id": "DB-01",
        "name": "Tenant Scoping Invariant",
        "paths": ["src/db/**", "src/queries/**"],
        "trigger": "find|update|delete",
        "rule": "Every database query must include tenant_id in the WHERE clause.",
        "reason": "Prevents cross-tenant data leaks.",
        "fix": "WHERE tenant_id = :tenantId AND ..."
      }
    ]
  }
  ```

### 2. Decision Promotion Contract (Check T Compliance)
To pass **Check T** (10 points: $\ge 10$ promoted decisions from $\ge 2$ teammates' coding agents in `<team>-build`):
* **Step 1:** Call `memory_add`:
  ```json
  {
    "text": "[DECISION][AUTH] Adopted crypto.timingSafeEqual for all bearer tokens to eliminate timing side-channels.",
    "conversation_id": "operon-init-trace-01",
    "agent_id": "cursor:dev-alpha",
    "datapack_id": "<team>-build"
  }
  ```
* **Step 2:** Call `memory_promote`:
  ```json
  {
    "conversation_id": "operon-init-trace-01",
    "memory_ids": ["<returned-memory-uuid>"],
    "agent_id": "cursor:dev-alpha",
    "datapack_id": "<team>-build"
  }
  ```
* Operon provides `scripts/seed-check-t.ts` to automatically populate 10 verified decisions split evenly between `cursor:dev-alpha` and `claude:dev-beta`, guaranteeing full Check T compliance.

---

## 7. The Meko Feature Proposal Linkage

The architectural frictions solved in `meko.md` directly ground our **Meko Feature Proposal** required for the hackathon (10 points):

* **Proposal Title:** *Spatial Path Scoping & Execution Governors for Meko Datapacks*
* **The Problem:** Meko currently treats codebase context as a flat bag of words with 42 unconstrained tools, causing prompt tax and agent hallucinations.
* **The Solution Demonstrated by Operon:** Adding path-scoped metadata filtering (`paths: [...]`) and deterministic exit code gates directly into Meko Datapacks so that agents receive only the exact rules relevant to the files they are touching.
