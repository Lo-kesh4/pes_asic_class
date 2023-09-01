# PES_ASIC_CLASS
This Repository Guides you to complete ASIC flow from scratch (GUIDE: Kunal Ghosh)

## The COURSE files are present under those respective dau folders 

Update
```
sudo apt update
sudo apt upgrade
```

## Install the Prerequisites(for ubuntu)
```
chmod +x run.sh
./run.sh
```
The installed contents will be available at ~/riscv_toolchain

# Introduction
#### Flow : HLL -> ALP -> Binary -> (HDL) -> GDS
- HLL -> High-level language (c, c++) 
- ALP -> Assembly level program
- HDL -> Hardware Description Language
- GDS -> Graphic Data System (layout)

###### The Hardware needs to understand what the Application software is saying it to do.This relation can be eshtablished by System Software

____System Software____
- OS: Operating System: Handles IO, memory allocation, Low-level system function
- Compiler: Convert the input to hardware-dependent instruction
- Assembler: Convert the instructions provided by the compiler to Binary format
- HDL: A program that understands the Binary pattern and maps it to a netlist
- GDS: Layout

# COURSE 
<details>
<summary>DAY 1: Introduction to RISCV ISA and GNU Compiler Toolchain</summary>
<br>

## Introduction to Risc-v Basic Keywords
- **Instruction Set Architecture(ISA)**
  - An Instruction Set Architecture (ISA) refers to the set of instructions that a computer's central processing unit (CPU) can understand and execute. It defines the interface between software and hardware, specifying the operations that a CPU can perform, the data types it can manipulate, and the memory addressing modes it supports.

- **Risc-V ISA**
  - Risc-V ISA is an open-source ISA that has simpler and fixed length instructions that allows us to create custom processors for specific needs without being tied to proprietary architectures
 
- **Tools Used for the flow**
  - As we are aware of the flow, we will be using Risc-v ISA ALP and the RTL used will be picorv32a (We will be using rv64i during initial stages)

# Goal : Any High level Program that is written should be able to get executed in our CHIP

### List of well-known extensions present in Risc-V ISA

``` rv32i``` ``` rv64i``` ```rv32imc``` ```rv64imc``` ```rv32imafdc``` ```rv64imafdc``` ```rv32imcb``` ```rv64imcb``` ```rv32imc_sv32``` ```rv64gcv```

### Extensions and their Applications

- **I (Integer)** :The I set includes the base integer instruction set for RISC-V. It provides fundamental integer arithmetic and logical operations, data movement, and control flow instructions.
  - ADD, SUB, AND, OR, XOR, ADDI, SLTI, JAL, BEQ, LW

- **M (Multiply and Divide)** : The M set adds integer multiplication and division instructions to the base integer set. These instructions are particularly useful for arithmetic-heavy computations.
  - MUL, MULH, DIV, REM
  
- **A (Atomic)** : The A set introduces atomic memory access instructions. These instructions enable multiple operations on memory locations to be performed atomically, ensuring that other processors or threads cannot observe intermediate states.
  - LR (Load-Reserved), SC (Store-Conditional), AMO (Atomic Memory Operation)
  
- **F (Single-Precision Floating-Point)**: The F set adds single-precision floating-point instructions. These instructions enable arithmetic operations on 32-bit floating-point numbers.
  - FADD.S, FSUB.S, FMUL.S, FDIV.S, FCVT.W.S, FCVT.S.W

- **D (Double-Precision Floating-Point)** : The D set includes double-precision floating-point instructions. These instructions allow arithmetic operations on 64-bit floating-point numbers.
  - FADD.D, FSUB.D, FMUL.D, FDIV.D, FCVT.W.D, FCVT.D.W

- **C (Compressed)** : The C set introduces a compressed instruction format that reduces the size of code. Compressed instructions maintain the same functionality as their non-compressed counterparts but use shorter encodings.
  - C.ADDI4SPN, C.LWSP, C.ADDI, C.SW, C.JALR, C.BEQZ

- **G (Atomic and Lock-Free Operations)** : The G set, also known as the "GAS Set," is an alternative to the A set. It focuses on providing atomic and lock-free instructions to simplify hardware implementation.
  - LRV (Load-Reserved Variant), SCV (Store-Conditional Variant), AMO (Atomic Memory Operation Variants)

- **V (Vector)** :The V set adds vector instructions to the ISA, enabling Single Instruction, Multiple Data (SIMD) operations. These instructions allow efficient parallel processing of data elements in vectors.
  - VADD, VMUL, VFMADD, VLW, VSW

- **S (Supervisor)** : The S set, often used in privileged modes, includes instructions for managing and interacting with the supervisor-level operations of the system, such as handling exceptions and interrupts.
  - ECALL, EBREAK, SRET, MRET, WFI

- **B (Bit Manipulation)** : The B set introduces instructions for bit manipulation operations, allowing efficient manipulation of individual bits in registers and memory.
  - ANDI, ORI, XORI, SLLI, SRLI, SRAI

## 1. Create a simple C program That calculates sum from 1 to N -> sum1toN.c

____Compile it using C compiler____
```
gcc sum1toN.c -o 1toN.o
./1toN.o
```
-o allows you to name your output file

![Screenshot from 2023-08-19 16-53-59](https://github.com/yagnavivek/PES_ASIC_CLASS/assets/131575546/9ddb5124-dead-4fa0-a851-93aefa3cf6b7)


____compile using riscv compiler and view the output____
```
riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o 1toN.o sum1toN.c
spike pk 1toN.o
```
For reducing the number of instructions in ALP 
```
riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o 1toN.o sum1toN.c
```

![Screenshot from 2023-08-19 17-22-44](https://github.com/yagnavivek/PES_ASIC_CLASS/assets/131575546/998affd3-8ee0-4528-bc6d-c5442d1c30b1)

- -Onumber : level of optimisation required
- -mabi : specifies the ABI (Application Binary Interface) to be used during code generation according to the requirements
- -march : specifies target architecture

______We can check the different options available for all these fields using the commands______ 
go to the directory where riscv64-unkonwn-elf is present
- -O1 : ``` riscv64-unkonwn-elf --help=optimizer```
- -mabi : ```riscv64-unknown-elf-gcc --target-help```
- -march : ```riscv64-unknown-elf-gcc --target-help```

____To view the disassembled ALP code____
```
riscv64-unknown-elf-objdump -d 1toN.o
```
If you want to have less ALP code
```
riscv64-unknown-elf-objdump -d 1toN.o | less
```
- run the above command in another terminal to follow the below process 

____To debug the ALP generated by the compiler____
```
spike -d pk 1toN.o
```
![Screenshot from 2023-08-20 00-47-42](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/2185cfa4-f233-4fae-9a6e-042360ec571c)

- until pc 0 100b0 : the compiler debugs the ALP code until ```100b0```
- press ENTER: debug next instruction and successive ENTER debugs successive instructions 
- reg 0 a0 : shows content of register a0 
- q: quit the debug process
- you can clearly observe in the above picture that the value of the register is changing after debugging instruction

## 2.Signed and Unsigned Numbers
- RISC-V doubleword can represent 0 to (2^64 - 1) unsigned numbers or positive numbers.
- RISC-V doubleword can represent 0 to (2^63 - 1) positive and 1 to (-2^63) negative numbers.
- Right a C program for displaying the highest and lowest numbers of unsigned and signed.
```
highlow.c
```
![Screenshot from 2023-08-20 01-27-44](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/04c0a92e-37cc-4818-9eda-9686718106e5)

</details>
<details>
<summary>DAY 2 : Introduction to ABI and Basic Verification Flow </summary>
<br>

# Labwork using ABI Function Calls
## Algorithm for C Program using ASM
- Incorporating assembly language code into a C program can be done using inline assembly or by linking separate assembly files with your C code.
- When you call an assembly function from your C code, the C calling convention is followed, including pushing arguments onto the stack or passing them in registers as required.
- The program executes the assembly function, following the assembly instructions you've provided.

## Review ASM Function Calls
- We wrote C code in one file and your assembly code in a separate file.
- In the assembly file, we declared assembly functions with appropriate signatures that match the calling conventions of your platform.

**C Program**
`custom1to9.c`
  ``` c
  #include <stdio.h>
  
  extern int load(int x, int y);
  
  int main()
  {
    int result = 0;
    int count = 9;
    result = load(0x0, count+1);
    printf("Sum of numbers from 1 to 9 is %d\n", result);
  }
```
**Asseembly File**
`load.s`
``` s
.section .text
.global load
.type load, @function

load:

add a4, a0, zero
add a2, a0, a1
add a3, a0, zero

loop:

add a4, a3, a4
addi a3, a3, 1
blt a3, a2, loop
add a0, a4, zero
ret
```
## Simulate C Program using Function Call
**Compilation:** To compile C code and Asseembly file use the command

`riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o custom1to9.o custom1to9.c load.s` 

this would generate object file `custom1to9.o`.

**Execution:** To execute the object file run the command 

`spike pk custom1to9.o`

![Screenshot from 2023-08-21 23-30-51](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/5ff4eef3-ff03-46aa-bb5e-d05397d53051)

## Lab to Run C-Program on RISCV-CPU

`git clone https://github.com/kunalg123/riscv_workshop_collaterals.git`

`cd riscv_workshop_collaterals`

`ls -ltr`

`cd labs`

`ls -ltr`

`chmod 777 rv32im.sh`

`./rv32im.sh`

![Screenshot from 2023-08-21 23-35-55](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/ef5ba018-c420-40d5-8a93-45ca866051cc)

</details>
<details>
<summary>DAY 3: Introduction to Open-Source Simulator iVerilog </summary>
<br>
 - **Simulator**
   - It is a tool used for simulating the design. It looks for the changes in the input signals to evaluate the outputs.
   - If there is no change in the inputs, the simulator doesn't evaluate the outputs.
   - RTL is checked for adherence to the spec by simulating the design.
   - The tool used here is **iverilog** .

- **iVerilog**
  -  It is an open-source Verilog simulator used for testing and simulating digital circuit designs described in the Verilog hardware description language (HDL).
  -  Both the design and the testbench are fed to the simulator and it produces a vcd (value change dump) file.
  -  In order to view the vcd file, we use the GTKwave where we can see the waveforms.
    
   <img width="526" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/37b643b5-e41e-425d-85f0-a55d7e190571">

- **Design**
  - It is the actual verilog code or set of Verilog codes that ahs the intended functionality to meet the required specifications.
  - Verilog is used to describe the behavior and structure of digital circuits at different levels of abstraction, from high-level system descriptions down to low-level gate-level representations. 

- **Testbench**
  - A testbench is a specialized Verilog module or program used to verify the functionality and behavior of another Verilog module, circuit, or design. Testbenches are essential for testing and simulating digital designs before they are synthesized or manufactured as physical chips.
  - It is a setup to apply a stimulus to the design to check its functionality.

    <img width="526" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/72e6ffe4-abba-41f1-b79f-240f125b410b">


## Labs using iVerilog and GTKwave

<details>
<summary> Introduction to Lab </summary>

+ Make a directory named vsd `mkdir vsd`.
+ `cd vsd`.
+ `git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git`
+ Creates a folder called `sky130RTLDesignAndSynthesisWorkshop` in the `vsd` directory.

![Screenshot from 2023-08-30 16-23-04](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/f656f864-9473-4657-9bec-95945d3b087e)

- my_lib : contains all the library files

  - lib : contains sky130 standard cell library used for our synthesis

  - verilog_model : contains all the standard cell verilog modules of the standard cells contained in the .lib

  - verilog_files : contains all the verilog source files and testbench files which are required for labs


<details>
<summary> iVerilog GTKwave Part-1 </summary>	


+ `cd vsd/sky130RTLDesignAndSynthesisWorkshop/verilog_files`

+ we have loaded the source code along with the testbench code into the iverilog simulator

+ `iverilog good_mux.v tb_good_mux.v`

+ We can see that an output file `a.out` has been created.

+ `./a.out`

+ The output of the iverilog, a vcd file,  is created which is loaded into the simualtor gtkwave.

+ ` gtkwave tb_good_mux.vcd `

![Screenshot from 2023-08-30 16-33-36](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/e97618e0-892b-42cc-9274-c086798d95e6)

![Screenshot from 2023-08-30 16-35-12](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/d7bd97ad-4c12-4477-8dbf-642dcc395c0c)


<details>
<summary> iVerilog GTKwave Part-2 </summary>

+ In order to view the contents of the files,

+ `gvim tb_good_mux.v -o good_mux.v`

![Screenshot from 2023-08-30 16-42-45](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/4fb64991-68c4-4117-92bd-f2ec3e9b2901)


## Introduction to Yosys and Logic Synthesis

<details>
<summary> Introduction to Yosys </summary>

+ **Synthesizer**
  - It is a tool used for converting RTL design code to netlist.
  - Here, the synthesizer used is **Yosys**.

+ **Yosys**
  - It is an open-source framework for Verilog RTL synthesis and formal verification.
  - Yosys provides a collection of tools and algorithms that enable designers to transform high-level RTL (Register Transfer Level) descriptions of digital circuits into optimized gate-level representations suitable for physical implementation on hardware.
 
  <img width="561" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/5f879aaa-ec65-4362-9f91-f39999069732">

   - Design and .lib files are fed to the synthesizer to get a netlist file.
   - **Netlist** is the representation of the design in the form of standard cells in the .lib
     
+ Commands used to perform different opertions:
  - `read_verilog` to read the design
  - `read_liberty` to read the .lib file
  - `write_verilog` to write out the netlist file
 
+ To verify the synthesis

<img width="566" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/fd73f6b8-f594-4e4f-bb1a-b600fb4475f8">

   - Netlist along with the tesbench is fed to the iverilog simulator.
   - The vcd file generated is fed to the gtkwave simulator.
   - The output on the simulator must be same as the output observed during RTL simulation.
   - Same RTL testbench can be used as the primary inputs and primary outputs remain same between the RTL design and synthesised netlist.


<details>
<summary> Introduction to Logic Synthesis </summary>
+ **Logic Synthesis**
  - Logic synthesis is a process in digital design that transforms a high-level hardware description of a digital circuit, typically in a hardware description language (HDL) like Verilog or VHDL, into a lower-level representation composed of logic gates and flip-flops.
  - The goal of logic synthesis is to optimize the design for various criteria such as performance, area, power consumption, and timing.

 + **.lib**
   - It is a collection of logical modules like And, Or, Not etc.
   - It has different flavors of same gate like 2 input AND gate, 3 input AND gate etc with different performace speed.
  
+ **Why different flavors  of gate?**
  - In order to make a circuit faster, the clock frequency should be high.
  - For that, the time period of the clock should be as low as possible.
  
<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/bc2242db-49e8-4c19-a06e-8f8e82f55729">

+ In a sequential circuit, clock period depends on:
  - Clock to Q of flip-flop A.
  - Propagation delay of combinational circuit.
  - Setup time of flip-flop B.

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/112de4cd-6e0c-46ec-ad94-0cb6540af7e1">

+ **Why need fast and slow cells?**
  - To ensure that there are no HOLD issues at flip-flop B, we require slow cells.
  - For a smaller propagation time, we need faster cells.
  - The collection forms the .lib

+ **Faster Cells vs Slower Cells**
  - Load in digital circuit is of Capacitence.
  - Faster the charging or dicharging of capacitance, lesser is the cell delay.
  - However, for a quick charge/ discharge of capacitor, we need transistors capable of sourcing more current i.e, we need **wide transistors**.
  - Wider transistors have lesser delay but consume more area and power.
  - Narrow transistors have more delay but consume less area and performance.
  - Faster cells come with a cost of area and power.
 
+ **Selection of the Cells**
  - We have to guide the Synthesizer to choose the flavour of cells that is optimum for implementation of logic circuit.
  - More use of faster cells leads to bad circuit in terms of power and area and also hold time violations.
  - More use of slower cells leads to sluggish circuits amd may not meet the performance needs.
  - Hence the guidance is offered to the synthesiser in the form of **constraints**.
 

## Labs using Yosys and Sky130 PDKs
<details>
<summary> Yosys good_mux  </summary>	

+ To invoke **yosys**
  - `cd`
  - `cd vsd/sky130RTLDesignAndSynthesisWorkshop/verilog_files`
  - Type `yosys`
 
  ![Screenshot from 2023-08-30 16-51-31](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/2f4da88e-72af-49c7-90e5-7ae68e84885a)

+ To read the library
    
     ` read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
    
+ To read the design

    `read_verilog good_mux.v`

 + To syntheis the module

      ` synth -top good_mux`

  <img width="334" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/f75014c5-c9f0-4813-ae56-ddbb71f79111">
  <img width="287" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/4ab7cd35-c5d7-4ca9-a310-8d76056a67e1">

+ To generate the netlist

  `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`

  <img width="271" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/97358dfc-ec44-40e0-ac79-97c3576e6300">

It gives a report of what cells are used and the number of input and output signals.

+ To see the logic realised

  `show`

  <img width="300" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/9b2957ea-f0bc-4f2c-b796-aa565bd0865c">

  The mux is completely realised in the form of sky130 library cells.

+ To write the netlist

   - `write_verilog good_mux_netlist.v`
   - `!gvim good_mux_netlist.v`
     
   - To view a simplified code
     
     ` write_verilog -noattr good_mux_netlist.v`
     
     `!gvim good_mux_netlist.v`
  
  
<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/74fc2a01-3c35-4db1-8220-96595c6c236e">

<img width="479" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/7adc16f5-f635-4532-b90c-a9f9c496f95f">

</details>
</details>
</details>
</details>
</details>
</details>
</details>

<details>
<summary>DAY 4:GLS, SYnthesis solution mismatch</summary>
<details>
<summary> Introduction to Dot Lib </summary>	

+ To view the contents in the .lib

  `gvim ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`

  <img width="443" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/91edd5d4-bb82-48ec-b0bd-ca233d8a8063">

  + The first line in the file `library ("sky130_fd_sc_hd__tt_025C_1v80") ` :
    
    - tt : indicates variations due to process and here it indicates **Typical Process**.
    - 025C : indicates the variations due to temperatures where the silicon will be used.
    - 1v80 : indicates the variations due to the voltage levels where the silicon will be incorporated.
+ It also displays the units of various parameters.

  <img width="284" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/d01d750e-fc1c-4de0-8e72-e6842c14f90b">
  <img width="229" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/39f26ac7-7302-4dc7-a517-6a5a031e2cae">

+ It gives the features of the cells
+ To enable line number `:se nu`
+ To view all the cells `:g//`
+ To view any instance `:/instance`
+ Since there are 5 inputs, for all the 32 possible combinations, it gives the delay, power and all the other parameters for each cell.
+ The below image shows the power consumption and area comparision.
  
<img width="911" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/2a6b20a3-33d1-47e0-814f-6cff100ec2a7">

## Hierarchical vs Flat Synthesis
<details>
<summary> Hierarchical Synthesis Flat Synthesis </summary>	

**Hierarchical Synthesis**
  Hierarchical synthesis is an approach in digital design and logic synthesis where complex designs are broken down into smaller, more manageable modules or sub-circuits, and each module is synthesized individually. These synthesized modules are then integrated back into the overall design hierarchy. This approach helps manage the complexity of large designs and allows designers to work on different parts of the design independently.
  
+ The file we used in this lab is `multiple_modules.v`

  - `cd vsd/sky130RTLDesignAndSynthesisWorkshop/verilog_files`
  -  `gvim multiple_modules.v`

<img width="321" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/384b4475-a6e7-4905-9a70-cfdff657e6db">

+  `multiple_modules` instantiates `sub_module1` and `sub_module2`

+  Launch `yosys`
+  read the library file  `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+  read the verilog file ` read_verilog multiple_modules.v`
+  `synth -top multiple_modules` to set it as top module
![Screenshot from 2023-08-30 21-01-12](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/b9ee4ecd-a66c-4d11-a023-6f0db699c46a)
![Screenshot from 2023-08-30 21-01-39](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/8cea5682-dc93-4085-9598-06ffaa22d341)

+  `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ To view the netlist `show multiple_modules`

  <img width="304" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/721a0563-1fbe-4ce7-975c-0ef8d50e7fe6">

- Here it shows `sub_module1` and `sub_module2` instead of AND gate and OR gate.

+ `write_verilog -noattr multiple_modules_hier.v`
+ `!gvim multiple_modules_hier.v`

<img width="371" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/fcc68430-e284-4b54-80af-dfbcfbade0ea">
 <img width="300" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/4e8ffd10-6efb-4d2d-878e-221d685502c1">

 **Flattened Synthesis**
  Flattened synthesis is the opposite of hierarchical synthesis. Instead of maintaining the hierarchical structure of the design during synthesis, flattened synthesis combines all modules and sub-modules into a single, flat representation. This means that the entire design is synthesized as a single unit, without preserving the modular organization present in the original high-level description.

+  Launch `yosys`
+  read the library file  `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+  read the verilog file ` read_verilog multiple_modules.v`
+  `synth -top multiple_modules` to set it as top module
+  `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `flatten` to write out a flattened netlist
+ `show`

<img width="924" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/bd069e1f-4da5-473a-b041-562cbef042f0">

+ `write_verilog -noattr multiple_modules_flat.v`
+ `!gvim multiple_modules_flat.v`
  
<img width="365" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/18760a81-9f03-4b11-9b8f-dd4758a25ab7">
<img width="300" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/e3a80209-7339-4cef-833c-2c3bb1fc4dec">

## Various Flop Coding Styles and Optimization
<details>
<summary> Why Flops and Flop Coding Styles</summary>	

**Why do we need a Flop?**

+ A flip-flop (often abbreviated as "flop") is a fundamental building block in digital circuit design.
+ It's a type of sequential logic element that stores binary information (0 or 1) and can change its output based on clock signals and input values.
+ In a combinational circuit, the output changes after the propagation delay of the circuit once inputs are changed.
+ During the propagation of data, if there are different paths with different propagation delays, then a glitch might occur.
+ There will be multiple glitches for multiple combinational circuits.
+ Hence, we need flops to store the data from the combinational circuits.
+ When a flop is used, the output of combinational circuit is stored in it and it is propagated only at the posedge or negedge of the clock so that the next combinational circuit gets a glitch free input thereby stabilising the output.
+ We use control pins like **set** and **reset** to initialise the flops.
+ They can be synchronous and asynchronous.

**D Flip-Flop with Asynchronous Reset**
+ When the reset is high, the output of the flip-flop is forced to 0, irrespective of the clock signal.
+ Else, on the positive edge of the clock, the stored value is updated at the output.

 `gvim dff_asyncres_syncres.v`
 
 <img width="445" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/582609c7-faf6-4981-9643-ec5ad543b65f">

**D Flip_Flop with Asynchronous Set**
+ When the set is high, the output of the flip-flop is forced to 1, irrespective of the clock signal.
+ Else, on positive edge of the clock, the stored value is updated at the output.

`gvim dff_async_set.v`

<img width="357" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/f45ca71f-8eef-402a-a966-9035a51fa21d">

**D Flip-Flop with Synchronous Reset**
+ When the reset is high on the positive edge of the clock, the output of the flip-flop is forced to 0.
+ Else, on the positive edge of the clock, the stored value is updated at the output.

  `gvim dff_syncres.v`

<img width="409" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/d22a7aa2-059f-48bd-b0c4-32a294248c8b">

**D Flip-Flop with Asynchronous Reset and Synchronous Reset**
+ When the asynchronous resest is high, the output is forced to 0.
+ When the synchronous reset is high at the positive edge of the clock, the output is forced to 0.
+ Else, on the positive edge of the clock, the stored value is updated at the output.
+ Here, it is a combination of both synchronous and asynchronous reset DFF.

`gvim dff_asyncres_syncres.v`

<img width="439" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/8ee2f2a5-31e9-447c-a23f-b347fc7b642c">

<details>
<summary> Lab Flop Synthesis Simulations </summary>	

**D Flip-Flop with Asynchronous Reset**
+ **Simulation**
  - `cd vsd/sky130RTLDesignAndSynthesisWorkshop/verilog_files`
  - `iverilog dff_asyncres.v tb_dff_asyncres.v`
  - `./a.out`
  - `gtkwave tb_dff_asyncres.vcd`

![Screenshot from 2023-08-30 21-29-12](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/99d1a373-1cb4-4d73-9181-fa3f7609abe0)

<img width="500" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/326bf88d-74d9-407f-8c45-1e1e28ea1911">

+ **Synthesis**
  - `cd vsd/sky130RTLDesignAndSynthesisWorkshop/verilog_files`
  - `yosys`
  - `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
  - `read_verilog dff_asyncres.v`
  - `synth -top dff_asyncres`
  - `dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
  - `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
  - `show`

    <img width="925" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/92f225cf-a014-4a89-be7a-a9560a6d6359">

 **D Flip_Flop with Asynchronous Set**
 + **Simulation**
   - `cd vsd/sky130RTLDesignAndSynthesisWorkshop/verilog_files`
   - `iverilog dff_async_set.v tb_dff_async_set.v`
   - `./a.out`
   - `gtkwave tb_dff_async_set.vcd`

![Screenshot from 2023-08-30 21-52-24](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/46bda055-83d9-49e0-a4ed-2aa46b55fa8f)
<img width="500" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/51dd2cf5-ea6c-4b00-bf2a-5ae8674e2272">

+ **Synthesis**
  - `cd vsd/sky130RTLDesignAndSynthesisWorkshop/verilog_files`
  - `yosys`
  - `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
  - `read_verilog dff_async_set.v`
  - `synth -top dff_async_set`
  - `dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
  - `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
  - `show` 

<img width="922" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/87e93a5e-c904-4eca-b3a7-4657c8f8f0cc">

**D Flip-Flop with Synchronous Reset**
+ **Simulation**
   - `cd vsd/sky130RTLDesignAndSynthesisWorkshop/verilog_files`
   - `iverilog dff_syncres.v tb_dff_syncres.v`
   - `./a.out`
   - `gtkwave tb_dff_syncres.vcd`
 
![Screenshot from 2023-08-30 21-55-11](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/624478c8-d3cf-4731-a0b2-c03e8882c2e5)

   <img width="500" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/472e9a2d-bb95-437d-b790-cfe72294ad07">
  

+ **Synthesis**
  - `cd vsd/sky130RTLDesignAndSynthesisWorkshop/verilog_files`
  - `yosys`
  - `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
  - `read_verilog dff_syncres.v`
  - `synth -top dff_syncres`
  - `dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib `
  - `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
  - `show`

<img width="925" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/ff5b11e7-11a8-40c9-9e08-e090eeb0f547">

<details>
<summary> Interesting Optimisations </summary>	

+ `gvim mult_2.v`

 <img width="434" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/d37ce39a-16c6-428a-a6be-ef6a5fc3c2aa">

+ `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `read_verilog mult_2.v`
+ `synth -top mul2`

 <img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/05011316-bfc4-41c3-8e87-46855d117243">

+ `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `show`

 <img width="305" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/fb4d176c-0d06-43e3-bad9-7945e02c2889">

+ `write_verilog -noattr mul2_netlist.v`
+ `!gvim mul2_netlist.v`
  
 <img width="436" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/9a0eca57-0656-4cb3-99f0-ad7a0d0f356e">

+ `gvim mult_8.v`

  <img width="443" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/3f9fa46f-56b9-43bf-8d46-325d75f76a95">

+ `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib  `
+ `read_verilog mult_8.v`
+ `synth -top mult8`

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/13359e0d-0676-4313-b791-3992655ee4f7">

+ `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `show`

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/4c8b811f-b793-45fa-a8e7-a65663ef3f74">

+ `write_verilog -noattr mult8_netlist.v`
+ `!gvim mult8_netlist.v`

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/37c89aea-497d-4e0d-99c5-c46dffd63b7d">

