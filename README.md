# PES_ASIC_CLASS
This Repository Guides you to complete ASIC flow from scratch (GUIDE : Kunal Ghosh)

## The COURSE files are present under those respective dau folders 

Update
```
sudo apt update
sudo apt upgrade
```

## Install the Prerequisites(for ubuntu)
```
chmod +x run.sh
./run_ubuntu.sh
```
The installed contents will be available at ~/riscv_toolchain

# Introduction
#### Flow : HLL -> ALP -> Binary -> (HDL) -> GDS
- HLL -> High level language (c , c++) 
- ALP -> Assembly level program
- HDL -> Hardware Description Language
- GDS -> Graphic Data System (layout)

###### The Hardware needs to understand what the Application software is saying it to do.This relation can be eshtablished by System Software

____System Software____
- OS : Operating System : Handles IO, memory allocation, Low level system function
- Compiler : Convert the input to hardware dependent instruction
- Assembler : Convert the instructions provided by compiler to Binary format
- HDL : A program that understands the Binary pattern and map it to a netlist
- GDS : Layout

# COURSE 
<details>
<summary>DAY 1</summary>
<br>


## 1. Create a simple C program That calculates sum from 1 to N -> sum1toN.c

____Compile it using C compiler____
```
gcc sum1toN.c -o 1toN.o
./1toN.o
```
-o allows you to name your output file


____compile using riscv compieler and view the output____
```
riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o 1toN.o sum1toN.c
spike pk 1toN.o
```
