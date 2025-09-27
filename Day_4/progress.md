# Day 4: Gate-Level Simulation (GLS) & Verilog Pitfalls

Welcome to **Day 4**! Today’s session dives into crucial verification and language-specific topics that are vital for robust digital design. We will cover:

* **Gate-Level Simulation (GLS)** to verify the synthesized netlist.
* **Blocking vs. Non-Blocking Assignments** and their impact on synthesis.
* **Synthesis-Simulation Mismatches** and how to avoid them.

The hands-on labs will highlight common pitfalls and best practices in RTL coding.

---

## Table of Contents

1.  [Gate-Level Simulation (GLS)](#1-gate-level-simulation-gls)
2.  [Synthesis-Simulation Mismatch](#2-synthesis-simulation-mismatch)
3.  [Blocking vs. Non-Blocking Assignments](#3-blocking-vs-non-blocking-assignments)
    * [3.1 Blocking Statements (`=`)](#31-blocking-statements-)
    * [3.2 Non-Blocking Statements (`<=`)](#32-non-blocking-statements-)
    * [3.3 Comparison Table](#33-comparison-table)
4.  [Labs](#4-labs)
5.  [Summary](#5-summary)

---

## 1. Gate-Level Simulation (GLS)

**GLS** is the process of simulating the **gate-level netlist** produced by the synthesis tool. It's a critical step to confirm that the synthesized circuit is functionally correct and meets timing requirements.

### Why Perform GLS?
* **Synthesis Validation**: Confirms the synthesis tool correctly translated the RTL into logic gates.
* **Timing Verification**: Simulates the design with realistic gate and wire delays (from an SDF file), which is essential for catching timing violations like setup and hold time errors.
* **Testability Check**: Verifies that Design-for-Test (DFT) structures, like scan chains, are working correctly after synthesis.

---

## 2. Synthesis-Simulation Mismatch

A **synthesis-simulation mismatch** is a dreaded scenario where the pre-synthesis RTL simulation behaves differently from the post-synthesis GLS or the final hardware.

**Common Causes:**
* Using non-synthesizable Verilog constructs (e.g., `#delays`, `initial` blocks for logic, `fork-join`).
* Writing ambiguous RTL, such as incomplete sensitivity lists or `if/case` statements that imply latches where none were intended.
* Race conditions caused by improper use of blocking (`=`) and non-blocking (`<=`) assignments.

**Key Point:** Writing clean, synthesizable, and unambiguous RTL is the best way to prevent mismatches.

---

## 3. Blocking vs. Non-Blocking Assignments

Verilog's two assignment operators are a frequent source of bugs for newcomers.

### 3.1 Blocking Statements (`=`)
* **Execution:** Statements are executed **sequentially** in the order they appear. The next statement is "blocked" until the current one is complete.
* **Use Case:** Best for modeling **combinational logic** inside an `always @(*)` block.
* **Inference:** Infers wires and combinational logic.

### 3.2 Non-Blocking Statements (`<=`)
* **Execution:** Statements are scheduled to occur **concurrently**. The right-hand side of all statements is evaluated first, and only then are the left-hand side variables updated.
* **Use Case:** Essential for modeling **sequential logic** (flip-flops and registers) inside an `always @(posedge clk)` block.
* **Inference:** Infers flip-flops.

### 3.3 Comparison Table

| Feature | **Blocking (`=`)** | **Non-Blocking (`<=`)** |
| :--- | :--- | :--- |
| **Execution** | Sequential, immediate | Concurrent, scheduled |
| **Best For** | Combinational Logic | Sequential Logic |
| **Infers** | Logic gates, wires | Flip-flops, registers |
| **Analogy** | A sequence of instructions | A snapshot in time |

---

## 4. Labs

### Lab 1: Ternary Operator MUX
This lab simulates a simple and efficient 2:1 multiplexer using a ternary operator.

```verilog
module ternary_operator_mux (input i0, input i1, input sel, output y);
  assign y = sel ? i1 : i0;
endmodule
````

<img width="2879" height="1799" alt="image" src="https://github.com/user-attachments/assets/4fac5357-ca83-48e0-861f-7d41ad588841" />


-----

### Lab 2: Synthesis Using Yosys

Synthesizing the ternary MUX results in an optimized `sky130_fd_sc_hd__mux2_1` standard cell, as expected.

<img width="1024" height="639" alt="image" src="https://github.com/user-attachments/assets/14f0d6c5-99ba-431e-b81d-b238667c51f9" />


-----

### Lab 3: Gate-Level Simulation (GLS) of MUX

Running GLS on the synthesized netlist confirms that its functional behavior is identical to the original RTL simulation.

```shell
iverilog /path/to/primitives.v /path/to/sky130_fd_sc_hd.v ternary_operator_mux_net.v testbench.v
```

<img width="2879" height="1799" alt="image" src="https://github.com/user-attachments/assets/f6228144-2c75-447f-933e-bb28b36fe67f" />


-----

### Lab 4: Bad MUX Example (Common Pitfalls)

This Verilog module contains common mistakes that lead to synthesis-simulation mismatches.

```verilog
module bad_mux (input i0, input i1, input sel, output reg y);
  // Issues: Incomplete sensitivity list and using non-blocking for combo logic.
  always @ (sel) begin
    if (sel)
      y <= i1;
    else 
      y <= i0;
  end
endmodule
```

The simulation shows that the output `y` only updates when `sel` changes, ignoring changes in `i0` and `i1`.

<img width="1024" height="639" alt="image" src="https://github.com/user-attachments/assets/146e0710-bca2-4592-b2a7-4fb07756fcf1" />


-----

### Lab 5: GLS of Bad MUX

When `bad_mux` is synthesized, the tool often infers a simple MUX, ignoring the incomplete sensitivity list. Running GLS on this netlist would show a correctly functioning MUX, which **mismatches** the buggy behavior seen in the RTL simulation.

<img width="1024" height="639" alt="image" src="https://github.com/user-attachments/assets/e3dad699-599a-4e8e-801b-728e848d4b3d" />


-----

### Lab 6: Blocking Assignment Caveat

This lab demonstrates how the order of blocking assignments can create unexpected logic.

```verilog
module blocking_caveat (input a, input b, input c, output reg d);
  reg x;
  // Bug: `d` is assigned using the OLD value of `x` from the previous simulation event.
  always @ (*) begin
    d = x & c;
    x = a | b;
  end
endmodule
```

The simulation shows unexpected behavior in `d` because it's not using the updated value of `x` within the same `always` block execution.


<img width="1024" height="638" alt="image" src="https://github.com/user-attachments/assets/b1a5f629-ae19-42a7-93ae-5878b8ad2f79" />

-----

### Lab 7: Synthesis of the Blocking Caveat Module

When the corrected version (`x = a | b; d = x & c;`) is synthesized, Yosys correctly infers the intended logic: `d = (a | b) & c`. This is mapped to an efficient OR-AND (`sky130_fd_sc_hd__o21a_1`) standard cell.

<img width="1024" height="639" alt="image" src="https://github.com/user-attachments/assets/6b0b566f-7a71-40f9-a080-1599bc0b0bd4" />


-----

## 5\. Summary

  - **Gate-Level Simulation (GLS)** is an essential step to verify that the synthesized netlist is functionally and temporally correct.
  - **Synthesis-Simulation Mismatches** are often caused by non-synthesizable code or ambiguous RTL. Following strict coding guidelines is key.
  - **Blocking (`=`) vs. Non-Blocking (`<=`)** is a critical concept: use blocking for combinational logic and non-blocking for sequential logic to avoid race conditions and mismatches.

> [\!TIP]
> Always simulate your RTL and gate-level netlist, and pay close attention to warnings from synthesis and simulation tools\!

```
```
