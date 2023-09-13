## 1 A Tour of Computer Systems
### 1.2 Programs Are Translated by Other Programs into Different Forms

The hello program begins life as a high-level C program because it can be read and understood by human beings in that form. However, in order to run hello. c on the system, the individual C statements must be translated by other programs into a sequence of low-level machine-language instructions. These instructions are then packaged in a form called an executable object program and stored as a binary disk file. Object programs are also referred to as executable object files.

On a Unix system, the translation from source file to object file is performed by a compiler driver
![[comlilation_system.png]]

Preprocessing phase. The preprocessor (cpp) modifies the original C program according to directives that begin with the '#' character. For example, the #include <stdio.h> command in line 1 of hello.c tells the preprocessor to read the contents of the system header file stdio. h and insert it directly into the program text. The result is another C program, typically with the . i suffix.

Compilation phase. The compiler (cc1) translates the text file hello. i into the text file hello.s, which contains an assembly-language program. This program includes the following definition of function main:
![[hello.s汇编.png | 400]]
Assembly phase. Next, the assembler (as) translates hello.s into machine-language instructions, packages them in a form known as a relocatable object program, and stores the result in the object file hello.. This file is a binary file containing 17 bytes to encode the instructions for function main. If we were to view hello.o with a text editor, it would appear to be gibberish.

Linking phase. Notice that our hello program calls the printf function, which is part of the standard C library provided by every C compiler. The printf function resides in a separate precompiled object file called printf. o, which must somehow be merged with our hello. o program. The linker (1d) handles this merging. The result is the hello file, which is an executable object file (or simply executable) that is ready to be loaded into memory and executed by the svstem.

### 1.3 It Pays to Understand How Compilation Systems Work

### 1.4 Processors Read and Interpret Instructions Stored in Memory

#### Hardware Organization of a System
![[hardware_organization.png]]
Buses
Running throughout the system is a collection of electrical conduits called buses that carry bytes of information back and forth between the components. Buses are typically designed to transfer fixed-size chunks of bytes known as words. The number of bytes in a word (the word size) is a fundamental system parameter that varies across systems. Most machines today have word sizes of either 4 bytes (32 bits) or 8 bytes (64 bits). In this book, we do not assume any fixed definition of word size. Instead, we will specify what we mean by a "word" in any context that requires this to be defined.

I/O Devices
Input/output (I/O) devices are the system’s connection to the external world. Our
example system has four I/O devices: a keyboard and mouse for user input, a
display for user output, and a disk drive (or simply disk) for long-term storage of
data and programs. Initially, the executable hello program resides on the disk.
Each I/O device is connected to the I/O bus by either a controller or an adapter.
The distinction between the two is mainly one of packaging. Controllers are chip
sets in the device itself or on the system’s main printed circuit board (often called
the motherboard). An adapter is a card that plugs into a slot on the motherboard.
Regardless, the purpose of each is to transfer information back and forth between
the I/O bus and an I/O device.

Main Memory
The main memory is a temporary storage device that holds both a program and
the data it manipulates while the processor is executing the program. Physically,
main memory consists of a collection of dynamic random access memory (DRAM)
chips. Logically, memory is organized as a linear array of bytes, each with its own
unique address (array index) starting at zero. In general, each of the machine
instructions that constitute a program can consist of a variable number of bytes.
The sizes of data items that correspond to C program variables vary according
to type. For example, on an x86-64 machine running Linux, data of type short
require 2 bytes, types int and float 4 bytes, and types long and double 8 bytes.

Processor
The central processing unit (CPU), or simply processor, is the engine that interprets (or executes) instructions stored in main memory. At its core is a word-size
storage device (or register) called the program counter (PC). At any point in time,
the PC points at (contains the address of) some machine-language instruction in
main memory.  

From the time that power is applied to the system until the time that the
power is shut off, a processor repeatedly executes the instruction pointed at by the
program counter and updates the program counter to point to the next instruction.
A processor appears to operate according to a very simple instruction execution
model, defined by itsinstruction set architecture. In this model, instructions execute
in strict sequence, and executing a single instruction involves performing a series
of steps. The processor reads the instruction from memory pointed at by the
program counter (PC), interprets the bits in the instruction, performs some simple
operation dictated by the instruction, and then updates the PC to point to the next
instruction, which may or may not be contiguous in memory to the instruction that
was just executed.
There are only a few of these simple operations, and they revolve around
main memory, the register file, and the arithmetic/logic unit (ALU). The register
file is a small storage device that consists of a collection of word-size registers, each
with its own unique name. The ALU computes new data and address values. Here
are some examples of the simple operations that the CPU might carry out at the
request of an instruction:

. Load: Copy a byte or a word from main memory into a register, overwriting
the previous contents of the register.
. Store: Copy a byte or a word from a register to a location in main memory,
overwriting the previous contents of that location.
. Operate: Copy the contents of two registers to the ALU, perform an arithmetic
operation on the two words, and store the result in a register, overwriting the
previous contents of that register.
. Jump: Extract a word from the instruction itself and copy that word into the
program counter (PC), overwriting the previous value of the PC.

#### Running the hello Program
....

### 1.5 Caches Matter
![[缓存.png]]
### 1.6 Storage Devices Form a Hierarchy
memory hierarchy
![[memory_hierarchy.png]]
###  1.7 The Operating System Manages the Hardware
Files are abstractions for I/O devices, virtual memory is an abstraction for both the main memory and disk I/O devices, and processes are abstractions for the processor, main memory, and I/O devices. 
![[abstractrion_os.png]]
Processes
When a program such as hello runs on a modern system, the operating system
provides the illusion that the program is the only one running on the system. The
program appears to have exclusive use of both the processor, main memory, and
I/O devices. The processor appears to execute the instructions in the program, one
after the other, without interruption. And the code and data of the program appear
to be the only objects in the system’s memory. These illusions are provided by the
notion of a process, one of the most important and successful ideas in computer
science. (each process appears to have exclusive use of the hardware.)

A single CPU can appear to execute multiple processes concurrently by having the
processor switch among them. The operating system performs this interleaving
with a mechanism known as context switching. 

The operating system keeps track of all the state information that the process
needs in order to run. This state, which is known as the context, includes information such as the current values of the PC, the register file, and the contents of main
memory. At any point in time, a uniprocessor system can only execute the code
for a single process. When the operating system decides to transfer control from
the current process to some new process, it performs a context switch by saving
the context of the current process, restoring the context of the new process, and then passing control to the new process. The new process picks up exactly where
it left off.
![[context_switching.png]]
As Figure 1.12 indicates, the transition from one process to another is managed by the operating system kernel. The kernel is the portion of the operating
system code that is always resident in memory. When an application program
requires some action by the operating system, such as to read or write a file, it
executes a special system call instruction, transferring control to the kernel. The
kernel then performs the requested operation and returns back to the application
program. Note that the kernel is not a separate process. Instead, it is a collection
of code and data structures that the system uses to manage all the processes.

Threads
each running in the context of the process and sharing the same code and global
data.

Virtual Memory
In Linux, the topmost region of the address space is reserved for code and data
in the operating system that is common to all processes. The lower region of the
address space holds the code and data defined by the user’s process. Note that
addresses in the figure increase from the bottom to the top.
![[virtual_addr_space.png]]

For virtual memory to work, a sophisticated interaction is required between
the hardware and the operating system software, including a hardware translation
of every address generated by the processor. The basic idea is to store the contents
of a process’s virtual memory on disk and then use the main memory as a cache
for the disk. Chapter 9 explains how this works and why it is so important to the
operation of modern systems.  (没懂什么意思？？？)

Files
A file is a sequence of bytes, nothing more and nothing less. Every I/O device,
including disks, keyboards, displays, and even networks, is modeled as a file. All
input and output in the system is performed by reading and writing files, using a
small set of system calls known as Unix I/O.

### 1.8 Systems Communicate with Other Systems Using Networks
From the point of view of an individual system, the network can be viewed as just another I/O device.
![[网络也可以看成IOdevice.png]]

The Importance of Abstractions in Computer Systems
![[Some abstractionsprovided by a computersystem..png]]

## 2 Representing and Manipulating Information
### 2.1 Information Storage




