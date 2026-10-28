---
type: Skill
name: tdd
title: Test-Driven Development (TDD)
description: >-
  Seam-based test-driven development loop. Triggers on "TDD", "red-green-refactor",
  "write tests first", or when building new features test-first.
tags: [testing, tdd, seams, verification]
---

# Test-Driven Development (TDD)

TDD is the vertical red → green cycle. Verify public behavior at architectural seams, never internal implementation details.

---

## What a Good Test Is

* **Specification, Not Mirror:** Tests verify behavior through public interfaces. If internal code is refactored without changing public behavior, tests must not break.
* **Independent Truth:** Expected values must come from an independent source (spec, worked example, hardcoded literal)—never recomputed using the same logic as the implementation.

---

## Seams: Where Tests Live

A **seam** is a public boundary where behavior is observed without reaching inside:
1. **Agree Seams First:** Identify the public interface before writing a test.
2. **Never Test Internals:** Do not test private functions, hidden state, or implementation details.
3. **Mock Only at Boundaries:** Mock network I/O, clock/time, and third-party systems—never internal collaborators.

*Detailed mocking patterns and boundary guidelines: see [Seams & Mocking Guide](./references/seams-and-mocking.md).*

---

## Anti-Patterns to Reject

* **Tautological Tests:** Assertions that pass by construction (`expect(add(a, b)).toBe(a + b)`).
* **Implementation Coupling:** Mocks private functions or queries internal state directly.
* **Horizontal Slicing:** Writing all tests upfront before implementing code. Bulk tests test imagined behavior.
* **Vertical Tracer Bullets (The Right Way):** One seam → One failing test → Minimal implementation to pass → Repeat.

---

## The Loop Rules

1. **Red Before Green:** Write one failing test asserting the expected behavior. Run it and confirm it fails for the expected reason.
2. **Minimal Pass:** Write only enough code to pass the test. Do not anticipate future requirements (adhere to `ponytail`).
3. **Run Suite:** Confirm all tests pass.
4. **Advance Slice:** Pick the next test case or edge case and repeat.
