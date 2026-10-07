# Variable Initialization Rules: RTL vs Testbench

This document provides a side-by-side comparison of initialization-before-use rules across both the RTL-MISRA-Style-Guide and Testbench-MISRA-Style-Guide.

## Core Principle

**Variables and state must be initialized before they are read or used.**

This rule applies to both RTL and testbench code and is enforced through:
- Explicit assignment at declaration or before first use
- Reset or initialization in hardware state
- Constructor or setup methods in classes
- No reliance on implicit or default values

---

## RTL Initialization Rules

### Section 3.4: Reset Rules

- Every sequential element shall have a defined reset or initialization strategy.
- Reset behavior shall be explicit and consistent across the design.
- Synchronous resets shall be preferred unless asynchronous reset is necessary and justified.
- Use a single reset polarity convention consistently across the design.
- Do not infer logic from uninitialized or implicitly reset state.

### Section 3.3: Incomplete Logic is Prohibited

- Combinational logic shall assign all outputs for all legal input combinations.
- All `case` statements shall include a `default` unless exhaustive coverage is otherwise guaranteed and documented.
- `if` statements that drive outputs must cover all legal branches.
- Do not infer latches by omission.

### Intent

RTL initialization is about **hardware state safety**:
- Every flip-flop or latch has explicit reset behavior
- State machines cover all reachable and unreachable states
- Combinational logic does not hide uninitialized values
- Memory and register contents are known at power-up and after reset

---

## Testbench Initialization Rules

### Section 4.2: State and Data Integrity

- All class fields shall be initialized or set before use.
- Do not rely on default values that are not explicit.
- Do not use global state for core test behavior unless it is part of the verification architecture.
- Shared data structures must be protected from race conditions or conflicting updates.

### Section 13.3: `X` and `Z` Propagation

- Be careful with unknown values in checkers and sequences.
- Unknowns can hide the real failure mode and make coverage or assertions misleading.
- If a signal is expected to be valid, assert that it is valid before using it in logic or comparison.

### Intent

Testbench initialization is about **deterministic verification behavior**:
- Class members and local variables have explicit initialization
- Unknown (`X`) values do not propagate silently in checks
- Shared test state is visible and controllable
- Every sequence and check knows the initial state

---

## Side-by-Side Comparison

| Aspect | RTL | Testbench |
|--------|-----|-----------|
| **What gets initialized** | Flip-flops, registers, latches, state machines | Class fields, local variables, arrays, structs |
| **When** | At power-up, after reset, or at declaration | At object construction or before first use |
| **Mechanism** | Reset signal, `initial` block, or parameter default | Constructor, `new()`, or explicit assignment |
| **Unknown values** | Prohibited in logic; X-values are design failures | Must be handled explicitly in checks |
| **Scope** | Hardware behavior | Simulation behavior |
| **Enforcement** | Lint, formal verification, simulation | Code review, simulation, assertions |

---

## Examples

### RTL: Flip-Flop Initialization

Good:
```systemverilog
always_ff @(posedge clk or negedge rst_n) begin
  if (!rst_n) begin
    count <= 8'd0;  // explicit reset value
  end else begin
    count <= count + 8'd1;
  end
end
```

Bad:
```systemverilog
always_ff @(posedge clk) begin
  count <= count + 8'd1;  // count is uninitialized at startup
end
```

### RTL: Combinational Logic Coverage

Good:
```systemverilog
always_comb begin
  result = 8'h00;  // default assignment
  case (selector)
    2'b00: result = data_a;
    2'b01: result = data_b;
    2'b10: result = data_c;
    2'b11: result = data_d;
  endcase
end
```

Bad:
```systemverilog
always_comb begin
  case (selector)  // missing default; result is uninitialized if no match
    2'b00: result = data_a;
    2'b01: result = data_b;
  endcase
end
```

### Testbench: Class Member Initialization

Good:
```systemverilog
class packet;
  rand logic [7:0] payload;
  logic [7:0] count;
  
  function new();
    payload = 8'h00;
    count = 8'h00;
  endfunction
endclass

module tb;
  packet p;
  initial begin
    p = new();  // fields initialized in constructor
    p.payload = 8'hFF;
  end
endmodule
```

Bad:
```systemverilog
class packet;
  logic [7:0] payload;  // no constructor
endclass

module tb;
  packet p;
  initial begin
    if (p.payload == 8'h00) begin  // payload is uninitialized (X)
      // ...
    end
  end
endmodule
```

### Testbench: Variable Initialization Before Use

Good:
```systemverilog
initial begin
  logic [7:0] counter = 8'd0;
  
  counter = counter + 8'd1;
  $display("counter = %0d", counter);
end
```

Bad:
```systemverilog
initial begin
  logic [7:0] counter;
  
  counter = counter + 8'd1;  // counter is uninitialized (X)
  $display("counter = %0d", counter);
end
```

### Testbench: Unknown Value Handling

Good:
```systemverilog
always @(posedge clk) begin
  if (data_valid) begin
    assert (data !== 8'hXX) else $error("data contains unknowns");
    expected_value = data;
  end
end
```

Bad:
```systemverilog
always @(posedge clk) begin
  if (data == expected_value) begin
    // passes silently if data is X
  end
end
```

---

## Review Checklist: Initialization

Before accepting code, confirm:

### RTL
- [ ] Every flip-flop has explicit reset behavior
- [ ] Every latch has explicit initialization or is explicitly approved
- [ ] State machine covers all reachable states
- [ ] `case` statements include `default` (or exhaustiveness is documented)
- [ ] Combinational logic assigns all outputs for all inputs
- [ ] No latch inference by omission
- [ ] Reset polarity is consistent
- [ ] Memory and register initialization is documented

### Testbench
- [ ] All class members are initialized in constructor or before use
- [ ] Local variables are initialized before first read
- [ ] Arrays are initialized (all elements or explicitly looped)
- [ ] No reliance on default X/Z values
- [ ] Unknowns in critical checks are handled explicitly
- [ ] Shared state is initialized before parallel processes access it
- [ ] All branches in conditionals initialize required values
- [ ] Transaction IDs or tags are initialized and valid

---

## Common Mistakes

### RTL

1. **Missing reset on state variable:**
   ```systemverilog
   logic [7:0] state;  // no reset defined
   ```
   Fix: Add synchronous or asynchronous reset to state.

2. **Incomplete case coverage:**
   ```systemverilog
   case (cmd)
     CMD_A: result = 8'h00;
     CMD_B: result = 8'h01;
     // missing default; result is latched if cmd has other values
   endcase
   ```
   Fix: Add explicit `default: result = 8'h00;`

3. **Inferred latch:**
   ```systemverilog
   if (enable) begin
     output = data;
   end
   // if enable is false, output retains its value (latch)
   ```
   Fix: Assign output explicitly in all branches.

### Testbench

1. **Uninitialized class member:**
   ```systemverilog
   class packet;
     logic [7:0] id;  // no initialization
   endclass
   ```
   Fix: Initialize in constructor or at declaration.

2. **Reading before assignment:**
   ```systemverilog
   logic [7:0] sum;
   sum = sum + 5;  // sum is uninitialized
   ```
   Fix: Initialize sum before use: `logic [7:0] sum = 8'd0;`

3. **X propagation in scoreboard:**
   ```systemverilog
   if (actual_data == expected_data) begin
     // passes if actual_data is X
   end
   ```
   Fix: Check for valid state before comparison: `if (actual_data !== 8'hXX && actual_data == expected_data)`

---

## Summary

| Rule | RTL | Testbench |
|------|-----|-----------|
| **Initialize at declaration or before first use** | Yes (reset blocks) | Yes (constructors, initial blocks) |
| **No implicit/default values** | Yes (explicit reset required) | Yes (constructor initialization required) |
| **Handle unknown values explicitly** | Yes (design failures) | Yes (must assert or check validity) |
| **Cover all code paths** | Yes (case defaults, if coverage) | Yes (all branches initialize) |
| **Document assumptions** | Yes (reset strategy) | Yes (initial state of objects) |

Both standards enforce the same fundamental principle: **every variable and state must be initialized before it is used.**
