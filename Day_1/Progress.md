# 🚀 2-to-1 Multiplexer Lab (Simulation + Synthesis)
## 1. Design File (good_mux.v)

Verilog code for 2-to-1 multiplexer
File: good_mux.v


```verilog
module good_mux (input i0, input i1, input sel, output reg y);
always @ (*)
begin
if(sel)
y <= i1;
else
y <= i0;
end
endmodule
```


## 2. Testbench File (tb_good_mux.v)
Testbench for 2-to-1 multiplexer
File: tb_good_mux.v

```verilog 
`timescale 1ns/1ps
module tb_good_mux;
reg i0, i1, sel;
wire y;

good_mux uut (.i0(i0), .i1(i1), .sel(sel), .y(y));

initial begin
    $dumpfile("tb_good_mux.vcd");
    $dumpvars(0, tb_good_mux);

    i0 = 0; i1 = 0; sel = 0; #10;
    i0 = 1; i1 = 0; sel = 0; #10;
    i0 = 0; i1 = 1; sel = 1; #10;
    i0 = 1; i1 = 1; sel = 1; #10;

    $finish;
end


endmodule
```

## 3. Simulate the Design

Compile the design and testbench:

```shell
iverilog good_mux.v tb_good_mux.v
```
Run the simulation:

```shell
./a.out
```
View the waveform:

```shell
gtkwave tb_good_mux.vcd
```
<img width="2048" height="1279" alt="image" src="https://github.com/user-attachments/assets/2ced70a7-888a-46ef-bba6-8b8b1bb749cf" />

## 4. Code Analysis

Inputs: i0, i1 (data), sel (select line)

Output: y (registered output)

Logic: If sel = 1, y = i1; else y = i0.

## 5. Introduction to Yosys & Gate Libraries

Yosys = open-source synthesis tool that converts Verilog to a gate-level netlist.

Synthesis: HDL → logic circuit

Optimization: Speed/area improvements

Tech Mapping: Maps logic to cells in .lib

Verification: Checks correctness

## 6. Synthesis Lab with Yosys

Run the following steps:

Start Yosys
```shell
yosys
```
Read liberty library
```shell
read_liberty -lib /address/to/sky130_fd_sc_hd__tt_025C_1v80.lib
```
Read Verilog code
```shell
read_verilog /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files/good_mux.v
```
Synthesize design
```shell
synth -top good_mux
```
Technology mapping
```shell
abc -liberty /address/to/sky130_fd_sc_hd__tt_025C_1v80.lib
```
Visualize netlist
```shell
show
```
Netlist screenshot
<img width="2048" height="1279" alt="image" src="https://github.com/user-attachments/assets/3c2de0f3-de9d-4374-ba3b-7ec5cc013617" />


## 7. Summary

Wrote design + testbench for mux

Simulated with iverilog + GTKWave

Learned basics of Yosys synthesis

Understood why .lib has multiple versions of the same gate
