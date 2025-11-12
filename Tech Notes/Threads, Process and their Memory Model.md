## Process

- Process is an instance of a program that is getting executed.
- It has its own resource like memory, thread etc. OS allocates these resources to process when it's created.

![[Pasted image 20250427181342.png]]

- Two processes never share resources with each other.
- A **process** runs a program and manages resources (memory, files, etc.).

## Thread

- Smallest set of instructions that can be executed by a CPU independently. (lightweight process)
- **Thread** is an **individual path of execution within a process**. 
- A **thread** runs a **sequence of instructions** inside that process — and multiple threads can run **different sequences** at the same time within the **same memory space**.
- A process can have multiple threads.
- When a process is created, it starts with one thread, called the **main thread**, and from that multiple threads can be spawned.
- **Every process must have at least one thread** — the **main thread** — to actually _do_ anything. That main thread **runs the program's code**.

## Threads and Processes in Java

![[Screenshot 2025-04-27 at 6.34.29 PM 1.png]]

- Whenever a new process is created, a new instance of JVM is created and assigned to the process.
- Assigned heap memory to a process can be controlled by `-Xms` and `-Xmx` args.

### Code segment
- Contains the compiled **BYTECODE** (that is, the machine code) of the Java program.
- It is read only.
- All threads in a process share the Code segment.

### Data segment
- Contains the **GLOBAL** and **STATIC** variables.
- Threads can read and modify the same data.
- Synchronization is required between multiple threads.
- All threads in a process share the Data segment.

### Heap
- Objects created during runtime are stored in Heap.
- Threads can read and modify the same data.
- Synchronization is required between multiple threads.
- All threads in a process share the Heap memory.

### Stack
- Each thread has it's own stack.
- It manages method calls, local variables.

### Register
- When JIT (Just in-time) compiler compiles the bytecode to machine code, it uses Register's to optimize the generated machine code. It stores intermediate values during compilation.
- Also helps in context switching. (Stores in the intermediate  results of a thread)
- Each thread has it's own register.

### Program Counter
- It points to the instruction which is currently getting executed.
- Increments it's counter after the instruction has successfully executed.

All these are managed by the JVM.

## Flow of execution

- OS scheduler schedules the threads so that they can be run on the CPU.
- If a thread is currently running, it loads the instruction which is pointed by Program counter to the register and the OS assigns the instruction to the CPU to execute. The instruction is read by the CPU and it starts executing it
- When it has to **context switch**, that is, it stores the intermediate results in the register and then the scheduler assigns another thread to the CPU.
- When there are multiple cores in a CPU, threads can run in parallel.

## Multithreading

- Allow a program to perform multiple tasks at the same time.
- Multiple threads share the same resource such as memory space but still can perform task independently.

### Benefits

- Improved performance
- Maximum resource utilization

### Challenges

- Concurrency issues like deadlock, data inconsistency etc.
- Synchronization overhead.
- Testing and debugging is difficult.