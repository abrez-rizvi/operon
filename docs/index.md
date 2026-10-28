---
okf_version: "0.2"
---

# Operon Index & Catalog

Orchestrated workflows, atomic skills, and deterministic tools for AI coding assistants.

## Workflows (Adaptive Pipelines)

* [**Feature Delivery**](../workflows/feature-delivery/WORKFLOW.md) — `grilling` (Align) ➔ `tdd` + `ponytail` (Build) ➔ `operon diff` (Close)
* [**Bug Fix**](../workflows/bug-fix/WORKFLOW.md) — `systematic-debugging` (Repro) ➔ `tdd` + `ponytail` (Fix) ➔ `operon clean` + `diff` (Close)
* [**Codebase Audit**](../workflows/codebase-audit/WORKFLOW.md) — `ponytail/audit` (Map) ➔ `operon scan` + `helpers` (Scan) ➔ Triage Report (Close)

## Skills (Atomic Capabilities)

* [**Ponytail**](../skills/ponytail/SKILL.md) — "Lazy Senior Dev" anti-bloat ladder (`operon diff`, `operon scan`, `operon helpers`).
* [**Grilling**](../skills/grilling/SKILL.md) — Frontier decision-tree interview rounds to settle ambiguous requirements.
* [**Systematic Debugging**](../skills/systematic-debugging/SKILL.md) — Tiered debugging: Tier 1 fast-path vs Tier 2 6-phase scientific loop (`operon clean`).
* [**Test-Driven Development**](../skills/tdd/SKILL.md) — Seam-based TDD with vertical tracer bullets and anti-mock discipline.
* [**Session Handoff**](../skills/handoff/SKILL.md) — Serializes session state into `.handoff.md` for context restoration.
* [**UI Visual Audit**](../skills/ui-visual-audit/SKILL.md) — Layout bounds, density checks, and real-estate ratio verification.

## Directives & Tooling

* [**Task-to-Workflow Routing**](../AGENTS.md) — Master directive mapping intents to workflows.
* [**CLI Dispatcher**](../tools/operon.ps1) — Single root runner (`tools/operon.ps1` / `tools/operon.sh`) for deterministic host checks.
* [**Architecture Graph**](graph.html) — Interactive layered progression visualizer & flow simulator.
* [**Tools**](../tools/) — `validate-okf.py` (OKF v0.2 conformance tester) and `count-tokens.py` / `.ps1` (token footprint auditor).
