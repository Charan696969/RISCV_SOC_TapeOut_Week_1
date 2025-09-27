
# Day 3: Logic Optimization & Sequential Circuit Synthesis

On **Day 3**, the focus shifted to **RTL optimization** and the synthesis of **sequential circuits**. These labs explore how the Yosys synthesis tool intelligently simplifies various Verilog constructs into efficient, minimal gate-level netlists using the SKY130 standard cell library.

---

## 1. Lab 1: Ternary Operator as an AND Gate

This lab demonstrates how Yosys optimizes a conditional assignment into a basic logic gate.

### Verilog Code (`opt_check.v`)
```verilog
module opt_check (
  input a, 
  input b, 
  output y
);
	assign y = a ? b : 1'b0;
endmodule
````

### Logic Analysis

The expression `y = a ? b : 0` is logically equivalent to a 2-input **AND** operation (`y = a & b`). If `a` is `0`, the output is `0`. If `a` is `1`, the output is `b`. This matches the truth table for an AND gate.

### Synthesis Result

Yosys correctly identifies this behavior and synthesizes the logic into a single **`sky130_fd_sc_hd__and2_1`** standard cell.

<img width="1024" height="639" alt="image" src="https://github.com/user-attachments/assets/096f1f29-b216-42d0-a33c-9c9500e1bd0b" />


-----

## 2\. Lab 2: Ternary Operator as an OR Gate

Similar to the first lab, this example shows optimization into a different logic gate.

### Verilog Code (`opt_check2.v`)

```verilog
module opt_check2 (
  input a, 
  input b, 
  output y
);
	assign y = a ? 1'b1 : b;
endmodule
```

### Logic Analysis

The expression `y = a ? 1 : b` is logically equivalent to a 2-input **OR** operation (`y = a | b`). If `a` is `1`, the output is `1`. If `a` is `0`, the output is `b`. This matches the truth table for an OR gate.

### Synthesis Result

Yosys simplifies the RTL to a single **`sky130_fd_sc_hd__or2_1`** standard cell.

<img width="1024" height="639" alt="image" src="https://github.com/user-attachments/assets/94a79b36-6d5c-4837-a67e-f1fd0087f741" />


-----

## 3\. Lab 3: Synthesis of a 3-Input AND Gate

This lab shows a slightly more complex combinational circuit.

### Verilog Code (`opt_check3.v`)

```verilog
module opt_check3 (
  input a, 
  input b, 
  input c,
  output y
);
	assign y = a & b & c;
endmodule
```

### Logic Analysis

This module describes a simple 3-input **AND** gate. The output `y` is high only when inputs `a`, `b`, and `c` are all high.

### Synthesis Result

The synthesis tool maps this directly to a **`sky130_fd_sc_hd__and3_1`** standard cell.

<img width="1024" height="639" alt="image" src="https://github.com/user-attachments/assets/2a2bbe31-b105-4d8b-81d3-6aad159d3bcc" />


-----

## 4\. Lab 4: Complex Ternary as an XNOR Gate

This lab demonstrates Yosys's ability to simplify a complex, nested expression into a single, efficient gate.

### Verilog Code (`opt_check4.v`)

```verilog
module opt_check4 (
  input a, 
  input b, 
  input c, 
  output y
);
  assign y = a ? (b ? (a & c) : c) : (!c);
endmodule
```

### Logic Analysis

Although the expression seems complex, it simplifies significantly:

  - If `a = 1`, the expression becomes `y = (b ? (1 & c) : c)`, which simplifies to `y = (b ? c : c)`, so **`y = c`**.
  - If `a = 0`, the expression becomes **`y = !c`**.
    This behavior, where `y = c` when `a=1` and `y = !c` when `a=0`, is the definition of an **XNOR** gate (`y = a XNOR c`, or `y = ~(a ^ c)`). The input `b` is irrelevant.

### Synthesis Result

Yosys successfully performs this optimization and synthesizes the logic into a single **`sky130_fd_sc_hd__xnor2_1`** cell, correctly identifying that input `b` is unused.

<img width="2879" height="1799" alt="image" src="https://github.com/user-attachments/assets/272568be-2727-46c3-989f-0fd788ad256d" />


-----

## 5\. Lab 5: D-Flip-Flop with Constant Input

This lab introduces sequential logic, showing how Yosys handles a flip-flop with a tied input.

### Verilog Code (`dff_const1.v`)

```verilog
module dff_const1(
  input clk, 
  input reset, 
  output reg q
);
  always @(posedge clk, posedge reset)
  begin
    if(reset)
      q <= 1'b0;
    else
      q <= 1'b1;
  end
endmodule
```

### Logic Analysis

This describes a standard D-type Flip-Flop (DFF) with an asynchronous, active-high reset. When `reset` is high, `q` is `0`. When `reset` is low, on every rising clock edge, `q` is loaded with a constant value of `1`. This is equivalent to a DFF whose **D input is tied to logic high (VDD)**.

### Synthesis & Simulation

Yosys synthesizes this using a **`sky130_fd_sc_hd__dfxtp_1`** (D-Flip-Flop with Asynchronous Reset) cell. The waveform confirms that `q` stays low during the reset period and then goes high on the first clock edge after the reset is de-asserted.

<img width="2879" height="1786" alt="image" src="https://github.com/user-attachments/assets/13575c9d-9144-495d-9c7d-0da70ee98e22" />

<img width="2879" height="1799" alt="image" src="https://github.com/user-attachments/assets/42a877bc-ac7f-4fb4-86e3-01f4145b5947" />

-----

## 6\. Lab 6: D-Flip-Flop with Constant Output

This final lab is an extreme case of optimization for sequential circuits.

### Verilog Code (`dff_const2.v`)

```verilog
module dff_const2(
  input clk, 
  input reset, 
  output reg q
);
  always @(posedge clk, posedge reset)
  begin
    if(reset)
      q <= 1'b1;
    else
      q <= 1'b1;
  end
endmodule
```

### Logic Analysis

The code specifies that the output `q` should be set to `1` on reset, and also set to `1` on every clock edge. Therefore, the output `q` is **always `1`**, regardless of the `clk` or `reset` inputs. The logic does not depend on any sequential behavior.

### Synthesis & Simulation

Yosys correctly deduces that no flip-flop is needed. The output `q` is simply tied to a constant logic high value. The post-mapping schematic shows this implemented using tied inverter cells, a common technique for generating stable power and ground signals. The waveform confirms that `q` is always high.

<img width="2879" height="1799" alt="image" src="https://github.com/user-attachments/assets/b7fa0e38-48d9-4081-8214-b6354b744cfa" />

<img width="1024" height="639" alt="image" src="https://github.com/user-attachments/assets/2997f7ed-232c-459a-8474-9c610d9b2069" />

-----

## 7. Lab 7: D-Flip-Flop with Constant Output (Low)

This lab is the logical counterpart to Lab 6, demonstrating optimization to a constant low signal.

### Verilog Code (`dff_const3.v`)
```verilog
module dff_const3(
  input clk, 
  input reset, 
  output reg q
);
  always @(posedge clk, posedge reset)
  begin
    if(reset)
      q <= 1'b0;
    else
      q <= 1'b0;
  end
endmodule
````

### Logic Analysis

This code dictates that the output `q` must be `0` when reset is active, and also `0` on every clock edge when not in reset. Consequently, the output `q` is **always `0`**. The `clk` and `reset` inputs have no actual effect on the final steady-state value.

### Synthesis & Simulation

As expected, Yosys optimizes this entire module away. No flip-flop is synthesized. The output `q` is simply connected directly to ground (logic 0). The simulation waveform confirms this behavior, showing `q` remaining low for the entire duration.

<img width="2879" height="1799" alt="image" src="https://github.com/user-attachments/assets/5d32d58c-4c9b-43f5-96b9-cfd36f8e16ae" />

<img width="2868" height="1791" alt="image" src="https://github.com/user-attachments/assets/60ffc4ad-e832-443a-927c-0c7600c50367" />


```
```
Of course. Here is the new section for the `counter_opt` lab.

## 8. Lab 8: Synthesis of an Optimized Counter

This lab is a key demonstration of how synthesis tools perform sequential optimization by removing unused logic, a process often called "logic trimming" or "un-used logic removal."

### Verilog Code (`counter_opt.v`)
```verilog
module counter_opt (
  input clk, 
  input reset, 
  output q
);
  reg [2:0] count;
  assign q = count[0];

  always @(posedge clk, posedge reset)
  begin
    if(reset)
      count <= 3'b000;
    else
      count <= count + 1;
  end

endmodule
````

### Logic Analysis

The module describes a standard 3-bit synchronous binary counter. However, the crucial part is the output assignment: `assign q = count[0];`. This means the final output `q` is only connected to the **least significant bit (LSB)** of the counter.

The other two flip-flops, which would hold `count[1]` and `count[2]`, are never observed at the output. An efficient synthesis tool should recognize that this logic is redundant and remove it completely. The only required logic is a circuit that generates the `count[0]` signal, which simply toggles on every clock cycle.

### Synthesis Result

Yosys correctly performs this optimization. It synthesizes the full 3-bit counter, but during the optimization and cleanup passes, it identifies that only the flip-flop for `count[0]` has a path to an output. The other two are removed. The resulting circuit for `count[0]` is a simple **Toggle Flip-Flop (T-Flip-Flop)**, which is the most efficient way to implement a divide-by-2 counter.

<img width="2879" height="1799" alt="image" src="https://github.com/user-attachments/assets/db1cb52b-d480-4c3a-8487-0e2412324d70" />

<img width="1024" height="639" alt="image" src="https://github.com/user-attachments/assets/997356be-57d5-458a-ba46-41468d44c6e6" />

```
```

## 9\. Summary

✅ **Day 3 Highlights:**

  - **Combinational Optimization:** Yosys can simplify RTL expressions (like ternary operators) into the most efficient base logic gates (AND, OR, XNOR).
  - **Redundancy Removal:** The synthesis tool automatically identifies and removes unused inputs, as seen in the XNOR gate lab.
  - **Sequential Optimization:** Yosys understands the behavior of sequential elements like flip-flops. It will synthesize them correctly but will also eliminate them entirely if their output is constant, replacing them with a simple tie-to-power cell to save area and power.

<!-- end list -->

```
```
