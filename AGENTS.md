---
type: Directive
title: Agent Task-to-Workflow Routing
description: Master routing directive mapping user task intents to orchestrated workflows.
tags: [agent-directive, routing, workflow, system-rule]
---

# Agent Directive: Task-to-Workflow Routing

Never execute skills in isolation or guess user intent. Map every incoming task to an orchestrated **Workflow**:

```text
User Request (Natural Language) ──► Match Workflow ──► Sequential Skills & Tools
```

## Intent Routing Table

| If the user intent is... | Route to Workflow | Bundled Skill Sequence |
| :--- | :--- | :--- |
| Building, adding, creating, or changing features | `workflows/feature-delivery` | `grilling` ➔ `tdd` + `ponytail` ➔ `operon diff` |
| Fixing errors, crashes, broken tests, or unexpected output | `workflows/bug-fix` | `systematic-debugging` ➔ `tdd` + `ponytail` ➔ `operon clean` + `diff` |
| Inspecting repository health, dead code, bloat, or scaling | `workflows/codebase-audit` | `ponytail/audit` ➔ `operon scan` + `helpers` ➔ Triage Report |

---

## Operating Rules
1. **Adaptive Scoping & Trivial Bypass:** Settle scope immediately. Never trigger full pipelines for trivial or cosmetic edits (e.g. color change, typo). Jump mid-phase when specs are settled; invoke atomic scripts directly for surgical tasks.
2. **No Cold File Edits:** Settle requirements or check diagnostic trace before modifying implementation code.
3. **Minimal Diffs & Monolith Ceiling:** Keep changes surgical via the `ponytail` ladder. Enforce modular extraction when any source file crosses **600 LOC** or contains multiple distinct domains/screens—anti-bloat prevents dependencies and dead code, not clean modular files.
4. **Deterministic Host Execution:** Use the dispatcher `.\tools\operon.ps1 <cmd>` (or `./tools/operon.sh <cmd>`) for checks (`diff`, `clean`, `scan`, `helpers`, `validate`). Zero prompt-token overhead.
5. **Session Closure (No Intra-Session Spam):** Run verification (`operon diff`, `operon clean`) when closing milestones. Only invoke `handoff` on explicit session pause, wrap-up, or agent handover. Never write `.handoff.md` on routine turn completions.
6. **Contextual Nudge & Back-Off:** Suggest a skill at the response tail if risk arises (e.g. wide diff, untested fix). If the user ignores the suggestion, back off immediately until a genuinely new risk occurs. Never nag.
