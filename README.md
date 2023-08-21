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
