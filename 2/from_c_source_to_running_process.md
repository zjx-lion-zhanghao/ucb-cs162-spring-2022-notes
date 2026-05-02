# From C Source Code to a Running Process

## 1. Big Picture

This document summarizes how a C program changes from source code into a running process.

The core idea is:

> A C program is first compiled and linked into an executable file.  
> When the user runs that executable, the operating system creates a process and at least one thread.  
> The CPU then executes the thread by fetching instructions from memory using the Program Counter.

A simplified flow is:

```text
foo.c
  ↓ compile
foo.o
  ↓ link
a.out
  ↓ run with ./a.out
Operating system creates a process
  ↓
Operating system creates the main thread
  ↓
Executable code and data are loaded or mapped into memory
  ↓
The Program Counter is set to the entry point
  ↓
CPU executes instructions one by one
```

---

## 2. Source Code: `foo.c`

A C source file is a human-readable text file.

Example:

```c
#include <stdio.h>

int global_x = 10;

int main() {
    int a = 1;
    int b = 2;
    int c = a + b + global_x;

    printf("c = %d\n", c);

    return 0;
}
```

At this stage, the file is only text. The CPU cannot directly execute C language syntax such as:

```c
int main()
printf()
return
```

The CPU only understands machine instructions.

---

## 3. Compilation: From `foo.c` to `foo.o`

The compiler translates C source code into object code.

Command:

```bash
gcc -c foo.c -o foo.o
```

This produces:

```text
foo.o
```

The object file already contains machine instructions, but it is not yet a complete executable program.

For example, if the program uses `printf()`, the implementation of `printf()` comes from the C standard library, not directly from `foo.c`.

Therefore, the object file still needs to be linked with libraries.

---

## 4. Linking: From `foo.o` to `a.out`

The linker combines object files and required libraries into one executable file.

Command:

```bash
gcc foo.o -o a.out
```

Or directly:

```bash
gcc foo.c
```

By default, this generates:

```text
a.out
```

Important point:

> `a.out` is only an executable file stored on disk.  
> It is not yet a process.  
> It is not yet a thread.  
> It is not yet running.

---

## 5. What Is Inside an Executable File?

A simplified executable file contains:

```text
a.out
+-------------------------+
| ELF header              |
+-------------------------+
| text / code segment     | machine instructions
+-------------------------+
| rodata segment          | read-only data, such as string constants
+-------------------------+
| data segment            | initialized global variables
+-------------------------+
| bss segment             | uninitialized global variables
+-------------------------+
| symbol / debug info     |
+-------------------------+
```

In an introductory operating systems slide, this may be simplified as:

```text
Executable
+----------------+
| data           |
+----------------+
| instructions   |
+----------------+
```

### Instructions

Instructions are CPU-level commands.

For example, a C statement like:

```c
c = a + b + global_x;
```

may become machine instructions such as:

```asm
load
add
store
```

The CPU executes these instructions one by one.

### Data

Data includes values needed by the program, such as:

```c
int global_x = 10;
```

It also includes string constants such as:

```c
"c = %d\n"
```

---

## 6. Running the Program: `./a.out`

When the user types:

```bash
./a.out
```

the shell receives the command and asks the operating system to run the executable.

In Unix/Linux systems, this usually involves system calls such as:

```text
fork()
execve()
```

A simplified view is:

```text
shell
  ↓
asks the OS to run ./a.out
  ↓
OS creates a new process
  ↓
OS loads/maps a.out into that process
  ↓
OS starts the main thread
```

So, from the user's perspective, the process and thread are created when `./a.out` is executed.

More precisely:

> The compiler does not create the process.  
> The linker does not create the process.  
> The operating system creates the process when the executable is run.

---

## 7. What Is a Process?

A process is a running instance of a program.

The executable file is static. The process is dynamic.

For example:

```bash
./a.out
```

creates one process.

If the same executable is run three times:

```bash
./a.out
./a.out
./a.out
```

then the operating system creates three different processes from the same executable file.

A process usually contains:

```text
Process
+----------------------------------+
| Process ID (PID)                 |
| Virtual address space            |
| Code segment                     |
| Data segment                     |
| Heap                             |
| Stack                            |
| Open files                       |
| Permissions                      |
| One or more threads              |
+----------------------------------+
```

A simple analogy:

```text
Executable file = recipe
Process = one actual cooking activity based on that recipe
```

The same recipe can be used many times. The same executable can create many processes.

---

## 8. What Is a Thread?

A thread is one path of execution inside a process.

Every process has at least one thread. This first thread is usually called the main thread.

A thread has its own execution context, including:

```text
Program Counter
Registers
Execution flags
Stack
Stack pointer
```

This means:

> A thread is the state needed to continue executing one path of a program.

A process is the resource container. A thread is the actual execution path.

Simplified structure:

```text
Process
+--------------------------------+
| Shared memory                  |
|  code                          |
|  global data                   |
|  heap                          |
|                                |
| Thread 1                       |
|  - Program Counter             |
|  - registers                   |
|  - stack                       |
|                                |
| Thread 2                       |
|  - Program Counter             |
|  - registers                   |
|  - stack                       |
+--------------------------------+
```

Threads in the same process usually share:

```text
code
global variables
heap
open files
address space
```

But each thread has its own:

```text
Program Counter
registers
stack
```

---

## 9. What Is an Instruction?

An instruction is one small operation that the CPU can execute.

Examples:

```text
load a value
add two values
store a value
jump to another address
call a function
return from a function
```

Instructions are stored in memory.

Example:

```text
Memory
0x400000: load a
0x400004: load b
0x400008: add
0x40000C: store c
0x400010: call printf
```

The CPU fetches and executes instructions one by one.

---

## 10. What Is a Register?

A register is a very small and very fast storage location inside the CPU.

Registers are used to hold:

```text
temporary calculation values
addresses
function arguments
return values
the Program Counter
the Stack Pointer
status flags
```

Registers are different from memory.

```text
CPU
+----------------+
| registers      | very fast, very small
+----------------+

Memory
+----------------+
| code           |
| data           |
| heap           |
| stack          |
+----------------+
```

The CPU often works like this:

```text
load data from memory into registers
perform calculation in registers
store result back to memory
```

---

## 11. What Is the Program Counter?

The Program Counter, or PC, is a special register.

It stores:

> the memory address of the next instruction to execute.

Example:

```text
PC = 0x400008
```

This means:

> The CPU should fetch and execute the instruction stored at memory address `0x400008`.

If memory contains:

```text
0x400000: load a
0x400004: load b
0x400008: add
0x40000C: store c
```

and:

```text
PC = 0x400008
```

then the next instruction to execute is:

```text
add
```

Important distinction:

```text
PC is not the instruction itself.
PC is the address of the next instruction.
```

Also:

```text
PC = Program Counter
PC does not mean Process Counter
```

---

## 12. Loading the Executable into Memory

The executable file is stored on disk.

When the program is executed, the operating system loads or maps its segments into the process address space.

A simplified process memory layout looks like this:

```text
High Address
+-------------------------+
| stack                   |
+-------------------------+
| shared libraries        |
+-------------------------+
| heap                    |
+-------------------------+
| bss                     |
+-------------------------+
| data                    |
+-------------------------+
| rodata                  |
+-------------------------+
| text / code             |
+-------------------------+
Low Address
```

Examples:

```c
int global_x = 10;
```

This belongs to the data segment.

```c
"c = %d\n"
```

This belongs to the read-only data segment.

```c
int a = 1;
int b = 2;
```

These local variables usually belong to the stack.

Memory allocated by:

```c
malloc(...)
```

belongs to the heap.

Modern operating systems often use virtual memory and demand paging. This means the OS may not copy the entire executable into physical memory immediately. Instead, it maps parts of the executable and loads pages when they are actually needed.

For basic understanding, it is enough to say:

> The OS makes the executable's code and data available in the process memory space.

---

## 13. Starting Execution

An executable file contains an entry point.

The operating system sets the main thread's Program Counter to the entry point.

The entry point is usually not `main()` directly. A real C program usually starts at a runtime entry such as:

```text
_start
```

The real execution flow is approximately:

```text
_start
  ↓
C runtime initialization
  ↓
main()
  ↓
return from main()
  ↓
exit()
```

So the OS sets:

```text
PC = entry point address
```

Then the CPU starts executing instructions from that address.

---

## 14. CPU Execution Cycle

Once the thread is running, the CPU repeatedly performs this cycle:

```text
1. Read the Program Counter
2. Fetch the instruction from memory at the address stored in the PC
3. Execute the instruction
4. Update registers
5. Update the PC
6. Repeat
```

Example:

```text
Memory
0x1000: create space for x
0x1004: put 10 into x
0x1008: load x into register R1
0x100C: add 1 to R1
0x1010: store R1 back to x
0x1014: return
```

If:

```text
PC = 0x1008
```

then the CPU fetches:

```text
load x into register R1
```

After executing it:

```text
R1 = 10
```

Then the PC moves to the next instruction:

```text
PC = 0x100C
```

---

## 15. When Is a Thread Actually Executing?

A thread is executing on a processor core when its execution context is loaded into the CPU registers.

That means the CPU registers currently contain that thread's:

```text
Program Counter
general-purpose register values
Stack Pointer
execution flags
```

If there is only one CPU core, only one thread can truly execute at one exact moment.

When the OS switches from one thread to another, it performs a context switch.

A context switch roughly means:

```text
1. Save Thread A's registers and PC into memory
2. Load Thread B's saved registers and PC from memory
3. Continue execution from Thread B's PC
```

This is how one CPU core can appear to run many programs or threads at the same time.

---

## 16. Program Termination

When `main()` finishes:

```c
return 0;
```

control returns to the C runtime.

The runtime then calls an exit routine, which eventually performs an operating system system call such as:

```text
exit
```

The OS then cleans up the process:

```text
destroys threads
releases memory
closes open files
records the exit status
notifies the parent process
```

After that:

```text
the process disappears
the thread disappears
the executable file a.out still remains on disk
```

The executable can be run again later.

---

## 17. Common Misunderstandings

### Misunderstanding 1: The compiler creates the process

Wrong:

```text
The compiler turns C code into a running process.
```

Correct:

```text
The compiler and linker create an executable file.
The operating system creates the process when the executable is run.
```

### Misunderstanding 2: The executable file is the process

Wrong:

```text
a.out = process
```

Correct:

```text
a.out = static executable file on disk
process = running instance of a.out
```

### Misunderstanding 3: A process and a thread are the same

Wrong:

```text
process = thread
```

Correct:

```text
process = resource container
thread = execution path
```

### Misunderstanding 4: The Program Counter is an instruction

Wrong:

```text
PC = instruction
```

Correct:

```text
PC = register that stores the address of the next instruction
instruction = machine command stored in memory
```

### Misunderstanding 5: Registers and memory are the same

Wrong:

```text
registers and memory are basically the same
```

Correct:

```text
registers are inside the CPU, very fast, and very small
memory is outside the CPU, larger, and slower
```

---

## 18. Final Summary

The complete execution process of a C program is:

```text
1. The programmer writes foo.c.
2. The compiler translates foo.c into object code.
3. The linker creates an executable file such as a.out.
4. The user runs ./a.out.
5. The shell asks the operating system to execute the file.
6. The OS creates a new process.
7. The OS creates the main thread inside that process.
8. The OS loads or maps the executable's code and data into memory.
9. The OS sets the thread's Program Counter to the program entry point.
10. The CPU fetches instructions from memory using the PC.
11. The CPU executes the instructions one by one.
12. When the program finishes, the OS destroys the process and its threads.
```

The most important sentence is:

> A C source file becomes an executable file through compilation and linking.  
> The executable becomes a running process only when the operating system runs it.  
> The process contains at least one thread, and the CPU executes that thread by using the Program Counter to fetch instructions from memory.
