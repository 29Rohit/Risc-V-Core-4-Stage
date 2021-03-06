# RISC-V based Microprocessor

This repository contains all the information needed to build your RISC-V pipelined core, which has support of base interger RV32I instruction format using TL-Verilog on makerchip platform.

# Table of Contents
- [Introduction to RISC-V ISA](#introduction-to-risc-v-isa)
- [Overview of GNU compiler toolchain](#overview-of-gnu-compiler-toolchain)
- [Introduction to ABI](#introduction-to-abi)
- [Digital Logic with TL-Verilog and Makerchip](#digital-logic-with-tl-verilog-and-makerchip)
  - [Combinational logic](#combinational-logic)
  - [Sequential logic](#sequential-logic)
  - [Pipelined logic](#pipelined-logic)
  - [Validity](#validity)
- [Basic RISC-V CPU micro-architecture](#basic-risc-v-cpu-micro-architecture)
  - [Fetch](#fetch)
  - [Decode](#decode)
  - [Register File Read and Write](#register-file-read-and-write)
  - [Execute](#execute)
  - [Control Logic](#control-logic)
- [Pipelined RISC-V CPU](#pipelined-risc-v-cpu)
  - [Pipelinig the CPU](#pipelining-the-cpu)
  - [Load and store instructions and memory](#load-and-store-instructions-and-memory)
  - [Completing the RISC-V CPU](#completing-the-risc-v-cpu)
- [Acknowledgements](#acknowledgements)

# Introduction to RISC-V ISA 

A RISC-V ISA is defined as a base integer ISA, which must be present in any implementation, plus optional extensions to the base ISA. Each base integer instruction set is characterized by
  1. Width of the integer registers (XLEN) 
  2. Corresponding size of the address space
  3. Number of integer registers (32 in RISC-V)

More details on RISC-V ISA can be obtained [here](https://github.com/riscv/riscv-isa-manual/releases/download/draft-20200727-8088ba4/riscv-spec.pdf).

# Overview of GNU compiler toolchain

The GNU Toolchain is a set of programming tools in Linux systems that programmers can use to make and compile their code to produce a program or library. So, how the machine code which is understandable by processer is explained below.

  * Preprocessor - Process source code before compilation. Macro definition, file inclusion or any other directive if present then are preprocessed. 
  * Compiler - Takes the input provided by preprocessor and converts to assembly code.
  * Assembler - Takes the input provided by compiler and converts to relocatable machine code.
  * Linker - Takes the input provided by Assembler and converts to Absolute machine code.

Under the risc-v toolchain, 
  * To use the risc-v gcc compiler use the below command:

    `riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o <object filename> <C filename>`

    More generic command with different options:

    `riscv64-unknown-elf-gcc <compiler option -O1 ; Ofast> <ABI specifier -lp64; -lp32; -ilp32> <architecture specifier -RV64 ; RV32> -o <object filename> <C      filename>`

    More details on compiler options can be obtained [here](https://www.sifive.com/blog/all-aboard-part-1-compiler-args)

  * To view assembly code use the below command,
    
    `riscv64-unknown-elf-objdump -d <object filename>`
    
  * To use SPIKE simualtor to run risc-v obj file use the below command,
  
    `spike pk <object filename>`
    
    To use SPIKE as debugger
    
    `spike -d pk <object Filename>` with degub command as `until pc 0 <pc of your choice>`

    To install complete risc-v toolchain locally on linux machine,
      1. [RISC-V GNU Toolchain](http://hdlexpress.com/RisKy1/How2/toolchain/toolchain.html)
      2. [RISC-V ISA SImulator - Spike](https://github.com/kunalg123/riscv_workshop_collaterals)
    
    Once done with installation add the PATH to .bashrc file for future use.

Test Case for the above commands [(Summation of 1 to 9)](codes/sum1ton.c),

  
  * To view the registers we can use command as `reg <core> <register name>`. 
  
  Below image shows how to debug the disassembled file using Spike simulator where a1,a2 register are checked before and after the instructions got executed.

![1](https://user-images.githubusercontent.com/20563301/157902045-a095ebe8-c818-4b7b-ab03-d000155a6f5b.PNG)


# Introduction to ABI

An Application Binary Interface is a set of rules enforced by the Operating System on a specific architecture. So, Linker converts relocatable machine code to absolute machine code via ABI interface specific to the architecture of machine.

So, it is system call interface used by the application program to access the registers specific to architecture. Overhere the architecture is RISC-V, so to access 32 registers of RISC-V below is the table which shows the calling convention (ABI name) given to registers for the application programmer to use.


![1](https://user-images.githubusercontent.com/20563301/157898348-d9e01341-3e50-4970-a814-e02b840e9f61.PNG)

 * Below image shows 2 sections from [load.S](codes/load.S) (one is load and other is loop).

![3](https://user-images.githubusercontent.com/20563301/157901111-7914bdd2-3af0-41e1-bc23-7d0987b0c297.PNG)


 

# Digital Logic with TL-Verilog and Makerchip

[Makerchip](https://makerchip.com/) is a free online environment for developing high-quality integrated circuits. You can code, compile, simulate, and debug Verilog designs, all from your browser. Your code, block diagrams, and waveforms are tightly integrated.

All the examples shown below are done on Makerchip IDE using TL-verilog. Also there are other tutorials present on IDE which can be found [here](https://makerchip.com/sandbox/) under Tutorials section.

## [Combinational logic](codes/Combinational_Calculator.tlv)

Starting with basic example in combinational logic is an inverter. To write the logic of inverter using TL-verilog is `$out = ! $in;`. There is no need to declare `$out` and `$in` unlike Verilog. There is also no need to assign `$in`. Random stimulus is provided, and a warning is produced. 

Below is snapshot of Combinational Calculator.

![combo calculator](https://user-images.githubusercontent.com/20563301/157898685-29a16839-d598-40d2-b9e0-d27bc795b5f3.PNG)


## [Sequential logic](codes/Sequential_Calculator.tlv)

Starting with basic example in sequential logic is Fibonacci Series with reset. To write the logic of Series using TL-Verilog is `$num[31:0] = $reset ? 1 : (>>1$num + >>2$num)`. This operator `>>?` is ahead of operator which will provide the value of that signal 1 cycle before and 2 cycle before respectively.

Below is snapshot of Sequential Calculator which remembers the last result, and uses it for the next calculation.

![seq calc op](https://user-images.githubusercontent.com/20563301/157899127-396a00fb-2ae7-4b3c-a2b1-736f4b349120.PNG)


## [Pipelined logic](codes/Cycle_Calculator.tlv)

Timing abstract powerful feature of TL-Verilog which converts a code into pipeline stages easily. Whole code under `|pipe` scope with stages defined as `@?`

Below is snapshot of 2-cycle calculator which clears the output alternatively and output of given inputs are observed at the next cycle.

![1](https://user-images.githubusercontent.com/20563301/158003722-58d3a543-3e3a-4ca1-9543-2430d21842ee.PNG)


## [Validity](codes/Cycle_Calculator_Validity.tlv)

Validity is TL-verilog means signal indicates validity of transaction and described as "when" scope else it will work as don't care. Denoted as `?$valid`. Validity provides easier debug, cleaner design, better error checking, automated clock gating.

Below is snapshot of 2-cycle calculator with validity. 

![2](https://user-images.githubusercontent.com/20563301/158003728-eaa423d4-ca0a-45d4-b43f-989d21f7759e.PNG)


# Basic RISC-V CPU micro-architecture

Designing the basic processor of 3 stages fetch, decode and execute based on RISC-V ISA.

## [Fetch](codes/Fetch.tlv)

* Program Counter (PC): Holds the address of next Instruction
* Instruction Memory (IM): Holds the set of instructions to be executed

During Fetch Stage, processor fetches the instruction from the IM pointed by address given by PC.

Below is snapshot from Makerchip IDE after performing the Fetch Stage.

![3](https://user-images.githubusercontent.com/20563301/158003733-9415de86-6f94-493a-b7bb-1c5fb14ba51f.PNG)


## [Decode](codes/Decode.tlv)

6 types of Instructions:
  * R-type - Register 
  * I-type - Immediate
  * S-type - Store
  * B-type - Branch (Conditional Jump)
  * U-type - Upper Immediate
  * J-type - Jump (Unconditional Jump)

Instruction Format includes Opcode, immediate value, source address, destination address. During Decode Stage, processor decodes the instruction based on instruction format and type of instruction.

Below is snapshot from Makerchip IDE after performing the Decode Stage.

![4](https://user-images.githubusercontent.com/20563301/158003740-658c6c1c-c06a-45ca-934e-ebfc7f4c97b4.PNG)


## [Register File Read and Write](codes/Register_File_Read.tlv)

Here the Register file is 2 read, 1 write means 2 read and 1 write operation can happen simultanously.

Inputs:
  * Read_Enable   - Enable signal to perform read operation
  * Read_Address1 - Address1 from where data has to be read 
  * Read_Address2 - Address2 from where data has to be read 
  * Write_Enable  - Enable signal to perform write operation
  * Write_Address - Address where data has to be written
  * Write_Data    - Data to be written at Write_Address

Outputs:
  * Read_Data1    - Data from Read_Address1
  * Read_Data2    - Data from Read_Address2

Below is snapshot from Makerchip IDE after performing the Register File Read followed by Register File Write.

![5](https://user-images.githubusercontent.com/20563301/158003759-7be224d9-b2c0-4d52-8d75-b6ac0da8abd4.PNG)


![6](https://user-images.githubusercontent.com/20563301/158003763-d058cc61-8445-4a05-a11a-57c47dae286b.PNG)


## [Execute](codes/ALU.tlv)

During the Execute Stage, both the operands perform the operation based on Opcode.

Below is snapshot from Makerchip IDE after performing the Execute Stage.

![7](https://user-images.githubusercontent.com/20563301/158003765-2411942d-e750-4753-87a1-9f5bc7cc504f.PNG)


## [Control Logic](codes/Branches.tlv)

During Decode Stage, branch target address is calculated and fed into PC mux. Before Execute Stage, once the operands are ready branch condition is checked.

Below is snapshot from Makerchip IDE after including the control logic for branch instructions.

![8](https://user-images.githubusercontent.com/20563301/158003771-f324a6d4-2cb8-4270-976a-ba408b72653f.PNG)

# Pipelined RISC-V CPU

Converting non-piepleined CPU to pipelined CPU using timing abstract feature of TL-Verilog. This allows easy retiming wihtout any risk of funcational bugs. More details reagrding Timing Abstract in TL-Verilog can be found in IEEE Paper ["Timing-Abstract Circuit Design in Transaction-Level Verilog" by Steven Hoover.](https://ieeexplore.ieee.org/document/8119264)

## [Pipelining the CPU](codes/Pipelining_the_CPU.tlv)

Pipelining the CPU with branches still having 3 cycle delay rest all instructions are pipelined. Pipelining the CPU in TL-Verilog can be done in following manner:
```
|<pipe-name>
    @<pipe stage>
       Instructions present in this stage
       
    @<pipe_stage>
       Instructions present in this stage
       
```

Below is snapshot of pipelined CPU with a test case of assembly program which does summation from 1 to 9 then stores to r10 of register file. In snapshot `r10 = 45`. Test case:
```
*passed = |cpu/xreg[10]>>5$value == (1+2+3+4+5+6+7+8+9);

```
![9](https://user-images.githubusercontent.com/20563301/158003776-e3cb3bd0-dc69-442a-9888-08a5220ce51a.PNG)


## [Load and store instructions and memory](codes/Load_Store.tlv)

Similar to branch, load will also have 3 cycle delay. So, added a Data Memory 1 write/read memory.

Inputs:
  * Read_Enable - Enable signal to perform read operation
  * Write_Enable - Enable signal to perform write operation
  * Address - Address specified whether to read/write from
  * Write_Data - Data to be written on Address (Store Instruction)

Output: 
  * Read_Data - Data to be read from Address (Load Instruction)

Added test case to check fucntionality of load/store. Stored the summation of 1 to 9 on address 4 of Data Memory and loaded that value from Data Memory to r17.
```
*passed = |cpu/xreg[17]>>5$value == (1+2+3+4+5+6+7+8+9);
```
Below is snapshot from Makerchip IDE after including load/store instructions.

![10](https://user-images.githubusercontent.com/20563301/158003784-7040ddce-e1aa-4174-9bb3-d2006838b4a0.PNG)

## [Completing the RISC-V CPU](codes/Final.tlv)

Added Jumps and completed Instruction Decode and ALU for all instruction present in RV32I base integer instruction set.

Below is final Snapshot of Complete Pipelined RISC-V CPU.

![11](https://user-images.githubusercontent.com/20563301/158003790-8aaa521a-eaf1-4130-b552-131b16623643.PNG)


# Acknowledgements
- [Kunal Ghosh](https://github.com/kunalg123), Co-founder, VSD Corp. Pvt. Ltd.
- [Steve Hoover](https://github.com/stevehoover), Founder, Redwood EDA
- [Shivam Potdar](https://github.com/shivampotdar), GSoC 2020 @fossi-foundation
- [Vineet Jain](https://github.com/vineetjain07), GSoC 2020 @fossi-foundation
