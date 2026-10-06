# MISRA-like Standard for SystemVerilog Testbenches

This document defines a stricter, MISRA-inspired coding standard for SystemVerilog verification code used in testbenches, UVM components, stimulus generation, and simulation infrastructure.

The purpose is not to constrain all verification creativity, but to keep testbench code deterministic, readable, reviewable, and maintainable under simulation.

## 1. Scope

This standard applies to:
- testbench modules
- classes and objects
- sequences and stimulus generation
- checkers and monitors
- scoreboards and coverage collectors
- UVM components and utilities
- simulation-only infrastructure code

It does not apply to synthesizable RTL logic.

## 2. Verification Objectives

Testbench code shall be:
- deterministic
- clear in intent
- robust under randomized or directed stimulus
- free from race conditions and hidden timing assumptions
- easy to debug and trace
- easy to maintain across regressions and reuse

## 3. Required Discipline for Testbench Code

### 3.1 Separation of concerns

- Keep stimulus generation separate from checking logic.
- Keep scoreboard logic separate from protocol modeling.
- Keep sequence generation separate from verification status reporting.
- Do not mix DUT-driving logic with DUT-checking logic.

### 3.2 Deterministic simulation behavior

- Do not rely on unspecified or hidden ordering between processes.
- Avoid unbounded wait loops without explicit exit conditions.
- Avoid implicit assumptions about scheduler ordering.
- Do not use `#0` or timing tricks unless intentionally required for a specific verification pattern.

### 3.3 Event and timing control discipline

- Event controls shall be explicit.
- Avoid unnecessary fork/join and dynamic process creation.
- Deadlock conditions must be impossible or clearly guarded.
- Use `disable fork` only when the intended process termination is explicit and correct.
- Avoid blocking in testbench code when a nonblocking or mailbox-driven approach is clearer.

## 4. Class and Object Rules

### 4.1 Clear ownership and intent

- Each class shall have a single well-defined responsibility.
- Do not build “god” classes that mix stimulus, checking, and reporting.
- Keep object state transitions explicit.
- Do not overload objects with hidden inherited behavior not required by the test.

### 4.2 State and data integrity

- All class fields shall be initialized or set before use.
- Do not rely on default values that are not explicit.
- Do not use global state for core test behavior unless it is part of the verification architecture.
- Shared data structures must be protected from race conditions or conflicting updates.

### 4.3 Randomization and constraints

- Randomization must be explicit and controlled.
- Constraints shall be readable and consistent with the intent of the stimulus.
- Do not write contradictory or overlapping constraints without review.
- Random generation should be reproducible when required by regression runs.

## 5. Procedural Discipline

### 5.1 Blocking and nonblocking assignments

- In testbench procedural blocks, use blocking assignments for variables and local logic unless a specific nonblocking pattern is required.
- Do not use blocking assignments in clocked procedural logic that models hardware timing unless the intent is explicit and deliberate.
- For RTL-equivalent behaviors, use clearly separated simulation modeling strategies.

### 5.2 Wait and timing behavior

- Avoid indefinite waiting based on undocumented signal conditions.
- Use explicit event triggers, timeouts, or assertion-based completion checks.
- All waits should have fail-safe timeout conditions.

### 5.3 Fork/JOIN patterns

- Use `fork/join_any`, `fork/join_none`, and `fork/join` with clear purpose.
- Do not let spawned processes continue in uncontrolled ways.
- Clean up spawned processes when no longer required.
- Avoid process leaks in long-running or randomized verification.

## 6. Scoreboards, Checkers, and Monitors

- Scoreboards shall compare expected and actual data using explicit rules.
- Designed checkers shall be deterministic and isolate failures clearly.
- Use transaction IDs or tags when ordering and correlation matter.
- Do not allow missed or stale transactions to remain silent.

### 6.1 Reporting requirements

- Failed checks shall produce clear messages with context.
- Required data for debugging shall be logged without excessive redundancy.
- Use named error or check types to support triage.

## 7. Constrained Random and Coverage Rules

- Coverage goals shall be explicit and meaningful.
- Randomization shall not hide invalid or untestable stimulus combinations.
- Avoid coding random constraints that render a test nonproductive or impossible to hit.
- Coverage holes shall be reviewed rather than ignored.

## 8. Assertions and Checks

- Assertions shall be clear, condition-specific, and meaningful.
- Use assertions to check protocol correctness, data consistency, and state transitions.
- Do not use assertions to hide testbench logic or hide failures behind generic messages.
- Assertion failures shall be actionable and easily linked to a signal or transaction.

### 8.1 Checker discipline

- Checkers shall be deterministic.
- Avoid hidden state transitions within checkers unless explicitly documented.
- Keep assertion semantics separated from stimulus generation.

## 9. Timeouts and Completion Conditions

- All long-running simulation tasks must have a timeout or completion condition.
- Do not allow tests to hang indefinitely.
- Use explicit completion and failure conditions for dynamic sequences.

## 10. Messaging and Debugging Rules

- Log messages shall be actionable and include context.
- Use standard formatting for failures, warnings, and checkpoints.
- Keep debug output consistent and structured.
- Do not over-log in normal operation; reserve logging for failures or significant milestones.

## 11. Data Structures and Reuse

- Shared data structures shall be well-typed and documented.
- Avoid hidden assumptions about packet size, ordering, or field meaning.
- Use typedefs and enumerations for protocol clarity.
- Prefer descriptive names over compact but opaque variable names.

## 12. UVM-Specific Guidance

### 12.1 Component roles

- Drivers shall generate stimulus.
- Monitors shall observe and decode transactions.
- Scoreboards shall compare expected and actual results.
- Agents shall assemble coherent interface behavior.
- Sequences shall be deterministic and reviewable.

### 12.2 UVM best practice alignment

- Avoid overloading base-class hooks with hidden side effects.
- Keep sequence items readable and minimal.
- Use explicit transaction fields and constraints.
- Do not rely on implicit ordering of callback execution.

## 13. Unacceptable Testbench Patterns

The following are prohibited or strongly discouraged in verification code:

- hidden process ordering assumptions
- long blocking waits without timeout
- randomization with undocumented constraints
- magic numbers in protocol checks
- stale or unbounded mailbox usage
- hidden global state shared between components
- event race conditions
- unbounded loops without exit conditions
- testbench code that silently ignores failed checks
- uncontrolled `fork` creation
- excessive logging that obscures the actual failure

## 14. Review Checklist

Before accepting a testbench, confirm:
- [ ] clear separation of stimulus, checking, and scoreboard logic
- [ ] no hidden process ordering assumptions
- [ ] all waits have explicit timeouts or completion criteria
- [ ] all checks are actionable and report context
- [ ] constrained randomization is readable and valid
- [ ] no deadlock or infinite wait paths
- [ ] no undeclared or hidden shared state
- [ ] event ordering is unambiguous
- [ ] code is deterministic under regression execution
- [ ] failure reporting is clear and traceable
- [ ] coverage and checks are meaningful
- [ ] sequences and components have clear ownership

## 15. Deviation Policy

Any exception in a testbench must be:
- explicit
- justified by the verification requirement
- reviewed by the responsible verification lead
- clearly documented
- limited in scope

Verification code may be more flexible than synthesizable RTL, but it must remain safe, deterministic, and maintainable.

## 16. Summary

This standard aims to keep SystemVerilog verification code structured, deterministic, and debuggable. The testbench is not just a harness; it is a critical part of reliability and validation. Poor testbench quality can hide bugs and weaken regression confidence.

The central rule is: a testbench should be as explicit, reviewable, and deterministic as the design it validates.

---

# Appendices

## Appendix A: Preferred patterns

- clear class responsibilities
- explicit transaction fields
- direct, readable constraints
- deterministic wait/timeout structure
- purpose-driven `fork` usage
- structured scoreboard checks
- explicit log messages for each failure

## Appendix B: Avoided patterns

- indefinite waits
- implicit event ordering
- hidden shared state
- overloaded classes
- race-prone randomization logic
- silent failures
- excessive debug output
- uncontrolled process creation

## Appendix C: Review signal

This standard shall be applied during:
- testbench review
- regression triage
- simulation debug sessions
- verification signoff reviews

---

Copyright notice: This is a project-specific verification coding guideline intended to support disciplined, reliable SystemVerilog testbench development. It does not replace formal UVM guidance, tool constraints, or project-specific verification policy.

