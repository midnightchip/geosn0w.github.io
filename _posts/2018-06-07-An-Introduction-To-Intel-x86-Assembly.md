---
layout: post
title: An introduction to x86 Intel Assembly For Reverse Engineering
---

When I first started Reverse Engineering, I was looking into something to begin with. I eventually decided to start with understanding assembly because after all, that's the best you can get when the source code isn't publicly available, unless you find pleasure in reading 1s and 0s or HEX dumps. A few decades ago, a lot of software used to be written in assembly language specific to the CPUs at the time. I remember writing assembly code for the 6502 CPU back on the Commodore 64 because sometimes, the BASIC was just too slow. It wasn't really fun. Nowadays, high-level programming languages are way better in terms of speed and ease of use, however, the assembly language is still actively used where very good control over the hardware has to be achieved and for reverse engineering.

Reverse Engineering is, basically, just a better understand of computers and how they work. You don't essentially learn how to hack or how to reverse engineer, you just learn how computer work. When I first started, I needed to chose a platform because Reverse Engineering can involve different stepts depending on the target CPU. I started with Intel x86 (IA-32). This is what I am going to cover in this write-up, and hopefully this will help beginners understand the basics a bit better.

### Beginning with the beginning

A very important thing you need to keep in mind when you learn how to read and interpret assembly is that each CPU has a different set of instructions which interact in different way with one another. Even the mnemonics are different from a CPU architecture to another. For example, the x86 Intel CPU has different instructions compared to the armv7 CPU. While some mnemonics may match, some of them are present on architecture but not in the others. Usually, the reference manuals contain a great deal of details about each instruction.

### CPU Registers

Before starting your journey onto the IA-32 assembly, it's a very good idea to learn more about the architecture itself. The CPU has what is called "registers". They are pretty much a very fast but limited memory on the microprocessor. The microprocessor can access these registers and retrieve the data way faster than it could do from RAM. The problem is, there are only a few of them and some registers are not general purpose, meaning they can't really be used for everything. What can't be fitted in the registers, lies in the RAM.

The IA-32 CPUs have 8 general purpose 32-Bit registers. Of course, these CPUs have way more registers than that, but only eight of them are truly general purpose. These registers are very important and you will find them scattered all over the place in the assembly output of any binary disassembler. A list of the available registers and a brief description is provided below.

EAX, EBX, ECX and EDX are general purpose registers usable for various operations. ECX is sometimes used as a counter by repetitive instructions.

EBP and ESP are the Stack Base Pointer and the Stack Pointer. Together, these two create the Stack Frame.

EDI and ESI are the Destination Index and Source Index registers and are used when dealing with the memory copying.

The preceding "E" in the register name is standing for "extended". That is because the name is carried over from the 16-bit architecture where the registers were, as you may have figured out, 16-bit wide. 

### CPU Flags

Aside from the registers, the CPU also has flags which are used during various operations. The flags are contained in the big EFLAGS special register. These flags are used for various purposes and they can manage the processor modes, and states and they can also store the logical state and are heavily used by conditional jump instructions. For example, the JNE instruction which stands for Jump If Not Equal will always use the ZF (Zero Flag) set by a previous CMP (compare) instruction. A CMP instruction takes two operands and subtracts one from the other. If they are equal, the Zero Flag will be set so the following JNE instruction will see that and skip the branch. At the same time, a JE (Jump if Equal) would jump if the flag is set.

### Arithmetics on the IA-32

The IA-32 instruction set contains a six instructions for arithmetic purposes. 

ADD  - Used: ADD Operand1, Operand2 - The result is stored in Operand1

SUB  - Used: SUB Operand1, Operant2 - The result is stored in Operand1

DIV  - Used: DIV Operand            - Divides the 64-bit value from EDX:EAX by the unsigned operand specified.

MUL  - Used: MUL Operand            - Multiplies the Unsigned operand specified, by EAX and stores the result as 64-bit in EDX:EAX

IDIV - Used: IDIV Operand           - Divides the 64-bit value from EDX:EAX by the signed operand specified.

IMUL - Used:                        - Multiplies the signed operand specified, by EAX and stores the result as 64-bit in EDX:EAX

### Example

In this example you can see the CMP, JNE and some general purpose registers used in a block of code that resembles the if conditions in a high-level programming language.

```asm
ADD EAX, 12 ; We add 12 to the EAX register's value
CMP EAX, 0xC ; We compare EAX with the hex representation of decimal 12
JNE 0x1001FFee1 ; If the comparison results that EAX = 12, the ZF is set and the JNE won't proceed to the address specified.
MOV EDI, [ECX+0x4b2] ; Move whatever is at ECX + 0x4b2 offset into the EDI register
JMP 0x1001DDe1 ; Unconditional jump to that address.
```

The uncondtional jump would not have the chance to be executed if the above JNE executed. Since the comparision on which the JNE instruction relies resulted in the two values being equal, the ZF (Zero Flag) is set and the Jump If Not Zero (JNZ) is skipped. The next instruction after the conditional jump is executed until the JMP which is an unconditional jump. This means that the jump will happen no matter what, as long as the CPU can reach the execution of this instruction.
