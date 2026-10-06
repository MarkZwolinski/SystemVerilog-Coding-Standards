# Testbench Linting and Review Guidance

This document explains why Verible is not a primary linting tool for SystemVerilog testbenches and what to use instead.

## 1. Why Verible is not the main tool for testbenches

Verible is primarily designed for synthesizable RTL and hardware design linting. It is very useful for:
- procedural block discipline
- port direction consistency
- signal declaration checks
- default/case coverage in RTL
- latch inference detection
- syntax and structural RTL rules

It is much less effective for the most common testbench risks, which are usually about:
- process ordering
- simulation scheduling
- UVM component behavior
- randomization and constraints
- timing-dependent synchronization
- scoreboard and checker logic
- hidden shared state
- race conditions between monitor/checker/driver processes

A testbench is primarily a simulation artifact, not a synthesizable design. Most failures in a testbench are not syntax problems; they are **behavioral or architectural issues**.

---

## 2. What Verible can still help with

Although Verible is not intended as a full testbench linter, it can still catch a few useful issues in simulation code:

### 2.1 Structural issues

- undeclared signals
- port direction mismatches
- missing default cases in procedural logic used in testbench helpers
- accidental latch-like patterns in simulation-only helper logic
- signal naming and module organization consistency

### 2.2 Basic hygiene

- malformed procedural blocks
- obvious syntax issues
- uninitialized or misdeclared helper logic
- accidental misuse of procedural constructs in helper modules

### 2.3 Limitations

Verible does not fully understand:
- UVM component semantics
- sequence behavior
- randomization constraints
- mailbox usage patterns
- scoreboard logic and queue management
- process race conditions and deadlocks
- simulation-only timing assumptions
- assertions that depend on event ordering

---

## 3. Recommended testbench linting strategy

### Primary strategy: review-driven verification

For testbenches, the most effective approach is usually:

1. **Lint for syntax/structure** using Verible where useful
2. **Review the testbench architecture** against a MISRA-like checklist
3. **Run simulation regressions** with deterministic seeds and coverage checks
4. **Use assertion-based checks** to verify protocol correctness
5. **Review process, timeout, and synchronization logic** explicitly

This is more effective than relying on a single automated linter because the biggest failures in verification code are often semantic and temporal, not syntactic.

---

## 4. Useful static checks for testbenches

Use a layered approach.

### 4.1 Tooling for basic structure

- Verible: useful for HDL structure, modules, and obvious issues
- SystemVerilog parser checks: syntax and declarations
- compiler warnings and simulation warnings: useful for uninitialized values and bad port usage

### 4.2 Custom review checks

These are not always available in standard lint tools, but they should be part of project review:

- no process leaks
- all waits have timeout
- all mailboxes/queues are bounded and drained
- no blocking waits on clocked logic without a timeout
- no silent ignores in checkers
- no hidden shared state among components
- all assertions have clear trigger conditions
- no randomization constraints without justification

### 4.3 Suggested review checklist

For every testbench component, reviewers should ask:

- Does this component have a single role?
- Is the stimulus separated from checking?
- Does every wait have a timeout or completion condition?
- Does this logic depend on ordering, sync, or delta cycles?
- Is there hidden shared state?
- Can this process deadlock?
- Is every failure message actionable?
- Is the test stable under randomization and repeated runs?

---

## 5. What to do instead of relying on Verible

### Recommended tools and practices

- **Verible** for syntax and structural checks where applicable
- **Simulator warnings** for race or uninitialized-state clues
- **Assertion-based checks** for behavioral correctness
- **Regression runs** with repeatable seeds
- **Code review** against the Testbench-MISRA-Style-Guide
- **UVM-specific review** for agent/monitor/driver/scoreboard discipline

### Recommended review categories

The testbench review should explicitly check:

1. Timing discipline
2. Process ownership
3. Timeout behavior
4. Failure reporting
5. Randomization sanity
6. Shared state management
7. Inter-component communication
8. Race condition safety
9. Determinism and repeatability
10. Coverage integrity

---

## 6. Practical guidance for the project

### Do this

- Use Verible for syntax and basic structural errors
- Review all testbench components against the testbench standard
- Require explicit timeout logic for all waits
- Require deterministic random seeds for regression stability
- Require review of shared state, mailboxes, and event usage

### Do not rely on this

- A single lint pass to prove testbench correctness
- Verible alone to detect race conditions
- Implicit assumptions about scheduler ordering
- Hidden timing tricks such as `#0`, polling, or event ordering by accident

---

## 7. Final recommendation

Verible is useful as a **supporting tool** for testbench hygiene, but it should not be treated as the primary compliance mechanism for verification code.

For testbenches, the stronger and more reliable strategy is:

- use Verible for structural checks only
- enforce a disciplined review process
- validate behavior under simulation
- require deterministic, explicit, and reviewable verification logic

This aligns with the purpose of the testbench standard: **catch behavioral weakness before it becomes a regression failure**.

---

Copyright notice: This document provides project guidance for the use of Verible in testbench review. It is not a claim that Verible is a complete testbench linter; rather, it explains how to use it carefully and where design review must take over.
