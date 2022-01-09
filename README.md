# ＲＩＳＣ－Ｖ　ｂａｓｅｄ　ＭＹＴＨ

> 🗒️ Notes on RISC-V based Microprocessor for you in Thirty Hours (MYTH) of the [VLSI Hardware Design Program]( https://www.vlsisystemdesign.com/).

# *Contents*
------------

# Introduction to RISC-V and ISA
### RISC-V
> RISC-V is an open standard Instruction Set Architecture (ISA) based on established reduced instruction set computer (RISC) principles. 

🔬 The RISC-V has been designed to support extensive customization and specialization which can be extended by the end user, but not be redefined. The different instructions included in RISC-V are listed below:

- Pseudo instructions
- Base integer instruction (RV64I, RV32I)
- Multiply extension (RV64M)
- Single and double floating point instruction (RV64F, RV64D)
- Application binary instruction
- Memory allocation and stack pointer

🖍️ Also it is important to know that each base integer instruction set is characterized by:

- Width of the integer registers (XLEN)
- Corresponding size of the address space
- Number of integer registers (32 in RISC-V)

*RISC-V has extensive documentation and clear definitions easy to follow that can be found in their main [website](https://riscv.org/).*

⚙️ To work with RISC-V from a system position, it is necessary to go through the compilation process related to the chosen programming language. For this practice, C, Assembly and HDL will be used depending on the contextual approach desired for the lab. Some flows, however, are widely used and can be shortly described.

### GNU compiler toolchain
> The GNU Toolchain is a set of programming tools and settings aimed for programmers to make and compile their code into programs or libraries.

As their main website](https://gcc.gnu.org/) declares: 
> "*The GNU Compiler Collection includes front ends for C, C++, Objective-C, Fortran, Ada, Go, and D, as well as libraries for these languages (libstdc++,...). GCC was originally written as the compiler for the GNU operating system. The GNU system was developed to be 100% free software, free in the sense that it respects the user's freedom.*"

So it is flexible and constatly modified to meet the active community desires.

For interacting with RISC-V design a GNU Toolchain available is the [RISC-V GNU Compiler Toolchain](https://github.com/riscv-collab/riscv-gnu-toolchain). It supports two build modes a generic ELF/Newlib toolchain and a more sophisticated Linux-ELF/glibc toolchain that aids the compilation process of RISC-V's functional requirements. 

> The build [script](https://github.com/kunalg123/riscv_workshop_collaterals/blob/master/run.sh) has all needed tools for this practice.

The idea behind its use is to go through the toolchain steps to translate our human readable program to be machine understandable for the processor engine work its logic. The core steps of GCC can be simplified as follows:

- Preprocessor - Process source code before compilation. Usually it handles:
    - Removal of Comments
    - Expansion of Macros
    - Expansion of the included files

- Compiler - Takes the precossed file and converts to intermediate compiled output, often Assemby code.

*By using the “-S” flag with gcc we can convert the preprocessed C source code into assembly language without creating an object file.*

- Assembler - Takes the object file delivered by the *Compiler* and converts to relocatable machine code to be consumed by the *Linker*.

It deals with the problem of machines understand only binary language, by converting the intermidiate compiled code into machine machine understandable code.
One of its functions leaves the addresses of any calls to external functions in the assembly code as undefined to be filled in later by the *Linker*.

> By using the “-c” flag in gcc we can convert the assembly code into machine-level code:

`gcc -c HelloWorld.c -o HelloWorld.o`

- *Linker*: Makes the last relations left by the *Assembler* for a fully functional machine code.

This is the final phase in which all the linking of function calls with their definitions are done. Since the *Assembler* left the address of all the external functions, the *Linker* knows where all these functions are implemented and can fill in these addresses with the actual definitions. Aditionally, it also optimizes the program with routines to be follows at running stage. 

#### Hardware Abstraction Layer
> To process the code in an abstraction of the RISC-V core, some additional steps after *Assembler* phase will be branched and external tools will be used for testing

- *RTL implementation* -> where hardware is implemented using a RTL descriptive language. (We are going to use TL verilog !)
- *Synthesis* -> This RTL code is synthesised into a netlist, which contains gates and circuit description.
- *Physical Design implementation* -> Implementation of the netlist into GDS.

## Fundamental Concepts
### Integer number representation
> The representation of integer numbers refers to how the computer stores or represents a number in memory, which is made by the translation of the base numbers in base 2 or bit systems.

#### 64-bit Signed and Unsigned Number Systems 
> A number in CPU may be represented in two ways: signed or unsigned. A variable number is signed if it can represent both positive and negative values; and unsigned when it can only represent non-negative numbers. 

*It is important to declare the right data type when declaring integers, not only to prevent reason of inequality, but also integer overflow*

### Introduction To Application Binary Interface
> The ABI is a *system call interface* used by applications to access registers specific to architecture. 
An *Application Binary Interface* (*ABI*) is an interface between two binary programs. Usually between a system and a user level program. It defines a calling convention to low level data structures and computational routines. 
In other words, an *Application Binary Interface* (*ABI*) is an interface between two binary programs which defines a calling convention to low level data structures and computational routines. 
For the context of RISC-V, the ABI conventions can be consulted [here](https://riscv.org/wp-content/uploads/2015/01/riscv-calling.pdf).

### Memory Allocation For Double Words and Instructions

#### Useful Commands Table
> More commands can be found [here](https://www.sifive.com/blog/all-aboard-part-1-compiler-args). 

| Purpose | Command | Notes |
| --- | --- | --- |
| Compile C to Object file | ```riscv64-unknown-elf-gcc <-01/-Ofast> -mabi=lp64 -march=rv64i -o <object filename.o> <filename.c>``` | |
|view assembly code use the below command| ```riscv64-unknown-elf-objdump <object file> -d <object filename.o>``` ||
| Run the Object file | ```spike pk <object filename.o>``` ||
| Debug the Object file | ```spike -d pk <object filename.o> ``` ||
|  we can decide the pc from where the debug starts | until pc 0 < pc of your choice> ||
| thereafter any register content can be viewed using the command | reg <core> <register name> ||

#### Practice
> Problem: Write a C program that outputs the sum of the first N natural numbers from 1 to N to be run on a RISC-V CPU with function call.

```
#include <stdio.h>

extern int load(int x, int y); 

int main() {
	  int r = 0;
    int c = 2;
    r = load(0x0, c+1);
    printf("Sum of number from 1 to %d is %d\n", c, r); 
}
```
Code for *load.S*:
```
.section .text
.global load
.type load, @function

load:
	add	a4, a0, zero //a4 or "r" = 0 
	add	a2, a0, a1 //"3" in register a2, after a1 is loaded with 3
	add	a3, a0, zero
	
loop:
	add	 a4, a3, a4 
	addi a3, a3, 1 //add 1
	blt	 a3, a2, loop 
	add	 a0, a4, zero //store result "r" in a0
	ret
```  
To go through the steps to make it run on a RISC-V CPU we can use this [script](https://github.com/kunalg123/riscv_workshop_collaterals/blob/master/labs/rv32im.sh) for building and working with HDL steps. Additional lab codes and supporting files can be found in its [source directory](https://github.com/kunalg123/riscv_workshop_collaterals).
 
# Digital Logic with TL-Verilog and Makerchip
> Tools and concepts for designing a RISC-V core.

### Tools
#### [TL-Verilog](https://www.redwoodeda.com/tl-verilog)
> As stated in their [main website](http://www.tl-x.org/), TL-X is a set of HDL (Hardware Description Language) features defined as extensions to existing HDL languages, including Verilog (as "TL-Verilog"), VHDL (as "TL-VHDL"), and SystemC (as "TL-C"). 

#### [Makerchip](https://www.makerchip.com/)
> Makerchip is an online platform for open access to tools and resources with the mission of aiding and simplify the entire process of circuit design.

*Thanks [@stevehoover](https://github.com/stevehoover) and [Redwood Team](https://www.redwoodeda.com/about)! :]*

## Key Concepts and labs
### What is Digital Logic?
> *"...picture yourself making your way through all the rams and
roms and approaching this shining jewel, a chorus of voices
and a blast of trumpets in the background heralding the great
seat of all-knowing logic."* - James W. Heisig

The book [Remembering the Kanji vol.1](https://www.amazon.com.br/Remembering-Kanji-Complete-Japanese-Characters/dp/0824835921) by James W. Heisig defines *logic* (理) as *"something like a central jewel in a computer, like the jewels in old clocks that keep them running smoothly."*. With that in mind, *logic*, for computers, can be summerized as **fundamental operations and structures upon which computer systems are built.** 

The digital part of computer logic is called *gates* and is responsible for performing some of the operations in these systems.

### Logic Gates
> A **logic gate** is an idealized model of computation using a *Boolean function* which performs *logic operations* taking one or more binary inputs and producing a single binary output.

There are many logic gates, but common ones are:

![image](https://github.com/nutc4k3/RISCV-Notes/blob/8c98399f3d04a15d84b3617f361965869cdb8870/images/basic_logic_gates.png)
	
#### Inverter Lab
![image](https://github.com/nutc4k3/RISCV-Notes/blob/8c98399f3d04a15d84b3617f361965869cdb8870/images/inverter.png)

#### 2-Input Logic Gates Lab
![image](https://github.com/nutc4k3/RISCV-Notes/blob/8c98399f3d04a15d84b3617f361965869cdb8870/images/logic_gates.png)

### Combinational Logic
> A *combinational circuit* can be seen as a digital logic circuit in which the output depends on a combination of inputs at their present values. 

#### Combinational Calculator Lab
![image](https://github.com/nutc4k3/RISCV-Notes/blob/8c98399f3d04a15d84b3617f361965869cdb8870/images/comb_calc.png)

### Sequential Logic
> Further from combinational logic, sequential logic outputs depends not only on the present value of its input signals but on the sequence of past inputs.

#### Free running counter	
![image](https://github.com/nutc4k3/RISCV-Notes/blob/8c98399f3d04a15d84b3617f361965869cdb8870/images/free_running_counter.png)	

#### Sequential Calculator Lab
![image](https://github.com/nutc4k3/RISCV-Notes/blob/8c98399f3d04a15d84b3617f361965869cdb8870/images/seq_calc.png)
	
### Pipelined Logic
> On top of sequential and combination logic, pipelined logic allows overlapping execution of multiple operations within the same circuit by processing the elements in different defined stages.

#### Pipelined 2-Cycle Calculator Lab
![image](https://github.com/nutc4k3/RISCV-Notes/blob/8c98399f3d04a15d84b3617f361965869cdb8870/images/pipelined_calc.png)
	
### Validity
> To validate the output of a operation, a “when” scope `?$valid` can be used.

#### 2-Cycle Calculator with Validity
![image](https://github.com/nutc4k3/RISCV-Notes/blob/8c98399f3d04a15d84b3617f361965869cdb8870/images/pipe_calc_w_validity.png)
	
#### Calculator with Memory
![image](https://github.com/nutc4k3/RISCV-Notes/blob/8c98399f3d04a15d84b3617f361965869cdb8870/images/calc_mem.png)
	
### Simplified Table

**To be added**










