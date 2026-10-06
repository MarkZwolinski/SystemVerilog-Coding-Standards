# Strict MISRA-like Standard for Synthesisable SystemVerilog RTL

This document combines the MISRA-style RTL governance rules with the most important source-review gotchas from Verilog and SystemVerilog design practice. It is intended as a practical review standard for synthesisable RTL in production, safety-critical, or verification-sensitive digital design.

This is not a substitute for a formal HDL safety standard. It is a design governance and review reference for keeping RTL deterministic, synthesizable, reviewable, and robust.

## 1. Scope

This standard applies to all synthesable RTL modules, including:
- datapaths
- controllers
- FSMs
- register blocks
- interfaces and glue logic
- safety-critical or high-integrity logic

It does not apply to verification-only constructs or testbench infrastructure.

## 2. Design Objectives

The RTL shall be:
- deterministic
- synthesizable
- clear and reviewable
- free from unintended latch, race, or contention behavior
- easy to lint, simulate, and formally verify
- safe under reset, clock, and power-up conditions

## 3. Required Language Discipline

### 3.1 Procedural blocks

- Use `always_comb` for combinational logic.
- Use `always_ff` for sequential logic.
- Use `always_latch` only when a latch is explicitly required and approved.
- Do not mix combinational and sequential intent in a single procedural block.
- Do not drive the same signal from more than one procedural block unless the design is explicit, intentional, and reviewed.
- Do not rely on inferred behavior from incomplete sensitivity controls.

Required:
- one clear purpose per procedural block
- complete assignment coverage in combinational logic
- consistent use of clock and reset signaling

### 3.2 Assignment rules

- Use blocking assignments (`=`) only in combinational logic.
- Use nonblocking assignments (`<=`) only in sequential logic.
- Never use blocking assignments in `always_ff` blocks.
- Never use nonblocking assignments in `always_comb` blocks.
- Do not assign the same variable from more than one procedural block without explicit justification.

### 3.3 Incomplete logic is prohibited

- Combinational logic shall assign all outputs for all legal input combinations.
- All `case` statements shall include a `default` unless exhaustive coverage is otherwise guaranteed and documented.
- `if` statements that drive outputs must cover all legal branches.
- Do not infer latches by omission.
- Do not rely on implicit defaults or uninitialized variables for logic output.

### 3.4 Reset rules

- Every sequential element shall have a defined reset or initialization strategy.
- Reset behavior shall be explicit and consistent across the design.
- Synchronous resets shall be preferred unless asynchronous reset is necessary and justified.
- Use a single reset polarity convention consistently across the design.
- Do not infer logic from uninitialized or implicitly reset state.
- Reset logic must be reviewable and race-free.

### 3.5 Clocking rules

- All clock relations shall be explicit.
- Multi-clock-domain logic shall be clearly identified and isolated.
- Crossing clock domains shall use defined synchronizer or handshake strategies.
- Do not allow asynchronous signals to be sampled without synchronization.
- Do not use low-level timing constructs in RTL that introduce simulation-only behavior.
- Do not infer CDC by coincidence; use documented synchronizer patterns.

## 4. Types, Width, and Signedness

### 4.1 Explicit width

- All signal widths shall be explicit unless a well-defined default is required by the project.
- Avoid implicit width extension or truncation without a documented rationale.
- Use `logic`, `logic signed`, `logic unsigned`, or explicit packed arrays with clear intent.
- Do not depend on tool default sizing when the result is safety- or protocol-critical.

### 4.2 Signedness discipline

- Signedness shall be declared deliberately.
- Do not compare signed and unsigned values without review.
- Do not rely on implicit signedness conversion rules.
- Use explicit casting where conversion is required.
- Ensure arithmetic uses the intended signedness and range.

### 4.3 Constants and literal discipline

- Use named constants or parameters instead of magic numbers.
- Numeric literals must match expected width and semantics.
- Use `localparam` or parameter declarations for design constants.
- Do not rely on implicit sizing of constants in critical arithmetic paths.

## 5. State Machine Rules

- State machines shall be encoded clearly using `enum` where practical.
- All states shall be covered in `case` statements.
- Unreachable or invalid states shall be trapped explicitly.
- Default transitions shall be defined for illegal states.
- State machine transitions shall be reviewable and deterministic.

### 5.1 Required behavior

- No next-state logic shall depend on ambiguous or hidden ordering.
- A state machine shall not have multiple independent assignments to the same state register.
- Sequential state updates shall be straightforward and traceable.
- Priority and ordering assumptions shall be explicit, not implicit.

## 6. CDC and Reset Integrity

- Cross-clock-domain logic shall be documented.
- Synchronizer chains shall be explicit and not optimized away by designer assumptions.
- Asynchronous inputs shall be synchronized before use in a different clock domain.
- CDC assumptions shall be reviewed for metastability risk.
- Signals crossing domains must be declared and treated as asynchronous unless proven otherwise.

## 7. Memory and Register Safety

- Memory ports shall be explicit and unambiguous.
- The write/read behavior of each memory shall be clear.
- Avoid multiple write sources to the same memory or register.
- Ensure memory enables and address validity are checked.
- Verified reset and initialization behavior shall be required for memories and control blocks.
- Do not infer unintended memory structures from procedural code.

## 8. Control Structure Rules

### 8.1 Case and if structure

- Use `unique case` when mutual exclusivity is intended.
- Use `priority case` only when priority-based behavior is intentional and required.
- Do not rely on implicit priority in unstructured branching.
- Avoid nested conditionals that obscure behavior.
- Avoid `casex` / `casez` unless there is a specific, reviewed, and required use case.

### 8.2 Loops

- Loops in synthesizable RTL shall be bounded and explicit.
- Avoid unbounded or tool-dependent loops.
- Avoid loops that produce unintended hardware or hidden timing behavior.
- Do not use loops as a substitute for clear combinational or sequential logic structure.

## 9. Verilog/SystemVerilog Gotchas to Treat as Hard Rules

These are the most common source-review hazards in RTL and must be treated as red flags during review.

### 9.1 `casex` and `casez`

- Avoid `casex` and `casez` in synthesisable logic unless there is a deliberate, documented reason.
- They can mask bits and hide intended logic states.
- They can create logic that appears valid but silently matches unintended values.

### 9.2 `X` and `Z` hazards

- Do not allow `X` and `Z` propagation to drive normal design logic unless explicitly intended.
- Use intentional reset and initialization to avoid unknown states.
- Review all cases where unknown values may propagate through logic, comparisons, or state updates.
- Unknown states in FSMs must be handled explicitly.

### 9.3 Implicit nets and undeclared signals

- Implicit signal declarations are prohibited.
- Every signal must be declared with an explicit intent and scope.
- Do not rely on implicit nettype behavior or undeclared signal creation.

### 9.4 Tri-state and bus contention

- Tri-state logic shall be explicit and reviewable.
- Do not create multiple drivers to the same net without intentional resolved logic.
- Contentions must be reviewed for hardware safety and synthesis correctness.

### 9.5 Sensitivity list correctness

- Ensure that every signal required for a combinational process is included in the sensitivity list.
- Missing signals in procedural sensitivity lists are a classic synthesis mismatch.
- Do not rely on accidental updates from unlisted signals.

### 9.6 Procedural delays and event control misuse

- Do not use `#0`, waiting constructs, or event ordering tricks in synthesizable RTL unless it is part of a well-justified low-level protocol or testbench pattern.
- Timing-sensitive procedural behavior should not be used as a design primitive in production RTL.

### 9.7 Event ordering and simulation scheduling hazards

- Do not rely on undefined scheduling order between procedural blocks.
- Avoid coding logic that depends on simulation delta-cycle behavior.
- Don’t assume that signal updates occur in a particular order across unrelated processes.

### 9.8 Uninitialized signals and hidden state

- Registers and state variables shall be reset or initialized before being used.
- Do not rely on startup behavior or default unknown values.
- Hidden state in logic blocks is not acceptable for production RTL.

## 10. Unacceptable Constructs

The following are prohibited in synthesizable RTL unless specifically approved for a narrow and justified use:

- implicit signal declarations
- hidden latch inference
- multiple drivers to the same net or register
- combinational feedback loops
- `force` / `release` in synthesis logic
- assignment to the same signal from multiple procedural blocks
- procedural delays or timing-dependent constructs in synthesizable logic
- race-prone event control patterns
- undefined or non-portable synthesis assumptions
- unreviewed use of `X` or `Z` logic in critical paths
- `casex` / `casez` misuse
- tri-state contention without explicit design intent

## 11. Interface and Module Discipline

- Module ports shall be clearly defined and directionally consistent.
- Interfaces shall be used when they improve clarity and reduce error.
- Do not hide important protocol state in global variables.
- Parameter and port names shall be descriptive.
- All design assumptions shall be visible in the interface contract.
- Keep module boundaries explicit and avoid hidden cross-module dependencies.

## 12. Coding Review Checklist

Before accepting RTL, confirm:
- [ ] proper use of `always_comb` / `always_ff`
- [ ] no latches inferred
- [ ] no unintended multiple drivers
- [ ] reset strategy defined and consistent
- [ ] all combinational outputs assigned
- [ ] `default` coverage complete
- [ ] state machine covered and reachable states handled
- [ ] no signed/unsigned mismatch without review
- [ ] widths are explicit
- [ ] CDC boundaries are documented
- [ ] memory behavior is unambiguous
- [ ] no `casex` / `casez` misuse
- [ ] no implicit net declarations
- [ ] no uninitialized or unknown-state logic without review
- [ ] no reliance on event ordering or delta-cycle behavior
- [ ] code is reviewable and maintainable

## 13. Deviation Policy

Any exception to this standard must be:
- documented
- justified
- reviewed by the design authority
- traceable
- limited to a narrow and explicit scope

Do not conceal exceptions in otherwise clean RTL.

## 14. Summary

This standard combines MISRA-like design discipline with the highest-value Verilog/SystemVerilog gotchas. The intent is to eliminate ambiguous or dangerous logic, ensure deterministic hardware behavior, and make RTL understandable, lintable, and formally analyzable.

The central rule is simple: if the logic is not obvious, explicit, and testable, it is not acceptable for production synthesizable RTL.

---

# Appendices

## Appendix A: Preferred patterns

Use these patterns by default:
- `always_comb` for purely combinational logic
- `always_ff` for all sequential register behavior
- clear reset conditions in all flops
- `enum` for state encoding
- `unique case` for mutually exclusive decision logic
- explicit declarations for all signals
- default-handling for case and state logic

## Appendix B: Forbidden patterns

Avoid these patterns in production RTL:
- use of blocking assignments in `always_ff`
- omission of default branch in `case`
- undocumented CDC crossings
- inferred latches
- ambiguous `if` chains without full coverage
- multi-driver logic
- implicit width assumptions
- `casex` / `casez` without explicit review
- implicit net declarations
- unknown-state propagation in safety-sensitive logic
- timing/event-order dependencies in synthesizable RTL

## Appendix C: Review signal

This standard shall be applied during:
- design review
- RTL review
- lint and formal review
- integration and signoff checks

---

Copyright notice: This document combines project-specific safety-oriented RTL guidance with common Verilog/SystemVerilog hazard review points. It is intended to support safe RTL design practices and is not a replacement for formal standards or vendor-specific synthesis guidance.
