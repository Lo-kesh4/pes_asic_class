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

</details>
</details>
</details>
</details>
</details>
</details>

<details>
<summary> DAY 5: Introduction to Optimisations </summary>
  
<details>
<summary> Combinational Optimisation </summary>
	
+ Combinational logic refers to logic circuits where the outputs depend only on the current inputs and not on any previous states.
+ Combinational optimization is a field of study in computer science and operations research that focuses on finding the best possible solution from a finite set of options for problems that involve discrete variables and have no inherent notion of time.
+ Optimising the combinational logic circuit is squeezing the logic to get the most optimized digital design so that the circuit finally is area and power efficient.
+ Techniques for Optimisations:
  - **Constant propagation** is an optimization technique used in compiler design and digital circuit synthesis to improve the efficiency of code and circuit implementations by replacing variables or expressions with their constant values where applicable.
  - **Boolean logic optimization**, also known as logic minimization or Boolean function simplification, is a process in digital design that aims to simplify Boolean expressions or logic circuits by reducing the number of terms, literals, and gates required to implement a given logical function.

</details>

<details>
<summary> Sequential Logic Optimisations </summary>	

+ Sequential logic optimizations involve improving the efficiency, performance, and resource utilization of digital circuits that include memory elements like flip-flops and latches.
+ Optimizing sequential logic is crucial in ensuring that digital circuits meet timing requirements, consume minimal power, and occupy the least possible area while maintaining correct functionality.
+ Optimisation methods:
  - **Sequential constant propagation**, also known as constant propagation across sequential elements, is an optimization technique used in digital design to identify and propagate constant values through sequential logic elements like flip-flops and registers. This technique aims to replace variable values with their known constant values at various stages of the logic circuit, optimizing the design for better performance and resource utilization.
  - **State optimization**, also known as state minimization or state reduction, is an optimization technique used in digital design to reduce the number of states in finite state machines (FSMs) while preserving the original functionality.
  - **Sequential logic cloning**, also known as retiming-based cloning or register cloning, is a technique used in digital design to improve the performance of a circuit by duplicating or cloning existing registers (flip-flops) and introducing additional pipeline stages. This technique aims to balance the critical paths within a circuit and reduce its overall clock period, leading to improved timing performance and better overall efficiency.
  - **Retiming** is an optimization technique used in digital design to improve the performance of a circuit by repositioning registers (flip-flops) along its paths to balance the timing and reduce the critical path delay. The primary goal of retiming is to achieve a shorter clock period without changing the functionality of the circuit.
 
</details>

## Combinational Logic Optimisations

<details>
<summary> opt_check </summary>	
	
+ `gvim opt_check.v`

  <img width="500" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/dad0961e-10d4-4a0c-9991-0ad6daea169f">

+ `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `read_verilog opt_check.v`
+ `synth -top opt_check`
+ `opt_clean -purge`
+ `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `show`

  <img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/44d6a65e-1405-49b6-a569-66a7c976308c">

  <img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/8861528b-55be-45e4-952e-c0600c811685">

</details>

<details>
<summary> opt_check2 </summary>	
	
+ `gvim opt_check2.v`

  <img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/d957a7b5-fb8a-4e59-a9d3-cb1730a7dd25">
  
+ `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `read_verilog opt_check2.v`
+ `synth -top opt_check2`
+ `opt_clean -purge`
+ `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `show`

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/c85299a9-10df-4b40-8f0f-f19ead681ad3">

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/d0b4fb18-71ff-4aa6-92b9-49eac8dd889b">

</details>

<details>
<summary> opt_check3 </summary>	
	
+ `gvim opt_check3.v`

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/3183db65-f77d-443a-9814-dc776c3c0990">

+ `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `read_verilog opt_check3.v`
+ `synth -top opt_check3`
+ `opt_clean -purge`
+ `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `show`

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/13709061-55c2-43ca-b798-c8398e1c7fdb">

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/2c885b4d-c274-4bae-abd0-15853f864f62">

</details>

<details>
<summary> opt_check4 </summary>
	
+ `gvim opt_check4.v`

 <img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/75c65195-8f6b-416e-8074-306a46263746">

+ `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `read_verilog opt_check4.v`
+ `synth -top opt_check4`
+ `opt_clean -purge`
+ `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `show`

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/a1acd352-c271-4330-9fd8-6aafe72b8f11">

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/498cf442-ec8e-468e-a310-d1f93b93ce1a">

</details>

<details>
<summary> multiple_module_opt </summary>
	
+ `gvim multiple_module_opt.v`

 <img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/ad570bd8-44b5-4408-8715-02f1c5d15a29">

+ `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `read_verilog multiple_module_opt.v`
+ `synth -top multiple_module_opt`
+ `opt_clean -purge`
+ `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `show`
 
<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/4c3fd4bc-c599-41cf-af42-c07280dcca11">

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/1344d22d-51f5-439e-bc34-96b2a742474e">

</details>

## Sequential Logic Optimisations

<details>
<summary> dff_const1 </summary>	

+ `gvim dff_const1.v`

<img width="331" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/abec2938-5f2c-4cd5-b369-103c1b09f098">

**Simulation**

+ `iverilog dff_const1.v tb_dff_const1.v`
+ `/a.out`
+ `gtkwave tb_dff_const1.vcd`

![Screenshot from 2023-09-12 11-06-10](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/9491f3ba-124d-420d-8c9e-6e6d9e211991)

<img width="503" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/51301173-fdbd-476c-842e-2d08078f020d">

**Synthesis**

+ `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `read_verilog dff_const1.v`
+ `synth -top dff_const1`
+ `dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib `
+ `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `show`

<img width="194" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/c542c8b4-3624-4a22-92c5-35a4b1458c35">

<img width="925" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/fa1d8b2f-431e-4836-8a75-8c2bd3ce326e">

</details>

<details>
<summary> dff_const2 </summary>	

+ `gvim dff_const2.v`

<img width="355" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/e0e6e4da-0429-49db-b687-d99ab365ed17">

**Simulation**

+  `iverilog dff_const2.v tb_dff_const2.v`
+ `/a.out`
+ `gtkwave tb_dff_const2.vcd`

![Screenshot from 2023-09-12 11-10-58](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/59803a76-d1d5-4e36-8a97-ca554fcfad41)

 <img width="500" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/a90d628f-dd7d-4ae6-8b4d-072b6a9960b9">

 **Synthesis**
 
+ `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `read_verilog dff_const2.v`
+ `synth -top dff_const2`
+ `dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib `
+ `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `show`

 <img width="206" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/d4859923-36a2-4cf2-bcef-6befaf718913">

<img width="305" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/8e3503dd-d315-426f-9a3e-bd487014600a">

</details>

<details>
<summary> dff_const3 </summary>

+ `gvim dff_const3.v`

 <img width="272" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/3ce28559-0063-4ef1-9d7e-a3af839dd7e3">

**Simulation**

+ `iverilog dff_const3.v tb_dff_const3.v`
+ `/a.out`
+ `gtkwave tb_dff_const3.vcd`

![Screenshot from 2023-09-12 11-13-30](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/ffce898d-e6ea-475d-877d-d8592ab68ac5)

<img width="502" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/aed6c933-5c06-4687-ba9e-9c782626c030">

**Synthesis**

+ `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `read_verilog dff_const3.v`
+ `synth -top dff_const3`
+ `dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib `
+ `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `show`

<img width="197" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/7cc2308b-b988-4ba9-965f-06abf2472e2b">

<img width="922" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/bdea6f33-d357-47f4-a9fd-cc655dcce869">

</details>

<details>
<summary> dff_const4 </summary>	

+ `gvim dff_const4.v`

<img width="311" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/bc5661c4-50c6-4ccf-8aab-a5ace3ccfa14">

**Simulation**

+ `iverilog dff_const4.v tb_dff_const4.v`
+ `/a.out`
+ `gtkwave tb_dff_const4.vcd`

![Screenshot from 2023-09-12 11-15-23](https://github.com/Lo-kesh4/pes_asic_class/assets/131575546/3057ad4a-be0c-4e23-9023-653cb5d38af7)

<img width="500" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/1a9ee230-2ad6-4c92-8e37-0a753832180f">

**Synthesis**

+ `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `read_verilog dff_const4.v`
+ `synth -top dff_const4`
+ `dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib `
+ `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `show`

<img width="193" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/a152102c-a880-499d-98e6-f2d26201ea85">

<img width="306" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/4d4d33fc-11ec-4cb5-867d-2f34d892255a">

</details>

<details>
<summary> dff_const5 </summary>	

+ `gvim dff_const5.v`

<img width="251" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/af7acfca-6fb0-4b62-bb1e-d32746d0c07e">

**Simulation**

+ `iverilog dff_const4.v tb_dff_const4.v`
+ `/a.out`
+ `gtkwave tb_dff_const4.vcd`

<img width="529" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/e27a9ee5-6332-4147-8bf9-c246ca7d7996">

<img width="500" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/7eb3b371-189e-483c-80d9-37d38c062cd2">

**Synthesis**

+ `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `read_verilog dff_const4.v`
+ `synth -top dff_const4`
+ `dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib `
+ `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `show`

 <img width="205" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/e2719af7-b746-4830-b04f-87df97143f86">

<img width="923" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/8d95c49d-9fd9-4d76-bba1-b893c6f163fc">

</details>

## Sequential Optimisations for Unused Outputs
<details>
<summary> counter_opt </summary>

 + `gvim counter_opt.v`

<img width="349" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/cb5798fa-2ee9-4cf0-9372-3da9ff17bd66">

+ `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `read_verilog counter_opt.v`
+ `synth -top counter_opt`
+ `dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `show`

<img width="184" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/1c876b33-9d26-4efe-95cf-5d0814415da5">

<img width="923" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/b65d5ac8-3961-4a7b-9e8a-18bc0229f104">

</details>

<details>
<summary> counter_opt2 </summary>	

+ `gvim counter_opt2.v`

 <img width="347" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/b6262d9a-5892-4360-91fa-18f7e2aa39e7">

+ `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `read_verilog counter_opt2.v`
+ `synth -top counter_opt2`
+ `dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `show`

 <img width="200" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/14aadea7-9607-40a8-b5ed-e48eb255cd10">

<img width="923" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/78af2250-6ee8-4d94-b91b-138cb2877b1e">

</details>
</details>
<details>
<summary>DAY 6:GLS Synthesis-Simulation Mismatch and Blocking Non-blocking Statements </summary>

<details>
<summary> GLS Concepts And Flow Using Iverilog </summary>

 + **Gate Level Simualtion**
   - Gate-level simulation is a technique used in digital design and verification to validate the functionality of a digital circuit at the gate-level implementation.
   - It involves simulating the circuit using the actual logic gates and flip-flops that make up the design, as opposed to higher-level abstractions like RTL (Register Transfer Level) descriptions.
   - This type of simulation is typically performed after the logic synthesis process, where a high-level description of the design is transformed into a netlist of gates and flip-flops.
   - We perform this to verify logical correctness of the design after synthesizing it. Also ensuring the timing of the design is met.
  
<img width="608" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/6298b067-2f45-4dbc-ad25-762ac3d8be63">

+ **Synthesis-Simulation Mismatch**
  - A synthesis-simulation mismatch refers to a situation in digital design where the behavior of a circuit, as observed during simulation, doesn't match the expected or desired behavior of the circuit after it has been synthesized.
  - This discrepancy can occur due to various reasons, such as timing issues, optimization conflicts, and differences in modeling between the simulation and synthesis tools.
  - This mismatch is a critical concern in digital design because it indicates that the actual hardware implementation might not perform as expected, potentially leading to functional or timing failures in the fabricated chip.

+ **Blocking Statements**
  - Blocking statements are executed sequentially in the order they appear in the code and have an immediate effect on signal assignments.
  - Example:

  ``` v
   module BlockingExample(input A, input B, input C, output Y, output Z);
    wire temp;

    // Blocking assignment
    assign temp = A & B;

    always @(posedge C) begin
        // Blocking assignment
        Y = temp;
        Z = ~temp;
    end
   endmodule
  ```

+ **Non-Blocking Statements**
  - Non-blocking assignments are used to model concurrent signal updates, where all assignments are evaluated simultaneously and then scheduled to be updated at the end of the time step.
  - Example:
   ``` v
    module NonBlockingExample(input clock, input D, input reset, output reg Q);

    always @(posedge clock or posedge reset) begin
        if (reset)
            Q <= 0;  // Reset the flip-flop
        else
            Q <= D;  // Non-blocking assignment to update Q with D on clock edge
    end
  endmodule
   ```

+ **Caveats with Blocking Statements**
  + Blocking statements in hardware description languages like Verilog have their uses, but there are certain caveats and considerations to be aware of when working with them. Here are some important caveats associated with using blocking statements:
    - Procedural Execution: Blocking statements are executed sequentially in the order they appear within a procedural block (such as an always block). This can lead to unexpected behavior if the order of execution matters and is not well understood.
    - Lack of Parallelism: Blocking statements do not accurately represent the parallel nature of hardware. In hardware, multiple signals can update concurrently, but blocking statements model sequential behavior. As a result, using blocking statements for modeling complex concurrent logic can lead to incorrect simulations.
    - Race Conditions: When multiple blocking assignments operate on the same signal within the same procedural block, a race condition can occur. The outcome of such assignments depends on their order of execution, which might lead to inconsistent or unpredictable behavior.
    - Limited Representation of Hardware: Hardware systems are inherently concurrent and parallel, but blocking statements do not capture this aspect effectively. Using blocking assignments to model complex combinational or sequential logic can lead to models that are difficult to understand, maintain, and debug.
    - Combinatorial Loops: Incorrect use of blocking statements can lead to unintentional combinational logic loops, which can result in simulation or synthesis errors.
    - Debugging Challenges: Debugging code with many blocking assignments can be challenging, especially when trying to track down timing-related issues.
    - Not Suitable for Flip-Flops: Blocking assignments are not suitable for modeling flip-flop behavior. Non-blocking assignments (<=) are generally preferred for modeling flip-flop updates to ensure accurate representation of concurrent behavior.
    - Sequential Logic Misrepresentation: Using blocking assignments to model sequential logic might not capture the intended behavior accurately. Sequential elements like registers and flip-flops are better represented using non-blocking assignments.
    - Synthesis Implications: The behavior of blocking assignments might not translate well during synthesis, leading to potential mismatches between simulation and synthesis results.

</details>

## Labs on GLS and Synthesis-Simulation Mismatch
<details>
<summary> ternary_operator_mux </summary>	

+ `gvim teranry_operator_mux.v`

<img width="370" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/8539ab94-8f5a-4bff-8465-eb8bb6ca83b8">

**Simulation**

+ `iverilog ternary_operator_mux.v tb_ternary_operator_mux.v`
+ `./a.out`
+ `gtkwave tb_ternary_operator_mux.vcd`

<img width="626" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/893dc1c8-bc64-41bf-bb35-ee83e37b023d">

<img width="500" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/c675c505-880e-4c15-b079-3c528032c279">

**Synthesis**

+ `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `read_verilog ternary_operator_mux.v`
+ `synth -top ternary_operator_mux`
+ `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `show`

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/74fa07aa-5dc3-4f04-8fc1-babe67ded667">

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/9a4f4ebd-370f-45b5-a1c1-cd95b8e1e556">

**GLS to Gate-Level Simulation**

+ `iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v ternary_operator_mux_net.v tb_ternary_operator_mux.v`
+ `./a.out`
+ `gtkwave tb_ternary_operator_mux.vcd`

<img width="928" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/97f59e19-d561-4c1e-b8b5-46fb1eb21595">

<img width="498" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/5c4652e6-8364-4e81-9cd3-f1795f5d321a">

</details>

<details>
<summary> bad_mux </summary>	

 + `gvim bad_mux.v`

 <img width="290" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/deb388a2-8463-410b-b16e-5eaf81697d69">

**Simualtion**

+ `iverilog bad_mux.v tb_bad_mux.v`
+ `./a.out`
+ `gtkwave tb_bad_mux.vcd`

<img width="501" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/9c68b293-6f55-4742-bb06-5cdf9551a5ae">

<img width="500" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/46266c26-99ce-4a79-9e5c-9558ea15f407">

**Synthesis**

+ `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `read_verilog bad_mux.v`
+ `synth -top bad_mux`
+ `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `show`

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/a19eb270-87dd-40cc-95f8-3bd3cf02a396">

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/d15147f9-775d-44bb-93a5-d7249645d9bc">

**GLS to Gate-Level Simulation**

+ `iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v bad_mux_net.v tb_bad_mux.v`
+ `./a.out`
+ `gtkwave tb_bad_mux.vcd`

<img width="878" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/29d0b4bd-2874-4809-bf2b-728a73cccc09">
  
<img width="501" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/9d51d787-22d3-4495-95cc-c87c0ef71d17">

</details>

## Labs on Synth-Sim Mismatch for Blocking Statement

<details>
<summary> blocking_caveat </summary>	

+ `gvim blocking_caveat.v`

<img width="327" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/5827ea02-ec07-4164-9b9f-b064750ede9d">

**Simualtion**

+ `iverilog blocking_caveat.v tb_blocking_caveat.v`
+ `./a.out`
+ `gtkwave tb_blocking_caveat.vcd`

<img width="567" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/09e0265c-b794-4d3f-9e27-4403a6b9122a">

<img width="500" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/fe7f3ec3-eec1-40e7-b20d-47f6242c3619">

**Synthesis**

+ `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `read_verilog blocking_caveat.v`
+ `synth -top blocking_caveat`
+ `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
+ `show`

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/7f942ace-02b0-4421-8d20-5e7126cbb3df">

<img width="400" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/d660d89b-8a9a-43d3-9e78-ab1f7054a667">

**GLS to Gate-Level Simulation**

+ `iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v blocking_caveat_net.v tb_blocking_caveat.v`
+ `./a.out`
+ `gtkwave tb_blocking_caveat.vcd`

<img width="926" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/c541ebb5-b42d-4629-b12a-734541df3071">

<img width="502" alt="image" src="https://github.com/Veda1809/pes_asic_class/assets/142098395/d265a9a3-db89-46ed-b351-3e6ac9004cd9">

</details>

