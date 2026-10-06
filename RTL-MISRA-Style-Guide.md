# Strict MISRA-like Standard for Synthesisable SystemVerilog RTL

This document defines a stricter, MISRA-inspired coding standard for synthesisable SystemVerilog RTL intended for safety-critical, production, or verification-sensitive digital design.

This is not a substitute for a formal HDL safety standard. It is a design governance reference for keeping RTL deterministic, synthesizable, reviewable, and robust.

## 1. Scope

This standard applies to all synthesizable RTL modules, including:
- datapaths
- controllers
- FSMs
- register blocks
- interfaces and glue logic
- safety-critical or high-integrity logic

It does not apply to testbench-only constructs or verification infrastructure.

## 2. Design Objectives

The RTL shall be:
- deterministic
- synthesizable
- clear and reviewable
- free from unintended latch or race behavior
- easy to lint, simulate, and formally verify
- safe under reset, clock, and power-up conditions

## 3. Required Language Discipline

### 3.1 Use of procedural blocks

- Use `always_comb` for combinational logic.
- Use `always_ff` for sequential logic.
- Use `always_latch` only when a latch is explicitly required and approved.
- Do not use procedural blocks that mix combinational and sequential intent.
- Do not write multiple procedural blocks driving the same logic unless the design is explicit and reviewed.

Required:
- one clear purpose per procedural block
- complete assignment coverage in combinational logic
- consistent use of reset and clock signaling

### 3.2 Assignment rules

- Use blocking assignments (`=`) only in combinational logic.
- Use nonblocking assignments (`<=`) only in sequential logic.
- Never use blocking assignments in `always_ff` blocks.
- Never use nonblocking assignments in `always_comb` blocks.
- Never assign the same variable from more than one procedural block without explicit justification.

### 3.3 Incomplete logic is prohibited

- Combinational logic shall assign all outputs for all legal input combinations.
- All `case` statements shall include a `default` unless exhaustive coverage is otherwise guaranteed and documented.
- `if` statements that drive outputs must cover all legal branches.
- Do not infer latches by omission.

### 3.4 Reset rules

- Every sequential element shall have a defined reset or initialization strategy.
- Reset behavior shall be explicit and consistent across the design.
- Synchronous resets shall be preferred unless asynchronous reset is necessary and justified.
- Use a single reset polarity convention consistently across the design.
- Do not infer logic from uninitialized or implicitly reset state.

### 3.5 Clocking rules

- All clock relations shall be explicit.
- Multi-clock-domain logic shall be clearly identified and isolated.
- Crossing clock domains shall use defined synchronizer or handshake strategies.
- Do not allow asynchronous signals to be sampled without synchronization.
- Do not use low-level timing constructs in RTL that introduce simulation-only behavior.

## 4. Types, Width, and Signedness

### 4.1 Explicit width

- All signal widths shall be explicit unless a well-defined default is required by the project.
- Avoid implicit width extension or truncation without a documented rationale.
- Use `logic`, `logic signed`, `logic unsigned`, or explicit packed arrays with clear intent.

### 4.2 Signedness discipline

- Signedness shall be declared deliberately.
- Do not compare signed and unsigned values without review.
- Do not rely on implicit signedness conversion rules.
- Use explicit casting where conversion is required.

### 4.3 Constants

- Use named constants or parameters instead of magic numbers.
- Numeric literals must be consistent with expected width and semantics.
- Use `localparam` or parameter declarations for design constants.

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

## 6. CDC and Reset Integrity

- Cross-clock-domain logic shall be documented.
- Synchronizer chains shall be explicit and not optimized away by designer assumptions.
- Asynchronous inputs shall be synchronized before use in a different clock domain.
- CDC assumptions shall be reviewed for metastability risk.

## 7. Memory and Register Safety

- Memory ports shall be explicit and not inferred ambiguously.
- The write/read behavior of each memory shall be clear.
- Avoid multiple write sources to the same memory or register.
- Ensure that memory enables and address validity are checked.
- Verified reset and initialization behavior shall be required for memories and control blocks.

## 8. Control Structure Rules

### 8.1 Case and if statements

- Use `unique case` when mutual exclusivity is intended.
- Use `priority case` only when priority-based behavior is intentional and required.
- Do not rely on implicit priority in unstructured branching.
- Avoid nested conditionals that obscure behavior.

### 8.2 Loops

- Loops in synthesizable RTL shall be bounded and explicit.
- Avoid unbounded or tool-dependent loops.
- Avoid loops that produce unintended hardware or hidden timing behavior.

## 9. Unacceptable Constructs

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
- unreviewed use of `X` propagation in design-critical logic

## 10. Interface and Module Discipline

- Module ports shall be clearly defined and directionally consistent.
- Interfaces shall be used when they improve clarity and reduce error.
- Do not hide important protocol state in global variables.
- Parameter and port names shall be descriptive.
- All design assumptions shall be visible in the interface contract.

## 11. Coding Review Checklist

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
- [ ] code is reviewable and maintainable

## 12. Deviation Policy

Any exception to this standard must be:
- documented
- justified
- reviewed by the design authority
- traceable
- limited to a narrow and explicit scope

Do not conceal exceptions in otherwise clean RTL.

## 13. Summary

This standard enforces disciplined, synthesis-safe HDL practice. The intent is to eliminate ambiguous or dangerous logic, ensure deterministic hardware behavior, and make RTL understandable, lintable, and formally analyzable.

The central rule is simple: if the logic is not obvious, explicit, and testable, it is not acceptable for production synthesizable RTL.

---

# Appendices

## Appendix A: Required patterns

Use these patterns by default:
- `always_comb` for purely combinational logic
- `always_ff` for all sequential register behavior
- clear reset conditions in all flops
- `enum` for state encoding
- `unique case` for mutually exclusive decision logic

## Appendix B: Forbidden patterns

Avoid these patterns in production RTL:
- use of blocking assignments in `always_ff`
- omission of default branch in `case`
- undocumented CDC crossings
- inferred latches
- ambiguous `if` chains without full coverage
- multi-driver logic
- implicit width assumptions

## Appendix C: Review signal

This standard shall be applied during:
- design review
- RTL review
- lint and formal review
- integration and signoff checks

---

Copyright notice: This document is a project-specific guideline document and is intended to support safe RTL design practices. It is not a replacement for formal standards or vendor-specific synthesis guidance.

