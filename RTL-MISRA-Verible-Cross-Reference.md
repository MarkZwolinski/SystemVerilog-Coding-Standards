# RTL-MISRA-Style-Guide Cross-Referenced with Verible Linter

This document cross-references the RTL-MISRA-Style-Guide with Verible lint rules to show which linter checks enforce the project's coding standards.

**Verible Reference:** https://chipsalliance.github.io/verible/lint.html

## Summary

Verible is an open-source SystemVerilog linter maintained by Google and CHIPS Alliance. It provides static analysis checks that align well with MISRA-style RTL discipline. This document maps common Verible rules to the corresponding sections of the RTL coding standard.

---

## 1. Assignment and Procedural Block Rules

### MISRA Rule: Use blocking assignments only in combinational logic

- **Verible Rule:** `blocking-assignment-in-always-ff`
  - Flags blocking assignments in `always_ff` blocks
  - Severity: ERROR
  - Related: Section 3.2 Assignment rules

### MISRA Rule: Use nonblocking assignments only in sequential logic

- **Verible Rule:** `nonblocking-assignment-in-always-comb`
  - Flags nonblocking assignments in `always_comb` blocks
  - Severity: ERROR
  - Related: Section 3.2 Assignment rules

### MISRA Rule: Use `always_comb`, `always_ff`, `always_latch` intentionally

- **Verible Rule:** `always-comb-missing-edge`
  - Detects combinational logic in procedural blocks missing edge-triggered semantics
  - Severity: WARNING
  - Related: Section 3.1 Use of procedural blocks

- **Verible Rule:** `always-latch`
  - Flags inferred latch behavior without explicit use of `always_latch`
  - Severity: WARNING
  - Related: Section 3.3 Incomplete logic is prohibited

---

## 2. Latch and Incomplete Logic Rules

### MISRA Rule: Do not infer latches by omission

- **Verible Rule:** `always-comb-missing-default`
  - Flags missing default clause in case statements within combinational logic
  - Severity: WARNING
  - Related: Section 3.3 Incomplete logic is prohibited

- **Verible Rule:** `always-comb-blocking`
  - Flags when combinational block is incomplete or missing output assignments
  - Severity: WARNING
  - Related: Section 3.3 Incomplete logic is prohibited

- **Verible Rule:** `missing-default-case`
  - Requires default clause in case statements
  - Severity: WARNING
  - Related: Section 3.3 Incomplete logic is prohibited

- **Verible Rule:** `inferred-latch`
  - Detects unintended latch inference from incomplete if/case logic
  - Severity: WARNING (often configurable to ERROR)
  - Related: Section 3.3 Incomplete logic is prohibited

---

## 3. Reset and Clock Discipline Rules

### MISRA Rule: Every sequential element shall have defined reset

- **Verible Rule:** `missing-reset`
  - Flags sequential logic without reset or initialization
  - Severity: WARNING
  - Related: Section 3.4 Reset rules

- **Verible Rule:** `reset-or-missing-reset-signal`
  - Detects inconsistent reset or missing reset across flops
  - Severity: WARNING
  - Related: Section 3.4 Reset rules

### MISRA Rule: Use consistent reset polarity

- **Verible Rule:** `reset-polarity-mismatch`
  - Flags inconsistent reset active-high vs active-low usage
  - Severity: WARNING
  - Related: Section 3.4 Reset rules

---

## 4. Type, Width, and Signedness Rules

### MISRA Rule: Explicit width declaration

- **Verible Rule:** `implicit-parameter-width`
  - Flags parameters without explicit width specification
  - Severity: WARNING
  - Related: Section 4.1 Explicit width

- **Verible Rule:** `bit-width-mismatch`
  - Detects width mismatches in assignments and operations
  - Severity: WARNING
  - Related: Section 4.1 Explicit width

### MISRA Rule: Signedness discipline

- **Verible Rule:** `signed-unsigned-comparison`
  - Flags mixed signed/unsigned comparisons
  - Severity: WARNING
  - Related: Section 4.2 Signedness discipline

---

## 5. State Machine Rules

### MISRA Rule: All states shall be covered in case statements

- **Verible Rule:** `missing-default-case`
  - Requires default clause (relevant for state machines)
  - Severity: WARNING
  - Related: Section 5.0 State Machine Rules

- **Verible Rule:** `enum-name-style`
  - Enforces naming convention for state enums
  - Severity: WARNING
  - Related: Section 5.0 State Machine Rules

---

## 6. Control Structure Rules

### MISRA Rule: Use `unique` or `priority` case when appropriate

- **Verible Rule:** `case-missing-unique`
  - Suggests use of `unique` keyword for mutually exclusive cases
  - Severity: WARNING
  - Related: Section 8.1 Case and if structure

- **Verible Rule:** `case-default-missing`
  - Enforces default clause in case statements
  - Severity: WARNING
  - Related: Section 8.1 Case and if structure

### MISRA Rule: Avoid deeply nested logic

- **Verible Rule:** `else-not-preceded-by-begin`
  - Detects missing braces that can lead to confusing nesting
  - Severity: WARNING
  - Related: Section 8.1 Case and if structure

---

## 7. CDC and Clock Domain Crossing Rules

### MISRA Rule: Synchronize asynchronous signals across clock domains

- **Verible Rule:** `async-signal-crossing`
  - Flags potential CDC violations where async signals cross clock domains
  - Severity: WARNING (tool-specific, may require configuration)
  - Related: Section 6.0 CDC and Reset Integrity

---

## 8. Port and Interface Rules

### MISRA Rule: Module ports shall be clearly defined

- **Verible Rule:** `port-direction-mismatch`
  - Flags mismatched port directions in instantiation vs declaration
  - Severity: ERROR
  - Related: Section 10.0 Interface and Module Discipline

- **Verible Rule:** `undeclared-port`
  - Detects connections to undeclared module ports
  - Severity: ERROR
  - Related: Section 10.0 Interface and Module Discipline

---

## 9. Naming and Style Rules

### MISRA Rule: Use descriptive names

- **Verible Rule:** `module-name-style`
  - Enforces module naming conventions
  - Severity: WARNING
  - Related: Section 10.0 Interface and Module Discipline

- **Verible Rule:** `signal-name-style`
  - Enforces signal naming conventions for consistency
  - Severity: WARNING
  - Related: Section 10.0 Interface and Module Discipline

- **Verible Rule:** `parameter-name-style`
  - Enforces parameter naming conventions
  - Severity: WARNING
  - Related: Section 4.3 Constants

---

## 10. Unacceptable Constructs Rules (Gotchas)

### MISRA Rule: Avoid implicit net declarations

- **Verible Rule:** `undeclared-signal`
  - Flags undeclared or implicit signal usage
  - Severity: ERROR
  - Related: Section 9.3 Implicit nets and undeclared signals

### MISRA Rule: Avoid `casex` / `casez`

- **Verible Rule:** `case-wildcard-mismatch`
  - Flags potentially unsafe use of wildcard case statements
  - Severity: WARNING
  - Related: Section 9.1 `casex` and `casez`

### MISRA Rule: Multiple drivers to the same net

- **Verible Rule:** `multiple-driver`
  - Detects multiple procedural blocks driving the same signal
  - Severity: WARNING
  - Related: Section 3.2 Assignment rules and Section 9.0 Unacceptable Constructs

### MISRA Rule: Avoid combinational feedback loops

- **Verible Rule:** `combinational-loop`
  - Detects combinational feedback that could cause issues
  - Severity: WARNING
  - Related: Section 9.0 Unacceptable Constructs

---

## 11. Memory and Register Rules

### MISRA Rule: Memory ports and behavior shall be explicit

- **Verible Rule:** `memory-port-mismatch`
  - Detects incorrect memory port usage or instantiation
  - Severity: WARNING
  - Related: Section 7.0 Memory and Register Safety

---

## 12. Comparative Mapping Table

| MISRA Section | Topic | Verible Rule(s) | Severity |
|---|---|---|---|
| 3.1 | Procedural blocks | `always-comb-missing-edge`, `always-latch` | WARN |
| 3.2 | Assignment rules | `blocking-assignment-in-always-ff`, `nonblocking-assignment-in-always-comb`, `multiple-driver` | ERROR/WARN |
| 3.3 | Incomplete logic | `always-comb-missing-default`, `missing-default-case`, `inferred-latch` | WARN |
| 3.4 | Reset rules | `missing-reset`, `reset-polarity-mismatch` | WARN |
| 3.5 | Clocking rules | `async-signal-crossing` | WARN |
| 4.1 | Explicit width | `implicit-parameter-width`, `bit-width-mismatch` | WARN |
| 4.2 | Signedness | `signed-unsigned-comparison` | WARN |
| 4.3 | Constants | `parameter-name-style` | WARN |
| 5.0 | State machines | `missing-default-case`, `enum-name-style` | WARN |
| 8.1 | Case/if structure | `case-missing-unique`, `case-default-missing` | WARN |
| 9.1 | `casex`/`casez` | `case-wildcard-mismatch` | WARN |
| 9.3 | Implicit nets | `undeclared-signal` | ERROR |
| 9.0 | Multiple drivers | `multiple-driver` | WARN |
| 10.0 | Module ports | `port-direction-mismatch`, `undeclared-port` | ERROR |

---

## 13. Recommended Verible Configuration for MISRA Compliance

To align Verible with the MISRA-style RTL guide, configure the linter with these rule priorities:

```yaml
# .verible.yml or verible-linter configuration
rules:
  blocking-assignment-in-always-ff:
    severity: error
  nonblocking-assignment-in-always-comb:
    severity: error
  missing-default-case:
    severity: error
  inferred-latch:
    severity: warning
  missing-reset:
    severity: warning
  reset-polarity-mismatch:
    severity: warning
  undeclared-signal:
    severity: error
  multiple-driver:
    severity: warning
  port-direction-mismatch:
    severity: error
  undeclared-port:
    severity: error
  async-signal-crossing:
    severity: warning
  case-wildcard-mismatch:
    severity: warning
  combinational-loop:
    severity: warning
```

---

## 14. Using Verible with the MISRA RTL Guide

### Workflow:

1. **Run Verible static linting** on all RTL modules
2. **Treat ERROR-level violations as MISRA violations** (cannot merge without fix or waiver)
3. **Treat WARNING-level violations as review flags** (must be justified in code review)
4. **Use this cross-reference document** during code review to explain which MISRA rules are checked

### Example Usage:

```bash
# Run Verible linter on RTL
verible-verilog-lint --rules=+all src/*.sv

# Apply configuration from .verible.yml
verible-verilog-lint --rules=+all src/*.sv --waive=$(cat waivers.txt)
```

---

## 15. Gaps: MISRA Rules Not Covered by Verible

Some MISRA rules are **design-level** and cannot be checked by Verible alone:

- **Section 5.0 State Machine Rules:** Unreachable/invalid state trapping (requires formal verification or detailed review)
- **Section 6.0 CDC Integrity:** Metastability risk assessment (requires domain expertise and formal analysis)
- **Section 9.2 `X`/`Z` Hazards:** Unknown state propagation in critical paths (requires simulation and review)
- **Section 9.5 Sensitivity List Correctness:** Simulation/synthesis mismatch in some cases (requires detailed review and simulation)

These areas require **manual code review** in addition to Verible linting.

---

## 16. Summary

Verible provides strong automated checking for:
- ✅ Assignment discipline (blocking vs nonblocking)
- ✅ Latch inference detection
- ✅ Reset consistency
- ✅ Undeclared signals
- ✅ Multiple drivers
- ✅ Port connections
- ✅ Basic type/width issues

Verible has **limited capability** for:
- ⚠️ CDC detection (requires configuration and design knowledge)
- ⚠️ State machine validation
- ⚠️ Complex reset strategies
- ⚠️ Gotcha-level language hazards

**Recommendation:** Use Verible as the first line of defense for MISRA compliance, and combine with manual code review for design-level rules and high-risk patterns.

---

Copyright notice: This cross-reference document maps MISRA-style RTL coding standards to the Verible open-source SystemVerilog linter. It is a project-specific guide and does not replace formal Verible documentation or MISRA standards.
