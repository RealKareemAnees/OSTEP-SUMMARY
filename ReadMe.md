# Operating Systms Three Easy Pieces summary

by **Real Kareem Anees**;

- I made this summary to help me revise on OS Concepts

**Liinkedin:** https://www.linkedin.com/in/kareem-anees

![alt text](image.png)

THIS GUIDE IS MEANT FOR SYUDENTS WHO HAD SUCCESSFULY READ THE BOOK AND NEED A REFRESHER

# Operating Systems: Fundamentals & Process Management

## Chapter 3: Introduction to Operating Systems & The Von Neumann Model

### The Foundational Computer Architecture

All modern computers follow the **Von Neumann model of computing**. This is the underlying architectural principle that dictates how a processor executes programs:

#### The Fetch-Decode-Execute Cycle

A running program executes through a repetitive process:

```
┌─────────────────────────────────┐
│   Von Neumann Execution Loop    │
├─────────────────────────────────┤
│ 1. Fetch instruction from memory│
│    (using program counter)      │
│ 2. Decode instruction           │
│    (determine what to execute)  │
│ 3. Execute instruction          │
│    (add numbers, access memory, │
│     check condition, jump, etc) │
│ 4. Move to next instruction     │
│ 5. Repeat (millions/billions    │
│    of times per second)         │
└─────────────────────────────────┘
```

**Key Point:** From the program's perspective, instructions execute one at a time in sequential order. (Modern processors actually do bizarre optimizations underneath—like executing multiple instructions simultaneously or completing them out of order—but this simplicity is what programs assume.)

### The Operating System as Resource Virtualization Layer

The **operating system (OS)** is system software with two primary responsibilities:

1. **Virtualization**: Transform physical hardware resources into virtual, abstract resources that are easier and safer to use
2. **Resource Management**: Coordinate access to shared physical resources (CPU, memory, disk) among multiple competing programs

#### The Three Core Virtualization Domains

The OS virtualizes three main resource types:

| Resource   | Physical Reality                | Virtual Abstraction               | Benefit                                        |
| :--------- | :------------------------------ | :-------------------------------- | :--------------------------------------------- |
| **CPU**    | Single processor (or small set) | Infinite virtual CPUs             | Multiple programs appear to run simultaneously |
| **Memory** | Shared physical RAM             | Private address space per process | Each program sees its own isolated memory      |
| **Disk**   | Single shared storage device    | Files and directories             | Persistent, organized data access              |

### System Calls: The Process-OS Interface

To use the OS's virtualized resources, programs invoke **system calls** (also called syscalls). A typical modern OS exports **hundreds of system calls** that provide APIs for:

- Creating and managing processes
- Allocating and accessing memory
- Reading/writing files and devices
- Inter-process communication
- Setting process priorities and permissions

### The Mechanism vs. Policy Design Pattern

A critical OS design principle separates concerns:

| Aspect        | Definition                                                 | Example                                                   |
| :------------ | :--------------------------------------------------------- | :-------------------------------------------------------- |
| **Mechanism** | _How_ the OS performs an action (low-level implementation) | How does context switching work? (implementation details) |
| **Policy**    | _Which_ decision to make (high-level logic)                | Which process should run next? (scheduling algorithm)     |

This separation allows OS designers to change policies (e.g., switch scheduling algorithms) without rewriting mechanisms. It's a form of **modularity**—a fundamental software design principle.

---

## Chapter 4: The Abstraction—The Process

### What is a Process?

A **process** is the fundamental OS abstraction for a running program. It encapsulates:

- **Code**: the program instructions (immutable once loaded)
- **Data**: static initialized variables and dynamic allocated memory
- **Registers**: CPU register state (program counter, stack pointer, general-purpose registers)
- **Memory Address Space**: the virtual address space assigned to this process
- **I/O State**: open files, network sockets, and other device handles
- **Process ID (PID)**: unique identifier for this process instance

Multiple instances of the same program can run as separate processes, each with isolated state and memory.

### Process Creation: From Program to Running Process

When the OS launches a program, it performs several sequential steps:

#### Step 1: Load Code and Static Data

```
Disk (On-disk executable)  →  Memory (Address Space)
├─ Program bytecode         └─ Code segment (read-only)
└─ Static initialized data     Static data segment
```

**Implementation Detail:**

- **Early/Simple OS**: Eager loading — load entire program before execution starts
- **Modern OS**: Lazy loading — load code/data pieces only as needed during execution (requires paging/swapping machinery)

#### Step 2: Allocate and Initialize Runtime Memory Structures

The OS allocates several memory regions for the process:

```
┌────────────────────────────────┐
│     Process Memory Layout      │
├────────────────────────────────┤
│   Stack (Grows Downward)       │  High Address
│   ↓ ↓ ↓ ↓ ↓                    │  (Function frames,
│                                │   local variables,
│   (Free Space)                 │   return addresses)
│                                │
│   ↑ ↑ ↑ ↑ ↑                    │
│   Heap (Grows Upward)          │  (Dynamic allocation
│                                │   via malloc/free)
├────────────────────────────────┤
│   Uninitialized Data (BSS)     │  Static data
│   Initialized Static Data      │
│   Program Code (Text)          │  Low Address
└────────────────────────────────┘
```

**Key Memory Regions:**

| Region                  | Purpose                                                | Allocation                                          | Management                                      |
| :---------------------- | :----------------------------------------------------- | :-------------------------------------------------- | :---------------------------------------------- |
| **Stack**               | Local variables, function parameters, return addresses | OS allocates fixed size                             | Grows/shrinks automatically with function calls |
| **Heap**                | Dynamically allocated objects                          | OS provides initial allocation; grows on `malloc()` | Manual via `malloc()`/`free()` in C             |
| **Initialized Data**    | Global/static variables with initial values            | OS loads from executable                            | Fixed, set at initialization                    |
| **BSS (Uninitialized)** | Global/static variables without initial values         | OS zeroes out                                       | Fixed, set to zero                              |
| **Code/Text**           | Program instructions                                   | OS loads from executable                            | Read-only; shared among process instances       |

#### Step 3: Initialize I/O File Descriptors

In UNIX-based systems, the OS opens three standard file descriptors for every process:

| File Descriptor | Name   | Purpose         | Default Source/Destination |
| :-------------- | :----- | :-------------- | :------------------------- |
| **0**           | stdin  | Standard input  | Terminal keyboard          |
| **1**           | stdout | Standard output | Terminal screen            |
| **2**           | stderr | Standard error  | Terminal screen            |

These are stored in the process's open file table and enable the process to read from the keyboard and write to the screen without explicit file operations.

#### Step 4: Start Program Execution

The OS transfers CPU control to the process by jumping to the `main()` function entry point. From this moment, the process begins executing its instructions.

### The Process API

Any modern OS provides these fundamental operations on processes:

#### Create

- **System Call**: `fork()` (UNIX) or `CreateProcess()` (Windows)
- **Trigger**: User types shell command, double-clicks application icon
- **Kernel Action**: Allocates process structure, loads program code, initializes memory regions, adds to process list

#### Destroy

- **System Call**: `exit()` (implicit); `kill()` (explicit termination)
- **Trigger**: Process finishes naturally; user force-terminates runaway process
- **Kernel Action**: Deallocates all process resources, removes from scheduling queue

#### Wait

- **System Call**: `wait()`/`waitpid()`
- **Purpose**: Parent process blocks until a child process completes
- **Return Value**: Exit status code of child process

#### Miscellaneous Control

- **Suspend**: Pause process execution without terminating (e.g., `SIGSTOP` signal)
- **Resume**: Continue suspended process (e.g., `SIGCONT` signal)
- **Priority Adjustment**: `nice()`, `renice()` to change scheduling priority

#### Status

- **Query Runtime Metrics**: How long has the process run? What state is it in?
- **System Call**: `getpid()`, `getppid()`, `/proc/[pid]/` filesystem queries

### Process States and State Transitions

A process exists in one of three primary states at any moment:

#### Running

- **Definition**: Process is executing instructions on a CPU
- **Resource Allocation**: Actively using processor time
- **Transition Out**: Scheduler decides to switch processes (descheduling) OR process initiates I/O

#### Ready

- **Definition**: Process is prepared to run but OS scheduler hasn't selected it
- **Resource Status**: Waiting for CPU time
- **Reason**: Another process is running; scheduler has higher-priority processes
- **Transition**: To Running (scheduling), To Ready (deschedule)

#### Blocked

- **Definition**: Process is waiting for an external event and cannot proceed
- **Common Events**:
  - Disk I/O completion (reading a file)
  - Network packet arrival
  - Timer expiration
  - Lock acquisition in concurrent code
- **Transition**: To Ready when event completes (e.g., disk finishes read)

#### Extended States (in some operating systems)

| State          | Meaning                                            | Typical Use Case                |
| :------------- | :------------------------------------------------- | :------------------------------ |
| **Embryo/New** | Process being created, not yet runnable            | During `fork()` setup           |
| **Zombie**     | Process exited but parent hasn't called `wait()`   | Parent delayed in cleanup       |
| **Sleeping**   | Process blocked and temporarily not checking event | Long-term waits in some kernels |

### Process State Transition Diagram

```
┌──────────────┐
│   RUNNING    │ ← (Scheduler selects)
└──────┬───────┘
       │
       │ (Scheduler: time quantum         │ (Process: initiate I/O)
       │  expired or preempted)           │
       │                                  │
       ↓                                  ↓
   ┌────────────┐                    ┌─────────────┐
   │   READY    │                    │   BLOCKED   │
   └────────────┘                    └─────────────┘
       ↑                                  │
       │ (Event completed,            (I/O done,
       │  I/O ready, etc)             wake-up signal)
       └──────────────────────────────┘
```

### Scenario: Process Execution with CPU-Only Workload

Two processes, each performing pure CPU computation (no I/O):

```
Time  Process 0    Process 1    Notes
────────────────────────────────────────────────────
 1    Running      Ready
 2    Running      Ready
 3    Running      Ready
 4    Running      Ready        Process 0 completes
 5    –            Running
 6    –            Running
 7    –            Running
 8    –            Running      Process 1 completes
```

**Scheduler Decision**: After Process 0 completes, Process 1 is the only ready process, so it begins execution.

### Scenario: Process Execution with I/O Blocking

Two processes where Process 0 initiates disk I/O:

```
Time  Process 0    Process 1    Notes
────────────────────────────────────────────────────────
 1    Running      Ready
 2    Running      Ready
 3    Running      Ready        Process 0: initiate I/O
 4    Blocked      Running      OS blocks P0, runs P1
 5    Blocked      Running
 6    Blocked      Running
 7    Ready        Running      I/O completes, P0 ready
 8    Ready        Running      P1 still running
 9    Running      Ready        OS switches to P0
10    Running      Ready        P0 completes
```

**Why I/O Triggers Blocking:**

- Disk is extremely slow (~milliseconds) vs. CPU (~nanoseconds)
- If Process 0 busy-waits for disk, CPU sits idle for millions of cycles
- **Optimization**: Context switch to Process 1, utilize otherwise wasted CPU time
- **Resource Efficiency**: Improves CPU utilization by reducing idle cycles

### Process Data Structures: The Kernel Process Table

The OS maintains a **process list** (or **task list**) containing metadata for every active process. Each entry is called a **Process Control Block (PCB)** or **process descriptor**.

#### xv6 Kernel Process Structure

Here's the actual data structure from the xv6 operating system, a minimal but real Unix-like kernel:

```c
// CPU register state: saved when process stops, restored when resuming
struct context {
    int eip;        // Instruction pointer (next instruction to execute)
    int esp;        // Stack pointer (top of stack)
    int ebx;        // General-purpose registers (used for data/calculations)
    int ecx;
    int edx;
    int esi;
    int edi;
    int ebp;        // Base pointer (stack frame marker)
};

// Process state enumeration
enum proc_state {
    UNUSED,         // Slot not in use
    EMBRYO,         // Process being initialized
    SLEEPING,       // Blocked on I/O or event
    RUNNABLE,       // Ready to run (in scheduler queue)
    RUNNING,        // Currently executing on CPU
    ZOMBIE          // Exited, awaiting parent wait()
};

// Core process control block
struct proc {
    char *mem;                      // Start address of process memory
    uint sz;                        // Size of process memory (bytes)
    char *kstack;                   // Kernel stack bottom ptr

    enum proc_state state;          // Current process state
    int pid;                        // Process ID (unique identifier)
    struct proc *parent;            // Pointer to parent process

    void *chan;                     // Event channel (if blocked, what are we waiting for?)
    int killed;                     // Killed flag: non-zero if termination requested

    struct file *ofile[NOFILE];     // Open file table (indexed by file descriptor)
    struct inode *cwd;              // Current working directory inode

    struct context context;         // Register state (what to restore on context switch)
    struct trapframe *tf;           // Trap frame for interrupt handling
};
```

**Kernel Behavior When Saving/Restoring Process State:**

When the OS context-switches away from a process:

1. **Save**: Copy current CPU register values into `proc->context`
2. **Kernel Mode**: Enter OS scheduler code
3. **Select Next Process**: Find next ready process to run
4. **Restore**: Copy selected process's `proc->context` back into CPU registers
5. **Resume**: Return to user mode; process continues from exact instruction where it stopped

This mechanism makes process switching transparent: a process has no special knowledge it was paused.

---

## Chapter 5: The Process API in Historical Context

### The Era of Multiprogramming

**Timeline**: 1960s–1970s

The minicomputer era (e.g., DEC PDP family) made computers affordable for small organizations. This drove demand for **multiprogramming**: running multiple jobs simultaneously on shared hardware.

#### The Core Problem

```
Traditional Single-Job Execution:
┌─────────────────────────────────┐
│ Program A                       │
│ ├─ CPU computation: 50ms        │
│ ├─ Wait for disk I/O: 150ms     │ ← CPU is idle!
│ ├─ CPU computation: 50ms        │
│ └─ Complete                     │
│ Total time: 250ms               │
│ CPU utilization: (50+50)/250    │
│ = 40% ▲ Wasteful!               │
└─────────────────────────────────┘
```

#### The Multiprogramming Solution

```
Multiplexed Execution:
┌─────────────────────────────────┐
│ Program A: 50ms compute         │
│ Program A: disk I/O requested   │
│   ↓ CONTEXT SWITCH              │
│ Program B: 80ms compute         │
│ Program B: 20ms compute         │
│ Program B: disk I/O requested   │
│   ↓ CONTEXT SWITCH              │
│ Program A: disk I/O done        │
│ Program A: 50ms compute         │
│ Program A: complete             │
│ Program B: disk I/O done        │
│ Program B: 20ms compute         │
│ Program B: complete             │
│                                 │
│ CPU utilization: ~95%           │ ← Near full utilization!
└─────────────────────────────────┘
```

### Key OS Innovations Required for Multiprogramming

#### Memory Protection

- **Problem**: Early systems lacked hardware support; malicious or buggy programs could overwrite memory of other processes or the OS itself
- **Solution**: Hardware memory management unit (MMU) + virtual memory enforcement
- **Impact**: Enables safe isolation of multiple programs

#### Interrupt and Exception Handling

- **Problem**: OS needs to regain CPU control from user processes
- **Mechanism**: Hardware timer interrupts and exception traps
- **Result**: OS kernel can pause any process and context-switch

#### Concurrent Access Control

- **Problem**: Multiple processes accessing shared hardware (I/O devices, locks) create race conditions
- **Solution**: Semaphores, monitors, locks, condition variables (covered in later sections)

### The UNIX Philosophy

UNIX (developed at Bell Labs by Ken Thompson and Dennis Ritchie in the 1970s) succeeded because it:

1. **Simplified Design**: Elegant, focused architecture
2. **Composability**: Small programs ("tools") connected via pipes: `grep foo file.txt | wc -l`
3. **Accessibility**: Source code distributed freely; written in C (an accessible language)
4. **Portability**: C enabled porting across machines
5. **Extensibility**: Simple kernel design allowed others to add features

This philosophy survives in modern systems: Linux, macOS (UNIX-based), and modern Windows all carry UNIX's DNA.

---

## Chapter 6: Mechanisms—Limited Direct Execution

### The Core Challenge: Running User Programs Safely and Efficiently

The OS faces a fundamental tension:

```
EFFICIENCY          vs.        SAFETY & CONTROL
┌──────────────────┐           ┌──────────────────┐
│ Run user program │           │ OS must:         │
│ directly on CPU  │           │ • Prevent abuse  │
│ (minimal overhead)           │ • Manage resources│
│                  │           │ • Enforce policy │
└──────────────────┘           └──────────────────┘
```

**Direct Execution**: Let programs run directly on the CPU (fast)
**Limited**: But restrict what they can do (safe)

### The Dual-Mode Execution Model

Modern CPUs provide two execution modes:

#### User Mode

- **Permissions Granted**:
  - Execute non-privileged instructions (arithmetic, logic, jumps, memory access)
- **Permissions Denied**:
  - Change processor mode
  - Perform I/O operations
  - Modify memory protection registers
  - Halt the CPU
- **Purpose**: Isolate user processes; prevent them from harming each other or OS

#### Kernel Mode (Supervisor Mode)

- **Permissions Granted**:
  - Execute ALL instructions (including privileged ones)
  - Access all memory
  - Perform I/O
  - Manage interrupts and exceptions
- **Owner**: Operating system kernel only
- **Purpose**: Control system resources and manage user processes

### Transitioning Between Modes

#### User Mode → Kernel Mode: System Call

**Mechanism**: System call trap (software interrupt)

```
User Process:                   Kernel:
┌──────────────────────────┐   ┌──────────────────────────┐
│ User Mode                │   │                          │
│ ...normal code...        │   │                          │
│                          │   │                          │
│ syscall(SYS_open)  ──────┼──→│ Kernel Mode              │
│ [CPU switches mode]      │   │ Trap handler activated   │
│ [enters kernel]          │   │ Execute syscall logic    │
│ [hardware jumps to      │   │ (open file, validate)    │
│  kernel trap table]      │   │ Return result            │
│                          │  ─┼──→ Return to user mode   │
│ ..continue normal code.. │   │ User process resumes     │
└──────────────────────────┘   └──────────────────────────┘
```

**Key Points:**

- User process cannot directly execute privileged instructions
- Instead, it issues a **trap** (special instruction)
- Hardware automatically switches to kernel mode
- OS kernel handles request
- Returns control to user process with result

#### Kernel Mode → User Mode: Return from System Call

After the kernel completes a system call:

1. **Kernel executes privileged instruction**: Sets mode bit to user mode
2. **Context restored**: User process's registers and state
3. **Control transferred**: Program counter set to next instruction after syscall
4. **Resume**: User process executes as if nothing special happened

### Context Switching: Saving and Restoring Process State

When the OS scheduler switches away from one process to another:

**Step 1: Save process state**

```c
// Simplified pseudocode in assembler
// (This is done in a trap handler or interrupt handler)

// Save user registers
save_register_context(current_process.context);

// Hardware automatic actions:
// - esp (stack pointer) saved to kernel stack
// - eip (instruction pointer) saved
// - mode bit set to kernel

// Now in kernel mode; can access kernel memory
```

**Step 2: Select next process**

```c
next_process = scheduler_pick_next_process();
// Scheduler uses policy (e.g., round-robin, priority)
```

**Step 3: Restore next process state**

```c
// Switch address space (page table pointer)
set_page_table_register(next_process.page_table);

// Restore register context
restore_register_context(next_process.context);

// Executes privileged instruction to switch back to user mode
// Hardware clears mode bit, system continues execution
// with next process's instructions from saved pc (eip)
```

### Processor Timer Interrupt: Enforcing Time Multiplexing

**Problem**: A runaway process (infinite loop) could monopolize the CPU, starving other processes

**Solution**: Hardware timer interrupt

```
Initialization (during boot):
┌────────────────────────────────────────────┐
│ Kernel sets up timer:                      │
│ - Configure hardware timer                 │
│ - Set interval: e.g., 10ms                 │
│ - Register interrupt handler address       │
│ - Enable timer interrupts                  │
└────────────────────────────────────────────┘

Every 10ms (during user process execution):
┌────────────────────────────────────────────┐
│ Timer hardware fires interrupt              │
│ CPU automatically:                         │
│ 1. Suspends current process                │
│ 2. Saves process state                     │
│ 3. Switches to kernel mode                 │
│ 4. Calls kernel timer interrupt handler    │
│                                            │
│ Kernel timer handler:                      │
│ 1. Acknowledge timer                       │
│ 2. Update process accounting               │
│ 3. Call scheduler                          │
│ 4. Switch to next ready process            │
│ 5. Return to user mode                     │
└────────────────────────────────────────────┘
```

This **timer interrupt** is involuntary—the user process cannot prevent it. It guarantees the OS regains control periodically, enabling fairness and preventing starvation.

### I/O Operations: Blocking and Non-Blocking

#### Blocking I/O (Traditional Model)

```
Process requests disk read:
┌──────────────────────────────────┐
│ syscall(SYS_read, fd)            │
│ → Kernel mode                    │
│ → Initiate disk operation        │
│ → Process state: BLOCKED         │
│                                  │
│ Scheduler picks different Process│
│                                  │
│ (when disk finishes)             │
│ Interrupt fires (disk completion)│
│ Kernel wakes Process:            │
│ → Move from BLOCKED → READY      │
│                                  │
│ (when scheduler runs Process)    │
│ → Kernel return from syscall     │
│ → User mode                      │
│ → Read data in process buffer    │
└──────────────────────────────────┘
```

**Efficiency Benefit**: During I/O wait, CPU isn't idle—it runs other processes.

---

### CPU Virtualization Demonstration: Code Analysis

#### Example 1: Single-Process CPU Usage (cpu.c)

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/time.h>
#include <assert.h>
#include "common.h"

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "usage: cpu <string>\n");
        exit(1);
    }
    char *str = argv[1];
    while (1) {
        Spin(1);                    // Spin for 1 second
        printf("%s\n", str);        // Print the string
    }
    return 0;
}
```

**Execution Trace:**

```
$ gcc -o cpu cpu.c -Wall
$ ./cpu "A"
A
A
A
A
ˆC  (user presses Ctrl-C to terminate)
```

**What's Happening:**

1. `Spin(1)`: Busy-wait loop checks current time repeatedly until 1 second elapses
2. `printf()`: Outputs the string "A"
3. Loop repeats infinitely
4. CPU is continuously executing this single process

**Kernel Behavior:**

- Timer interrupt fires every ~10ms
- Kernel temporarily suspends this process
- Scheduler checks if higher-priority process exists (usually none)
- Scheduler returns to this process
- Process resumes execution

---

#### Example 2: Demonstrating CPU Virtualization with Multiple Processes

**Running 4 instances simultaneously:**

```
$ ./cpu A & ./cpu B & ./cpu C & ./cpu D &
[1] 7353
[2] 7354
[3] 7355
[4] 7356
A
B
D
C
A
B
D
C
A
...
```

**Observable Behavior:**

- 4 processes run on a single CPU
- Output interleaves: A, B, D, C, A, B, D, C...
- Each process gets a time slice (typically 10ms with Unix schedulers)
- **Illusion created**: Each process has its own private CPU

**How the OS Creates This Illusion:**

```
Timeline (horizontal = time):
┌─────────┬─────────┬─────────┬─────────┬─────────┐
│Process A│Process B│Process C│Process D│Process A│...
│10ms     │10ms     │10ms     │10ms     │10ms     │
│ A       │ B       │ C       │ D       │ A       │
└─────────┴─────────┴─────────┴─────────┴─────────┘

Each process perceives:
Process A: "I'm running continuously..."
           (unaware of 30ms gaps between its time slices)
```

**Kernel Mechanism:**

1. Timer fires every 10ms
2. Scheduler saves current process state
3. Scheduler selects next ready process (round-robin: A → B → C → D → A)
4. Kernel restores next process's state and returns to user mode
5. Process resumes execution unaware of being paused

---

#### Example 3: Memory Virtualization Demonstration (mem.c)

```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include "common.h"

int main(int argc, char *argv[]) {
    int *p = malloc(sizeof(int));       // Allocate memory for one int
    assert(p != NULL);
    printf("(%d) address pointed to by p: %p\n",
           getpid(), p);                 // Print process ID and address
    *p = 0;                              // Initialize value to 0
    while (1) {
        Spin(1);                         // Wait 1 second
        *p = *p + 1;                     // Increment value
        printf("(%d) p: %d\n", getpid(), *p);  // Print PID and current value
    }
    return 0;
}
```

**Single Process Execution:**

```
$ ./mem
(2134) address pointed to by p: 0x200000
(2134) p: 1
(2134) p: 2
(2134) p: 3
(2134) p: 4
(2134) p: 5
ˆC
```

**What's Happening:**

- `malloc()` allocates 4 bytes of memory and returns virtual address `0x200000`
- `getpid()` returns process ID 2134
- Loop increments the integer once per second
- From process's perspective: "I have memory at address 0x200000"

---

**Two Processes Running Simultaneously:**

```
$ ./mem &; ./mem &
[1] 24113
[2] 24114
(24113) address pointed to by p: 0x200000
(24114) address pointed to by p: 0x200000
(24113) p: 1
(24114) p: 1
(24114) p: 2
(24113) p: 2
(24113) p: 3
(24114) p: 3
(24113) p: 4
(24114) p: 4
...
```

**The Magic: Virtual Address Spaces**

```
Physical Reality:
┌────────────┐  ┌────────────┐
│ Physical   │  │ Physical   │
│ Memory     │  │ Memory     │
│ (Unified)  │  │ (contains  │
│            │  │ data for   │
│            │  │ both PIDs) │
└────────────┘  └────────────┘

What Each Process Sees (Virtual Reality):
┌──────────────────┐  ┌──────────────────┐
│ Process 24113    │  │ Process 24114    │
│ Virtual Memory   │  │ Virtual Memory   │
│                  │  │                  │
│ 0x200000 ----→   │  │ 0x200000 ----→   │
│ (my value = 1,   │  │ (my value = 23,  │
│  2, 3, 4...)     │  │  independent!)   │
│                  │  │                  │
└──────────────────┘  └──────────────────┘
```

**OS Mechanism: Virtual Address Translation**

```
User Process (PID 24113):
  Memory reference: 0x200000
           ↓
  [MMU (Memory Management Unit)]
  Consults Page Table for PID 24113
           ↓
  Virtual address 0x200000 → Physical address 0x3000
           ↓
  Read/write physical memory location 0x3000

User Process (PID 24114):
  Memory reference: 0x200000
           ↓
  [MMU (Memory Management Unit)]
  Consults Page Table for PID 24114
           ↓
  Virtual address 0x200000 → Physical address 0x9000
           ↓
  Read/write physical memory location 0x9000

Result: Both processes think they're at 0x200000,
        but they're actually at different physical locations!
```

**Key Insight**: The OS maps each process's virtual address space independently to physical memory. Processes remain isolated: modifications by one process never affect another's memory.

---

### Concurrency Introduction: The Race Condition Problem

#### Multi-threaded Counter Increment Example (threads.c)

```c
#include <stdio.h>
#include <stdlib.h>
#include "common.h"

volatile int counter = 0;       // Shared global counter (volatile: compiler doesn't optimize away)
int loops;                      // How many times each thread increments

void *worker(void *arg) {
    int i;
    for (i = 0; i < loops; i++) {
        counter++;              // Increment shared counter
    }
    return NULL;
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "usage: threads <value>\n");
        exit(1);
    }
    loops = atoi(argv[1]);
    pthread_t p1, p2;
    printf("Initial value : %d\n", counter);

    // Create 2 threads
    Pthread_create(&p1, NULL, worker, NULL);    // Thread 1
    Pthread_create(&p2, NULL, worker, NULL);    // Thread 2

    // Wait for both to finish
    Pthread_join(p1, NULL);
    Pthread_join(p2, NULL);

    printf("Final value : %d\n", counter);
    return 0;
}
```

**First Execution (loops = 1000):**

```
$ gcc -o thread thread.c -Wall -pthread
$ ./thread 1000
Initial value : 0
Final value : 2000
```

**Expected Result**: Each thread increments 1000 times → total 2000 increments
**Actual Result**: 2000 ✓ Correct!

---

**But Now Try Higher Values (loops = 100,000):**

```
$ ./thread 100000
Initial value : 0
Final value : 143012  // huh??  Expected 200000!

$ ./thread 100000
Initial value : 0
Final value : 137298  // what the??  Different value again!

$ ./thread 100000
Initial value : 0
Final value : 200000  // Correct sometimes!
```

**Why The Variance?**

The culprit: `counter++` is **not atomic**. It compiles to three machine instructions:

```
// What the C code does:
counter++;

// What the CPU actually executes:
1. LOAD R1, [address of counter]    // Load counter value into register
2. ADD R1, 1                         // Increment register
3. STORE R1, [address of counter]   // Write back to memory
```

**The Race Condition:**

```
Thread 1 (T1)                Thread 2 (T2)
────────────────────────────────────────────────────
LOAD R1, counter         (R1 = 143)

                         LOAD R2, counter    (R2 = 143)

ADD R1, 1                (R1 = 144)
STORE R1, counter        (counter = 144)

                         ADD R2, 1           (R2 = 144)
                         STORE R2, counter   (counter = 144)

Final value: 144 (should be 145!)
Both threads incremented, but result shows only one increment.
```

**Why Low Loop Values Sometimes Work:**

With `loops = 1000`, context switches occur _between_ increment operations frequently enough that thread interleaving is randomized. Sometimes:

- All of T1's increments complete before T2 starts
- Or increments are perfectly paired with context switches
- By luck, final count is correct

With `loops = 100,000`, more opportunity for race conditions; incorrect interleaving becomes statistically likely.

**The Fundamental Problem:**

```
┌─────────────────────────────────────────┐
│ CRITICAL SECTION PROBLEM               │
├─────────────────────────────────────────┤
│ Multiple threads accessing shared data  │
│ where at least one performs a write     │
│ and operations aren't atomic = RACE     │
└─────────────────────────────────────────┘
```

This is **the core concurrency problem** that requires synchronization primitives (locks, semaphores, condition variables—topics for later sections).

---

## Key Takeaways

#### Operating System Fundamentals

- The **Von Neumann model** dictates sequential instruction execution: fetch → decode → execute → repeat
- The OS **virtualizes resources** (CPU, memory, disk) through mechanisms and policies
- **Virtualization** creates the illusion of private resources (e.g., infinite virtual CPUs, private memory address spaces)
- **Resource management** ensures fair, efficient sharing of physical hardware among competing processes

#### The Process Abstraction

- A **process** is an instance of a running program with isolated state: code, memory, registers, file descriptors
- **Process creation** involves loading code/data, allocating memory regions (stack/heap), initializing I/O, and jumping to `main()`
- **Process states** (Running, Ready, Blocked) and transitions enable multiprogrammed execution
- The **process list** data structure tracks all active processes; the scheduler uses this to make policy decisions
- **Process isolation** (via virtual memory and mode bits) prevents buggy/malicious processes from harming others

#### Limited Direct Execution

- **Dual-mode CPU execution** (user/kernel mode) enforces hardware-level restrictions on what processes can do
- **System calls** allow processes to request privileged operations in a controlled manner; hardware mode switch to kernel
- **Context switching** saves/restores process state, enabling multiprogramming; **timer interrupts** force periodic context switches
- **I/O operations** cause processes to block; scheduler uses idle time to run other processes, improving CPU utilization
- **Virtual memory** (address translation) gives each process its own address space; physical memory is multiplexed

#### Concurrency Challenges

- **Non-atomic operations** on shared data (like `counter++`) can race when multiple threads execute concurrently
- **Race conditions** arise when operation order matters but isn't guaranteed; unsynchronized access produces undefined results
- Race conditions are **time-dependent** and often not reproducible; hard to detect and debug
- Resolution requires **synchronization primitives** (locks, semaphores) to enforce mutual exclusion on critical sections

---

## Important Terms

| Term                             | Meaning & Context                                                                                                                                                  |
| :------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Von Neumann Model**            | Foundational computer architecture: fetch instruction, decode it, execute it, repeat. Forms the basis for how CPUs execute sequential programs.                    |
| **Virtualization**               | OS technique of abstracting physical resources into virtual resources; e.g., single CPU → multiple virtual CPUs, physical RAM → private virtual address spaces.    |
| **Process**                      | Running instance of a program; includes code, data, registers, memory address space, I/O state, and process ID. Each process is isolated.                          |
| **Process ID (PID)**             | Unique integer identifier for each running process in the system; used to reference the process.                                                                   |
| **Process Control Block (PCB)**  | Kernel data structure storing all metadata about a process: state, registers, memory info, open files, parent pointer. Also called process descriptor.             |
| **Process State**                | Current condition of a process: Running (executing on CPU), Ready (ready to run, waiting for CPU), or Blocked (waiting for event).                                 |
| **Context Switch**               | Mechanism of saving running process's state and restoring next process's state to multiplex CPU among processes. Happens on timer interrupt or system call.        |
| **Scheduling**                   | OS policy-making: deciding which ready process to run next. Scheduler uses algorithms (round-robin, priority, etc.) to make this decision.                         |
| **System Call (Syscall)**        | Request from user process to kernel to perform privileged operation (e.g., file I/O, memory allocation, process creation). Causes trap to kernel mode.             |
| **Trap**                         | Software interrupt (special CPU instruction) that switches from user mode to kernel mode, allowing process to request OS service.                                  |
| **User Mode**                    | CPU execution mode with restricted permissions; user processes run here. Prevents direct access to privileged operations and hardware.                             |
| **Kernel Mode**                  | CPU execution mode with full permissions; only OS kernel runs here. Allows privileged instructions, I/O, memory management, interrupt handling.                    |
| **Mode Bit**                     | CPU register bit that determines execution mode (0=kernel, 1=user). Only privileged instructions can modify it.                                                    |
| **Timer Interrupt**              | Hardware timer fires periodically (e.g., every 10ms), forcing CPU to switch to kernel mode and invoke scheduler. Prevents process starvation.                      |
| **I/O Blocking**                 | Process transitions to Blocked state when it initiates I/O operation (e.g., disk read). Scheduler runs other processes while I/O completes, improving utilization. |
| **Address Space**                | Virtual memory region assigned to a process; contains code, static data, heap, and stack. Each process has isolated address space.                                 |
| **Virtual Address**              | Address as seen by process; OS memory management unit (MMU) translates to physical address. Enables isolation and address space virtualization.                    |
| **Physical Address**             | Actual location in hardware RAM; invisible to user processes; managed by OS page tables.                                                                           |
| **Memory Management Unit (MMU)** | Hardware component that translates virtual addresses to physical addresses using page tables. Enables memory virtualization.                                       |
| **Stack**                        | Memory region growing downward; stores function parameters, local variables, return addresses. Automatically managed by CPU (esp register).                        |
| **Heap**                         | Memory region growing upward; stores dynamically allocated objects via `malloc()`. Manually managed by programmer.                                                 |
| **Process List**                 | Kernel data structure (linked list or array) containing PCB entries for all active processes. Scheduler queries this to select next process.                       |
| **Zombie Process**               | Exited process whose resources haven't been released; awaiting parent process to call `wait()`. Preventive measure allows parent to query child's exit status.     |
| **Race Condition**               | Unintended behavior when multiple threads access shared data without synchronization; result depends on unpredictable execution order.                             |
| **Critical Section**             | Code segment accessing shared data; must be protected by synchronization to prevent race conditions.                                                               |
| **Atomic Operation**             | Instruction or operation that executes indivisibly; cannot be interrupted or interleaved with other threads. Prevents race conditions on that operation.           |
| **Multiprogramming**             | OS technique of running multiple programs concurrently on shared hardware; improved CPU utilization via context switching.                                         |
| **Scheduler**                    | OS kernel component responsible for selecting which ready process runs next; implements scheduling policy.                                                         |
| **Priority**                     | Indication of process importance; scheduler may prefer higher-priority processes. Adjustable via `nice()`.                                                         |
| **Foreground Process**           | Process attached to terminal; runs visibly. Stopping it (EOF/Ctrl-C) stops the process.                                                                            |
| **Background Process**           | Process detached from terminal; runs invisibly in background. Controlled via shell job control (e.g., `./program &`).                                              |
| **File Descriptor**              | Integer handle (0=stdin, 1=stdout, 2=stderr) used by process to reference open file or device. Stored in process's open file table.                                |
| **Lazy Loading**                 | OS memory technique of loading program code/data only when first accessed, not all at startup. Reduces initialization time and memory pressure.                    |
| **Eager Loading**                | OS memory technique of loading entire program into memory before execution starts. Simple but less efficient.                                                      |

---

**End of Study Guide—Chapters 3–6**

# Operating Systems: Process Scheduling and Virtualization Study Guide

A comprehensive technical guide to understanding operating system fundamentals, with focus on process management, CPU virtualization, and scheduling policies.

---

## Chapter 1: Interlude - The Process API

### Overview

The Process API provides the fundamental mechanisms through which user programs create, manage, and terminate processes. The design of these APIs directly impacts how applications interact with the operating system kernel. In UNIX systems, two system calls form the core of process creation: `fork()` and `exec()`.

### The fork() System Call

#### Purpose and Mechanism

The `fork()` system call creates a new process by making an **almost-identical copy** of the currently running process (the parent). This copy includes:

- The entire address space (code, data, heap, stack)
- All registers and program counter state
- Copies of all open file descriptors

**Critical Detail**: `fork()` returns a different value to parent and child:

- **Parent process**: receives the child's Process ID (PID) as a positive integer
- **Child process**: receives `0`
- **Error condition**: returns `-1` if fork fails

```c
int rc = fork();

if (rc < 0) {
    // fork() failed; cannot create new process
    fprintf(stderr, "fork failed\n");
    exit(1);
} else if (rc == 0) {
    // This code runs in the CHILD process
    printf("Child process PID: %d\n", getpid());
} else {
    // This code runs in the PARENT process
    printf("Parent process. Child ID: %d\n", rc);
}
```

**Kernel Behavior on `fork()`**:

1. OS allocates new process control block and kernel stack
2. OS copies entire parent memory space to child
3. OS initializes child's file descriptor table (shares same open files initially)
4. OS returns control to both processes via return-from-trap instruction
5. Each process resumes from the same instruction following `fork()`

#### Why fork() and exec() Are Separate

A key insight in UNIX design is the **separation of concerns**: `fork()` creates a new process, while `exec()` replaces that process's program with an entirely different one. This design enables shell functionality and interprocess features.

### The exec() Family of System Calls

#### Purpose and Mechanism

`exec()` is a family of system calls that **transforms** the currently running process into a completely different program. Unlike `fork()`, which creates a new process, `exec()` overwrites the current process's memory:

- The code segment is replaced
- Static data is replaced
- Heap and stack are re-initialized
- Program counter jumps to the new program's `main()` entry point

**Critical Detail**: A successful `exec()` call **never returns**. The original program ceases to exist; only the new program continues execution.

```c
char *myargs[3];
myargs[0] = strdup("wc");           // Program name
myargs[1] = strdup("file.txt");     // First argument
myargs[2] = NULL;                   // NULL terminates arg array

execvp(myargs[0], myargs);
// If we reach here, exec() FAILED
fprintf(stderr, "exec() failed\n");
```

**Kernel Behavior on `exec()`**:

1. OS validates the executable file
2. OS frees old process memory (code, data, heap)
3. OS allocates new memory for executable
4. OS loads executable into memory
5. OS initializes stack with command-line arguments (argv)
6. OS initializes registers and jumps to new program's entry point

#### Common exec() Variants

| Function   | Argument Format               | Path Lookup          |
| :--------- | :---------------------------- | :------------------- |
| `exec()`   | Full path required            | No                   |
| `execl()`  | List of arguments             | No                   |
| `execlp()` | List of arguments             | Yes (searches $PATH) |
| `execv()`  | Array of arguments            | No                   |
| `execvp()` | Array of arguments            | Yes (searches $PATH) |
| `execve()` | Array + environment variables | No                   |

### Practical Pattern: fork() + exec()

The shell implements command execution through this pattern:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

int main() {
    int rc = fork();

    if (rc < 0) {
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
        // CHILD: Set up program and execute
        char *myargs[3];
        myargs[0] = strdup("wc");
        myargs[1] = strdup("myfile.c");
        myargs[2] = NULL;
        execvp(myargs[0], myargs);
        // Only reached if exec() fails
        fprintf(stderr, "exec failed\n");
        exit(1);
    } else {
        // PARENT: Wait for child completion
        int status;
        pid_t wpid = wait(&status);
        printf("Child %d terminated with status %d\n", wpid, status);
    }
    return 0;
}
```

**Timeline of Execution**:

```
Parent                          Child
------                          -----
fork() called
     │
     ├─────────────────────────→ Child process created
     │                           │
     │                           Child is identical copy
     │                           │
wait(&status) blocks            exec() called
     │                           │
     │                           Program replaced with `wc`
     │                           │
     │                           wc executes on file
     │                           │
     │                           wc completes
     │                           │
     │                           exit() called
     │                           │
wait() returns                   Child terminated
     │
Parent continues
```

### Advanced Technique: Input/Output Redirection

The separation of `fork()` and `exec()` enables powerful shell features like output redirection. Between `fork()` and `exec()`, the child can modify its environment without affecting the parent.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>
#include <sys/wait.h>

int main(int argc, char *argv[]) {
    int rc = fork();

    if (rc < 0) {
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
        // CHILD: Redirect output to file

        // Step 1: Close standard output (file descriptor 0)
        close(STDOUT_FILENO);

        // Step 2: Open new file; OS assigns next available FD (0)
        open("./output.txt", O_CREAT | O_WRONLY | O_TRUNC, S_IRWXU);

        // Step 3: Any write to stdout now goes to output.txt
        char *myargs[3];
        myargs[0] = strdup("wc");
        myargs[1] = strdup("input.c");
        myargs[2] = NULL;
        execvp(myargs[0], myargs);
    } else {
        // PARENT: Wait for child
        wait(NULL);
    }
    return 0;
}
```

**Why This Works**:

UNIX systems locate file descriptors sequentially starting from 0. When `close(STDOUT_FILENO)` removes the stdout descriptor, the next `open()` call reuses descriptor 0. All subsequent writes to stdout are transparently redirected to the file.

**File Descriptor Table** (conceptual):

```
Before redirection:
FD 0 (stdin)  → /dev/stdin
FD 1 (stdout) → /dev/stdout
FD 2 (stderr) → /dev/stderr

After close(1) and open("file.txt"):
FD 0 (stdin)  → /dev/stdin
FD 1          → /home/user/file.txt
FD 2 (stderr) → /dev/stderr
```

### The wait() System Call

#### Purpose and Mechanism

`wait()` allows a parent process to **suspend execution** until a specific child process completes. This synchronizes parent and child execution.

```c
pid_t wait(int *status);
```

**Parameters**:

- `status`: Pointer to integer where exit status will be stored. Pass `NULL` if status not needed.

**Return Value**:

- On success: PID of terminated child
- On error: `-1`

**Kernel Behavior**:

1. Parent process transitions to **blocked state**
2. Parent is removed from run queue
3. Child process executes independently
4. When child calls `exit()`, OS stores exit status
5. OS transitions parent back to **ready state**
6. Parent resumes execution after `wait()` returns

### Understanding Process IDs (PIDs)

Each process has a unique identifier:

```c
pid_t getpid();   // Get current process's PID
pid_t getppid();  // Get parent's PID
```

**Key Facts**:

- PIDs are typically positive integers
- PID 1 is `init` (or `systemd` in modern systems), the first user process
- The kernel starts processes with low PIDs
- PIDs recycle after reaching maximum value

### Process States and Transitions

```
                    ┌─────────────┐
                    │   CREATED   │
                    └──────┬──────┘
                           │ fork() returns
                           ↓
    ┌────────────────────────────────────────────────┐
    │                                                │
    │                  READY STATE                   │
    │          (waiting for CPU time slice)          │
    │                                                │
    └────────────┬─────────────────────┬─────────────┘
                 │                     │
           scheduler                 wait()
           grants time               called
                 ↓                     ↓
    ┌────────────────────┐  ┌───────────────────┐
    │  RUNNING STATE     │  │  BLOCKED STATE    │
    │ (executing on CPU) │  │ (waiting for event)│
    └────────┬───────────┘  └─────────┬─────────┘
             │                        │
        exit() or                 event
        crash                     occurs
             │                        │
             └────────────┬───────────┘
                          ↓
                  ┌──────────────────┐
                  │    TERMINATED    │
                  │  (exit status    │
                  │   available)     │
                  └──────────────────┘
```

### Process Control and Signals

Beyond `fork()`, `exec()`, and `wait()`, the UNIX API provides **signals** for external process communication:

- `SIGINT` (signal 2): Interrupt signal (typically Ctrl+C)
- `SIGTSTP` (signal 20): Stop signal (typically Ctrl+Z)
- `SIGTERM` (signal 15): Termination request
- `SIGKILL` (signal 9): Forced termination (cannot be caught)

The `kill()` system call sends a signal to a process:

```c
int kill(pid_t pid, int sig);
```

Processes can register signal handlers:

```c
void handle_signal(int sig) {
    printf("Received signal %d\n", sig);
}

signal(SIGINT, handle_signal);  // Catch Ctrl+C
```

---

#### Key Takeaways

- `fork()` creates an identical child process; returns different values to parent and child
- `exec()` replaces the current process with a new program; successful calls never return
- Separation of `fork()` and `exec()` enables shell redirection, pipes, and complex process management
- `wait()` synchronizes parent and child execution, allowing parents to collect exit status
- Signals provide asynchronous communication to processes from the OS

#### Important Terms

| Term                 | Meaning & Context                                                                          |
| :------------------- | :----------------------------------------------------------------------------------------- |
| Process              | An instance of a running program with its own memory, registers, and file descriptors      |
| Process ID (PID)     | Unique integer identifier for each process; parent PID is available via `getppid()`        |
| fork()               | System call that creates a child process as a near-identical copy of the parent            |
| exec()               | Family of system calls that replaces the current process with a new program                |
| wait()               | System call that blocks parent until a child process terminates                            |
| File Descriptor (FD) | Integer handle representing an open file or I/O stream; 0=stdin, 1=stdout, 2=stderr        |
| Signal               | Asynchronous notification sent to a process; includes SIGINT (Ctrl+C), SIGTERM, SIGKILL    |
| Address Space        | Complete memory image of a process: code, data, heap, stack, kernel stack                  |
| Redirection          | Shell feature that connects process I/O to files or other processes using file descriptors |
| Pipe                 | IPC mechanism connecting stdout of one process to stdin of another                         |

---

## Chapter 2: Mechanism - Limited Direct Execution

### Overview

The CPU is a shared resource, and the OS must balance:

1. **Performance**: Allowing processes to run directly on hardware
2. **Control**: Maintaining ability to interrupt, schedule, and manage processes
3. **Isolation**: Preventing processes from interfering with each other or the kernel

Limited Direct Execution (LDE) solves this through hardware support for **privilege levels** and **protected control transfer**.

### Hardware Privilege Levels

Modern CPUs operate in two modes:

| Mode            | Capabilities               | Example Operations                                                                       |
| :-------------- | :------------------------- | :--------------------------------------------------------------------------------------- |
| **User Mode**   | Restricted hardware access | Read/write user memory, arithmetic, logical operations                                   |
| **Kernel Mode** | Full hardware access       | Load kernel data, configure trap handlers, modify memory management, control I/O devices |

Attempting privileged operations in user mode causes a **hardware exception**, which traps into the OS.

### The Limited Direct Execution Protocol

The OS uses a three-phase protocol to safely execute user programs:

#### Phase 1: Boot-Time Setup (Privileged)

```
Operating System @ Boot
  │
  ├─→ [PRIVILEGED] Initialize trap table
  │   - Map trap handlers to hardware
  │   - Store locations in CPU
  │
  ├─→ [PRIVILEGED] Load interrupt handlers
  │   - SIGINT handler
  │   - Timer interrupt handler
  │   - System call handler
  │
  └─→ [PRIVILEGED] Start timer device
      - Program for periodic interrupts
```

**What Happens**: The OS runs in kernel mode and configures hardware to know where the trap table lives. The trap table is essentially a vector of kernel functions:

```
Trap Table Entry 0: handle_division_by_zero()
Trap Table Entry 1: handle_invalid_memory_access()
...
Trap Table Entry 60: handle_system_call()
Trap Table Entry 64 (timer): handle_timer_interrupt()
```

#### Phase 2: Runtime Execution (Limited Direct)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         LIMITED DIRECT EXECUTION TIMELINE               │
└─────────────────────────────────────────────────────────────────────────┘

OS (Kernel Mode)          |  Hardware              |  User Program
──────────────────────────┼────────────────────────┼─────────────────
                          |                        |
1. Allocate process       |                        |
   control block          |                        |
                          |                        |
2. Allocate memory        |                        |
                          |                        |
3. Load program into      |                        |
   memory                 |                        |
                          |                        |
4. Setup kernel stack     |                        |
   with saved registers   |                        |
                          |                        |
5. [PRIVILEGED]           |                        |
   Return-from-trap       |                        |
   instruction            |                        |
                          |  Switch to:            |
                          |  - User mode           |
                          |  - PC = program entry  |
                          |                        |  Program executes
                          |                        |  directly on CPU
                          |                        |
                          |                        |  Executes instructions
                          |                        |  (arithmetic, memory)
                          |                        |
                          |                        |  Calls system call
                          |                        |
                          |                        |  trap instruction
                          |                        |  (illegal instruction
                          |                        |   or divide-by-zero)
                          |  Hardware detects      |
                          |  privileged operation  |
                          |                        |
                          |  [PRIVILEGED]          |
                          |  Save registers to     |
                          |  kernel stack          |
                          |                        |
                          |  Switch to kernel mode |
                          |                        |
6. OS resumes             |                        |
   (trap handler runs)    |                        |
                          |                        |
7. Examine trap number    |                        |
   and service request    |                        |
                          |                        |
8. [PRIVILEGED]           |                        |
   Return-from-trap       |                        |
                          |  Switch to:            |
                          |  - User mode           |
                          |  - PC = after trap     |
                          |                        |  Program resumes
                          |                        |  after system call
```

**Key Detail**: The hardware is responsible for saving registers when transitioning between user and kernel modes. This is typically done via a per-process **kernel stack**.

**Per-Process Kernel Stack**:

```
Kernel Stack
(in kernel memory,
 protected from user access)
┌──────────────────────────┐
│ Reserved for kernel      │ ← Top
├──────────────────────────┤
│ Saved User Registers     │
│ (PC, registers, flags)   │ ← Pushed by trap instruction
├──────────────────────────┤
│ Kernel Local Variables   │
│ (temporary values)       │
└──────────────────────────┘
```

### System Calls: Protected Transitions into the Kernel

#### The System Call Mechanism

A system call is a **controlled entry point** into the kernel. User programs cannot simply jump to arbitrary kernel addresses; instead, they must use the trap instruction to request a specific service by number.

```c
// User mode code
int fd = open("/home/user/file.txt", O_RDONLY);
```

**Under the Hood**:

1. **User Code Prepares Arguments**:
   - System call number loaded into designated register (e.g., `rax` on x86)
   - Arguments placed in specified registers/stack locations
   - Example: `open()` is system call #2 on some systems

2. **trap Instruction Executed**:

   ```
   Hardware Actions:
   - Verify CPU in user mode (if not, exception!)
   - Save PC, flags, and general-purpose registers
   - Push values onto kernel stack
   - Switch to kernel mode
   - Jump to kernel's trap handler (from trap table)
   ```

3. **Kernel's Trap Handler Runs**:

   ```c
   void trap_handler() {
       int syscall_num = get_syscall_number();  // From register

       switch(syscall_num) {
           case SYS_open:
               return sys_open(arg1, arg2, arg3);
           case SYS_read:
               return sys_read(arg1, arg2, arg3);
           case SYS_write:
               return sys_write(arg1, arg2, arg3);
           // ... more system calls
       }
   }
   ```

4. **return-from-trap Instruction**:

   ```
   Hardware Actions:
   - Restore all saved registers from kernel stack
   - Restore PC to point after trap instruction
   - Switch back to user mode
   - Jump back to user program
   ```

5. **User Program Continues**:
   - System call return value is in designated register
   - User program checks for errors and continues

#### System Call Security

The indirection through system call numbers provides protection:

**Without Protection** (VERY BAD):

```
User program could do:
jump 0xdeadbeef  // Jump to arbitrary kernel code
// Result: User code could read/write anything or crash kernel
```

**With Protection** (UNIX approach):

```
User program must do:
syscall_number = 2;  // open() is syscall #2
rax = 2;             // Place in rax register
trap                 // Controlled entry
// OS validates and executes only the intended function
```

### Managing Timer Interrupts: Regaining Control

#### The Problem

If a process runs directly on the CPU and never yields control, the OS cannot intervene:

```
Problem Scenario:
  │
  ├─→ OS loads Process A onto CPU
  │   (OS stops running)
  │
  ├─→ Process A enters infinite loop:
  │   while(1) { x++; }
  │
  └─→ OS is now stuck: cannot schedule other processes,
      handle I/O, or manage system
```

#### Solution: Timer Interrupts

The hardware timer provides an **asynchronous interrupt** mechanism:

**Timer Setup** (at OS boot):

```c
// Privileged operation
void setup_timer() {
    // Tell timer device to interrupt every 10ms
    timer_init(10);  // 10 milliseconds
}
```

**Timer Interrupt Flow**:

```
┌────────────────────────────────────────────────────────────┐
│                   TIMER INTERRUPT MECHANISM                │
└────────────────────────────────────────────────────────────┘

Process A running on CPU
(User mode, direct execution)
│
├─→ Instructions execute: arithmetic, memory ops
│
├─→ Time passes...
│
└─→ Hardware timer 🔔 triggers

Hardware Interrupt:
├─→ Save PC, registers to kernel stack (automatic)
├─→ Switch to kernel mode (automatic)
└─→ Jump to timer interrupt handler (from trap table)

OS Timer Handler:
├─→ Perform any accounting/bookkeeping
├─→ Decide: keep running Process A, or run Process B?
├─→ If context switch needed:
│   ├─→ Save current process's state
│   └─→ Restore another process's state
└─→ Execute return-from-trap

Return to User Mode:
├─→ Restore registers from kernel stack
├─→ Switch to user mode
└─→ Resume execution (may be different process!)
```

**Critical Insight**: The timer interrupt is the mechanism that prevents a single runaway process from monopolizing the CPU.

### Context Switching

#### Definition

A **context switch** is the act of stopping one process and starting another. The "context" is the complete process state: registers, PC, and memory mappings.

#### Context Switch Overhead

**What Gets Saved**:

```c
struct ProcessContext {
    uint64_t rax, rbx, rcx, rdx;          // General purpose registers
    uint64_t rsi, rdi, rbp, rsp;          // Special purpose registers
    uint64_t r8, r9, r10, r11, r12, r13;  // Extended registers
    uint64_t r14, r15;
    uint64_t rflags;                      // Flags register
    uint64_t rip;                         // Program counter
};
```

**Costs**:

1. **Direct Costs**:
   - Saving registers to kernel stack: ~100s of CPU cycles
   - Restoring registers from kernel stack: ~100s of CPU cycles
   - Flushing CPU caches: thousands of cycles (if TLBs/caches invalidate)

2. **Indirect Costs**:
   - New process's memory accesses may miss CPU caches
   - Translation Lookaside Buffer (TLB) entries from old process are now irrelevant
   - New process must rebuild caches from scratch

**Measurement**: On modern systems, a context switch typically costs 1-10 microseconds, depending on cache behavior.

#### Cooperative vs. Non-Cooperative Scheduling

**Cooperative Approach** (old systems like Macintosh, Xerox Alto):

```
OS trusts processes to voluntarily yield CPU:
├─→ Process makes system call
│   └─→ OS regains control
├─→ Process does illegal operation
│   └─→ OS regains control via exception
└─→ Problem: If process never yields, OS stuck!
```

**Non-Cooperative Approach** (modern systems):

```
OS uses timer interrupt to force control:
├─→ If process is cooperative: Great, uses fewer context switches
├─→ If process is not cooperative: Timer fires, OS regains control anyway
└─→ Result: OS always maintains control
```

---

#### Key Takeaways

- Limited Direct Execution achieves performance by running processes directly on hardware, while maintaining control through hardware mechanisms
- Trap instructions provide safe, controlled entry into kernel mode
- System call numbers protect the OS by preventing user programs from jumping to arbitrary kernel addresses
- Timer interrupts allow the OS to regain control from runaway processes
- Context switching stores and restores complete process state, but has measurable overhead

#### Important Terms

| Term                     | Meaning & Context                                                                                     |
| :----------------------- | :---------------------------------------------------------------------------------------------------- |
| Limited Direct Execution | OS mechanism allowing processes to run directly on CPU while maintaining control via traps and timers |
| User Mode                | CPU mode with restricted hardware access; executing processes run here                                |
| Kernel Mode              | CPU mode with full hardware access; OS code runs here                                                 |
| Trap                     | Hardware instruction (syscall) that transitions from user to kernel mode with automatic context save  |
| Trap Table               | Kernel data structure mapping exception/interrupt types to handler functions                          |
| Trap Handler             | Kernel function that services a trap (system call or exception)                                       |
| return-from-trap         | Privileged instruction that restores registers and switches back to user mode                         |
| Timer Interrupt          | Hardware mechanism that forces OS to regain control at regular intervals                              |
| Context                  | Complete process state: registers, PC, memory management settings                                     |
| Context Switch           | Process of stopping one process and starting another by saving/restoring context                      |
| Kernel Stack             | Per-process stack in kernel memory used to save registers during traps                                |

---

## Chapter 3: Scheduling - Introduction

### Overview

With the mechanisms of process management and CPU control established, we now address the **policy question**: Given multiple runnable processes, which one should the OS run next?

Scheduling policies directly impact:

- **Turnaround time**: How quickly jobs complete
- **Response time**: How quickly interactive jobs respond to user input
- **Fairness**: Whether all processes receive equal CPU time
- **System throughput**: How many jobs complete per unit time

### Workload Assumptions

Before evaluating policies, we establish simplifying assumptions about the processes (jobs) in the system:

1. Each job runs for the same amount of time
2. All jobs arrive at the same time
3. Once started, each job runs to completion (no preemption)
4. All jobs only use the CPU (no I/O operations)
5. The run-time of each job is known in advance

**Reality Check**: These assumptions are highly unrealistic:

- Real jobs have different lengths (assumption 1 violated)
- Jobs arrive throughout the day (assumption 2 violated)
- Modern OS preempts jobs (assumption 3 violated)
- Most jobs perform I/O (assumption 4 violated)
- Job lengths are unknown (assumption 5 violated)

As we progress, we'll relax these assumptions one by one.

### Scheduling Metrics

A **scheduling metric** is a quantitative measure used to evaluate scheduling policies.

#### Primary Metric: Turnaround Time

**Definition**:
$$T_{\text{turnaround}} = T_{\text{completion}} - T_{\text{arrival}}$$

Where:

- $T_{\text{completion}}$ = wall-clock time when job completes
- $T_{\text{arrival}}$ = wall-clock time when job arrives in system

**Under our assumptions** (all jobs arrive at $t=0$):
$$T_{\text{turnaround}} = T_{\text{completion}}$$

**Example**:

```
Jobs A, B, C each run 10 seconds

Timeline:
0     10    20    30
├─ A ─┤─ B ─┤─ C ─┤

Turnaround times:
A: 10 - 0 = 10 seconds
B: 20 - 0 = 20 seconds
C: 30 - 0 = 30 seconds

Average turnaround: (10 + 20 + 30) / 3 = 20 seconds
```

#### Secondary Metric: Response Time

In systems with interactive users, another metric matters:

**Definition**:
$$T_{\text{response}} = T_{\text{first\_run}} - T_{\text{arrival}}$$

Where $T_{\text{first\_run}}$ is when the job first gets CPU time.

**Example**:

```
If Process B must wait 20 seconds before first running:
T_response = 20 - 0 = 20 seconds
```

**Problem**: Optimizing for turnaround time (via FIFO) often harms response time (long jobs block interactive jobs).

### First In, First Out (FIFO) Scheduling

#### Algorithm Description

FIFO is the simplest scheduling policy:

1. Maintain a **queue** of ready processes
2. When CPU becomes available, select process at **front of queue**
3. When process completes, remove from queue; next process starts
4. New arriving processes join **end of queue**

#### FIFO Pseudocode

```c
// Global queue of processes ready to run
Queue ready_queue;

void schedule_next_process() {
    Process *next = dequeue(ready_queue);  // Remove first process

    if (next) {
        current_process = next;
        execute_on_cpu(next);
    }
}

void process_completes() {
    current_process = NULL;
    schedule_next_process();
}

void new_process_arrives(Process *p) {
    enqueue(ready_queue, p);  // Add to end of queue
}
```

#### FIFO Performance Under Ideal Assumptions

**Scenario: Three equal-length jobs (10 seconds each)**

```
Timeline:
0     10    20    30
├─ A ─┤─ B ─┤─ C ─┤

Results:
- A completes at t=10: turnaround = 10
- B completes at t=20: turnaround = 20
- C completes at t=30: turnaround = 30
- Average turnaround = 20 seconds
```

**FIFO is optimal** when all jobs have equal runtime (given our assumptions).

#### FIFO Performance: Impact of Relaxing Assumption 1

**Scenario: Jobs of different lengths**

Jobs:

- A: 100 seconds (CPU-intensive batch job)
- B: 10 seconds (short interactive job)
- C: 10 seconds (short interactive job)

All arrive at $t=0$. FIFO order: A, B, C

```
Timeline:
0      100    110    120
├──── A ────┤─ B ─┤─ C ─┤

Results:
- A completes at t=100: turnaround = 100
- B completes at t=110: turnaround = 110
- C completes at t=120: turnaround = 120
- Average turnaround = 110 seconds (POOR!)

Problem: B and C must wait entire duration of A
before starting. This is the "convoy effect."
```

**The Convoy Effect**: Short jobs get stuck behind a long job, similar to a grocery store line where a customer with many items blocks all others.

### Shortest Job First (SJF) Scheduling

#### Algorithm Description

To address the convoy effect, SJF runs jobs in order of runtime:

1. When CPU available, select **shortest remaining job**
2. Run that job to completion
3. Repeat

#### Why SJF Works

By running short jobs first:

- Short jobs complete quickly
- Long jobs don't block system
- Average turnaround improves dramatically

#### SJF Performance: Same Scenario

Jobs: A (100s), B (10s), C (10s), all arrive at $t=0$

**Optimal SJF order: B, C, A**

```
Timeline:
0    10    20          120
├ B ┤ C ┤────── A ──────┤

Results:
- B completes at t=10: turnaround = 10
- C completes at t=20: turnaround = 20
- A completes at t=120: turnaround = 120
- Average turnaround = 50 seconds (110 → 50!)
```

**Mathematical Proof**: Given our assumptions, SJF minimizes average turnaround time (proof omitted in systems course).

#### SJF Limitation: Unknown Job Lengths

In practice, the OS doesn't know job runtimes in advance. Scheduling algorithms like SJF (or STCF) would require **omniscience** — not available in real systems.

**Hypothesis**: While imperfect, an OS can **predict** future job behavior based on past behavior (central principle of MLFQ, discussed later).

### SJF with Late Arrivals

#### New Assumption: Jobs Arrive at Different Times

Now relax assumption 2: jobs can arrive at any time.

**Scenario**:

- Job A: arrives at $t=0$, runs 100 seconds
- Job B: arrives at $t=10$, runs 10 seconds
- Job C: arrives at $t=10$, runs 10 seconds

**With Non-Preemptive SJF** (A cannot be interrupted):

```
Timeline:
0        10              100   110   120
├─────── A ───────────────┤─ B ─┤─ C ─┤
         ↑
         B, C arrive here
         but A is still running

Results:
- A completes at t=100: turnaround = 100 - 0 = 100
- B completes at t=110: turnaround = 110 - 10 = 100
- C completes at t=120: turnaround = 120 - 10 = 110
- Average turnaround = 103.3 seconds

Problem: B and C must still wait for A, even though
SJF knows they're shorter!
```

**Root Cause**: Non-preemptive scheduling cannot interrupt A once started.

### Shortest Time-to-Completion First (STCF)

#### Algorithm Description

STCF solves the late-arrival problem by allowing **preemption**:

1. When new job arrives, compare its remaining time to current job's remaining time
2. If new job is shorter, **preempt** current job and run new job
3. When job completes, switch to next shortest remaining job

#### STCF vs. SJF

| Aspect                     | SJF                               | STCF                           |
| :------------------------- | :-------------------------------- | :----------------------------- |
| Preemptive?                | No                                | Yes                            |
| Can interrupt running job? | No                                | Yes                            |
| On new job arrival         | Ignore (keep current job running) | Compare remaining times        |
| Optimal for?               | All jobs arrive together          | Jobs arrive at different times |

#### STCF Performance: Same Late-Arrival Scenario

Jobs: A (100s at t=0), B (10s at t=10), C (10s at t=10)

```
Timeline:
0   10        20          30                    110
├ A ├─ B ─┤─ C ─┤────── A (remaining 90s) ────┤

Execution:
t=0-10:    A runs (no choice, B and C not here)
t=10:      B arrives; remaining: A=90, B=10, C=10
           → Preempt A, run B (shortest)
t=20:      B completes; remaining: A=90, C=10
           → Run C (10 < 90)
t=30:      C completes; remaining: A=90
           → Run A
t=120:     A completes

Results:
- A completes at t=120: turnaround = 120 - 0 = 120
- B completes at t=20: turnaround = 20 - 10 = 10 ✓
- C completes at t=30: turnaround = 30 - 10 = 20 ✓
- Average turnaround = 50 seconds
```

**Key Insight**: STCF minimizes average turnaround time across all realistic scenarios (jobs arriving at different times, preemption allowed).

**Remaining Problem**: STCF still requires knowing job lengths in advance (assumption 5 violated).

---

#### Key Takeaways

- Turnaround time measures how quickly jobs complete
- FIFO is simple but suffers from convoy effect when job lengths vary
- SJF minimizes average turnaround but requires knowing job lengths
- STCF (preemptive SJF) handles late-arriving jobs optimally but still needs job length predictions
- Real systems must predict job behavior to achieve SJF-like performance

#### Important Terms

| Term                                     | Meaning & Context                                                              |
| :--------------------------------------- | :----------------------------------------------------------------------------- |
| Scheduling Policy                        | OS algorithm determining which process runs next                               |
| Turnaround Time                          | Time from job arrival to completion; metric for batch workloads                |
| Response Time                            | Time from job arrival to first CPU execution; metric for interactive workloads |
| Convoy Effect                            | Short jobs delayed behind a long job in FIFO; reduces average turnaround       |
| FIFO (First In, First Out)               | Simplest policy; runs jobs in arrival order                                    |
| SJF (Shortest Job First)                 | Non-preemptive policy selecting shortest job; optimal for known runtimes       |
| Preemption                               | Ability to stop a running process and switch to another                        |
| STCF (Shortest Time-to-Completion First) | Preemptive version of SJF; optimal when job lengths known                      |
| Job Length (Runtime)                     | How long a process executes before completing                                  |
| Runnable (Ready)                         | Process state where job is waiting for CPU time                                |

---

## Chapter 4: Scheduling - The Multi-Level Feedback Queue (MLFQ)

### Overview

The Multi-Level Feedback Queue (MLFQ) is one of the most important scheduling algorithms in modern operating systems. It was developed by Corbata et al. in 1962 for the Compatible Time-Sharing System (CTSS) and later refined for Multics (work that won the ACM Turing Award).

**Core Problem MLFQ Solves**:

1. Optimize turnaround time (achieved by running short jobs first)
2. Minimize response time for interactive users
3. Do all this **without knowing job lengths in advance**

### The Fundamental Challenge

```
Ideal Requirements (Conflicting!)
├─→ Minimize turnaround: Run short jobs first (like SJF)
├─→ Minimize response time: Run interactive jobs quickly (like RR)
└─→ Don't know job lengths: Must learn from behavior
```

**Key Insight**: MLFQ learns job characteristics by **observing runtime behavior** and adjusting priorities accordingly.

### MLFQ: Basic Rules

#### Rule 1: Priority-Based Selection

```
If Priority(Process A) > Priority(Process B)
    → Run A (ignore B)
```

Processes are organized in multiple queues, each with a priority level:

```
Priority Queue Structure:

[High Priority] Q8  ┌────────────┬────────────┐
                    │ ProcessA   │ ProcessB   │
                Q7  ├────────────┤
                    │ (empty)    │
                Q6  ├────────────┤
                    │ (empty)    │
                Q5  ├────────────┤
                    │ (empty)    │
                Q4  ├────────────┼────────────┐
                    │ ProcessC   │ ProcD      │
                Q3  ├────────────┤
                    │ (empty)    │
                Q2  ├────────────┤
                    │ (empty)    │
[Low Priority] Q1   ├────────────┬────────────┐
                    │ ProcessE   │ ProcessF   │
```

#### Rule 2: Round-Robin Among Equal Priorities

```
If Priority(A) = Priority(B)
    → Run A and B in round-robin (each gets time slice, then switch)
```

**Implementation**:

```c
Process* schedule() {
    for (int queue = HIGHEST_PRIORITY; queue >= 0; queue--) {
        if (!queue[queue].empty()) {
            Process *p = dequeue_from_front(queue[queue]);
            enqueue_to_back(queue[queue], p);  // Round-robin
            return p;
        }
    }
    return NULL;  // No runnable process
}
```

### How MLFQ Adjusts Priorities

The key to MLFQ is **dynamic priority adjustment** based on observed behavior.

#### Rule 3: New Jobs Enter at Highest Priority

```
When a new process enters the system:
    → Place it at Q_max (highest priority)
```

**Rationale**:

- If it's a short interactive job, it will complete quickly and leave
- If it's a long batch job, it will be detected and moved down
- Initially treat all jobs as potentially interactive

#### Rule 4a: Using Full Time Slice → Lower Priority

```
If a process uses entire time slice (time quantum):
    → Reduce priority by 1 (move down one queue)
```

**Rationale**:

- Process consumed full time slice → likely CPU-intensive
- CPU-intensive jobs should have lower priority
- Keep high priority for jobs that yield CPU

#### Rule 4b: Yield CPU Early → Keep Priority

```
If a process yields CPU before time slice ends:
    → Keep at same priority level
```

**Rationale**:

- Process yielded CPU (waiting for I/O) → likely interactive
- Interactive jobs should stay at high priority
- Example: keyboard input, mouse click

#### Example 1: Long-Running Batch Job

```
Process A: CPU-intensive, will run for 100+ seconds
Time quantum per slice: 10 ms
Queue levels: 3 (Q2=highest, Q1=middle, Q0=lowest)

Timeline:

t=0ms:   A arrives
         │
         └─→ [Rule 3] A placed at Q2

t=10ms:  A's 10ms time slice expires
         │
         └─→ [Rule 4a] A used full slice
             └─→ Move A to Q1

t=20ms:  A's next slice expires (now on Q1)
         │
         └─→ [Rule 4a] A used full slice again
             └─→ Move A to Q0

t=30ms:  A's next slice expires (now on Q0)
         │
         └─→ [Rule 4a] A used full slice again
             └─→ A stays at Q0 (lowest)

Result: After 30ms, A has settled to lowest priority
```

**ASCII Timeline**:

```
Priority
   ↑ Q2 ▓▓▓░░░░░░
   │ Q1 ░░▓▓▓░░░░░
   │ Q0 ░░░░░▓▓▓▓▓▓
   └─────────────────→ Time (ms)
     0 10 20 30 40 50

▓ = Process running   ░ = Process not running
```

#### Example 2: Short Interactive Job Arrives

```
Process A has been running on Q0 for a while (long-running job)
Process B arrives (short interactive job, yields CPU for I/O)

Timeline:

t=100ms: B arrives
         │
         └─→ [Rule 3] B placed at Q2

t=110ms: B's time slice expires
         │ (but B only used 5ms, then did I/O read)
         │
         └─→ [Rule 4b] B yielded CPU before slice ended
             └─→ B stays at Q2

t=120ms: B's time slice expires again
         │ (B used 3ms, then did I/O write)
         │
         └─→ [Rule 4b] B yielded CPU before slice ended
             └─→ B stays at Q2

Result: B stays at high priority because of I/O behavior pattern
```

**Scheduler Behavior**:

```
CPU Allocation (each line = 10ms interval):

Q2: ▓B... ▓B... ▓B... ▓B...    (B gets CPU frequently, waits for I/O)
Q1: (empty)
Q0: ▓AAAA ▓AAAA A▓AAA AA▓AAA   (A gets CPU only when Q2 empty)

Time: 0 10 20 30 40 ...
```

### Problems with Basic MLFQ

The initial MLFQ rules have three critical flaws:

#### Problem 1: Starvation

**Scenario**: System has many short interactive jobs

```
Q2: B1, B2, B3 (interactive jobs, yielding CPU)
Q1: (empty)
Q0: A (long-running batch job)

Timeline:
All interactive jobs in Q2 run repeatedly (yielding for I/O).
They consume all CPU time.
Job A on Q0 never gets CPU time.
Result: A starves → never completes
```

#### Problem 2: Gaming the Scheduler

**Clever User's Attack**:

```c
// Malicious job that games MLFQ
while(1) {
    // Use 99% of time slice
    do_work_for(9.9ms);

    // Just before slice ends: yield CPU (e.g., I/O operation)
    read(99);  // Read to file we don't care about

    // Result: Rules 4b keeps us at high priority!
}
```

**Effect**: Malicious process stays at high priority despite being CPU-intensive, essentially monopolizing CPU.

#### Problem 3: Behavior Changes

**Scenario**: Process changes from batch to interactive

```
Phase 1: Process is CPU-intensive (t=0-30s)
→ Demoted to Q0 (lowest priority)

Phase 2: Process changes to I/O-intensive (t=31s+)
→ Process now yields CPU frequently
→ But still stuck at Q0 (low priority)

Result: Process gets low priority despite being interactive
```

### MLFQ: Improved Rule 1 - Priority Boost

To solve starvation and handle behavior changes, add periodic "reset":

#### Rule 5: Periodic Priority Boost

```
Every S seconds:
    → Move all processes to highest priority (Q_max)
```

**Effect**:

```
1. Guarantees no starvation: Every process gets CPU time periodically
2. Handles behavior changes: CPU-intensive job moved to high priority,
   can demonstrate if it changed to interactive
3. Fair fallback: Even if prediction is wrong, reset periodically
```

**Starvation Prevention Timeline**:

```
WITHOUT Priority Boost:
Q2: ▓B1 ▓B2 ▓B3 ▓B1 ▓B2 ▓B3  (interactive jobs)
Q0: A   A   A   A   (starves!)

S = 50ms boost interval
WITH Priority Boost:
Q2: ▓B1 ▓B2 ▓B3 ▓B1 ▓A  ▓B2 ▓B3 ▓A  ▓B1 ▓A
Q0: (gets reset periodically)

Result: A makes regular progress
```

#### Problem: "Voodoo Constants"

The question arises: **What should S be?**

John Ousterhout called such parameters "voodoo constants" — they seem to require black magic to set correctly:

- **Too high** (e.g., S=1000s): Long-running jobs starve between boosts
- **Too low** (e.g., S=1ms): Interactive jobs don't get fair allocation; too many context switches
- **Sweet spot** (e.g., S=50-100ms): System-dependent, must tune experimentally

This is a real problem in MLFQ tuning; there's no perfect value.

### MLFQ: Improved Rule 2 - Better Accounting

To prevent gaming, improve how time is accounted:

#### Rule 4 Revised: Track CPU Use in Each Priority Level

```
For each process, track:
    - cpu_used[i] = CPU time used while at priority level i
    - time_quantum[i] = maximum time allowed at level i

When cpu_used[i] ≥ time_quantum[i]:
    → Move to next lower priority
    → Reset cpu_used counter
```

**Example**:

```
Time quantums:
Q2: 10ms per slice
Q1: 20ms per slice
Q0: 40ms per slice (batch jobs get longer slices)

Malicious Process Behavior:
t=0ms:    Enters at Q2, cpu_used[2]=0
t=5ms:    Yields (I/O) → stays at Q2, cpu_used[2]=5
t=5.1ms:  Enters at Q2 again, cpu_used[2]=5
t=9.9ms:  CPU time reaches 9.9ms
t=10ms:   cpu_used[2] ≥ 10ms
          └─→ Move to Q1, reset cpu_used[1]=0

Result: Can't stay at Q2 indefinitely by gaming!
```

**Longer Time Quantums at Lower Priorities**:

Why increase time slice for lower priority?

```
Rationale:
- Lower priority = likely batch/CPU-intensive
- Batch jobs benefit from longer slices (fewer context switches)
- Reducing preemption frequency improves throughput
- Interactive jobs rarely reach low priority (and if they do,
  they'll be boosted back up eventually)
```

### Complete MLFQ Algorithm Summary

**Final Rule Set**:

| Rule Number | Description                                                                                   |
| :---------- | :-------------------------------------------------------------------------------------------- |
| 1           | If Priority(A) > Priority(B), run A                                                           |
| 2           | If Priority(A) = Priority(B), round-robin among them                                          |
| 3           | New job enters at highest priority                                                            |
| 4           | If job uses full time slice, lower priority; if yields early, stay same priority              |
| 5           | Every S seconds, move all jobs to highest priority (prevent starvation, handle phase changes) |

**Data Structure**:

```c
struct MLFQ {
    // Multiple priority queues
    Queue queues[NUM_PRIORITY_LEVELS];  // Q0 (low) to Q_max (high)

    // Per-process accounting
    struct Process {
        int priority_level;
        int cpu_used_this_level;
        int time_quantum;        // Time slice duration
        int arrival_time;
        int cpu_total;
    }* processes;

    // Parameters
    int time_slice[NUM_PRIORITY_LEVELS];  // Quantum at each level
    int boost_interval;                    // S (seconds between boosts)
};
```

**Pseudocode**:

```c
void mlfq_scheduler() {
    while (true) {
        // Check if priority boost time
        if (time_this_period() == boost_interval) {
            boost_all_to_highest_priority();
            reset_cpu_accounting();
        }

        // Find highest-priority non-empty queue
        for (int level = NUM_LEVELS - 1; level >= 0; level--) {
            if (!queues[level].empty()) {
                Process *p = dequeue(queues[level]);

                // Give process a time slice
                run_for_timeslice(p, time_slice[level]);

                if (process_completed(p)) {
                    // Job done, don't re-queue
                    free(p);
                } else if (cpu_used[p->level] >= time_slice[p->level]) {
                    // Used full slice, lower priority
                    p->priority_level--;
                    p->cpu_used_this_level = 0;
                    if (p->priority_level < 0) p->priority_level = 0;
                    enqueue(queues[p->priority_level], p);
                } else {
                    // Yielded early, stay at same priority
                    enqueue(queues[p->priority_level], p);
                }
                break;  // Next iteration will check for higher priority jobs
            }
        }
    }
}
```

### MLFQ in Practice

**Linux CFS (Completely Fair Scheduler)**: Modern Linux dropped MLFQ in favor of the Completely Fair Scheduler, which attempts to give all processes fair CPU allocation using different mechanisms.

**BSD/macOS**: Traditional MLFQ with multiple 32-level priority queues.

**Windows**: Hybrid approach with multi-level queues and dynamic priority adjustments.

---

#### Key Takeaways

- MLFQ learns job characteristics by observing behavior (CPU vs. I/O patterns)
- Promotion (via boosts) prevents starvation and handles behavior changes
- Demotion (via time quantum accounting) prevents gaming and prioritizes interactive jobs
- MLFQ represents a practical balance between knowing nothing and omniscience
- Trade-off: System must be tuned; optimal parameters depend on workload

#### Important Terms

| Term                              | Meaning & Context                                                                           |
| :-------------------------------- | :------------------------------------------------------------------------------------------ |
| MLFQ (Multi-Level Feedback Queue) | Scheduling algorithm with multiple priority queues; promotes/demotes jobs based on behavior |
| Priority Level                    | Queue number (Q0=lowest to Q_max=highest); determines scheduling preference                 |
| Time Quantum (Time Slice)         | Maximum CPU time a process gets before preemption                                           |
| Feedback                          | OS observes process behavior and adjusts priority accordingly                               |
| Priority Boost                    | Periodic reset of all processes to highest priority (prevents starvation)                   |
| Starvation                        | Process never receives CPU time; indefinite postponement                                    |
| Gaming the Scheduler              | Malicious practice of tricking scheduler into giving unfair advantage                       |
| CPU-Intensive (Batch) Job         | Process that uses most/all of time slices; needs CPU continuously                           |
| I/O-Intensive (Interactive) Job   | Process that yields CPU for I/O; needs responsiveness                                       |
| Behavior Learning                 | MLFQ predicts future by observing past (if yielded before, probably interactive)            |

---

## Chapter 5: Scheduling - Proportional Share (Lottery Scheduling)

### Overview

Previous scheduling policies (FIFO, SJF, STCF, MLFQ) focus on **optimizing average metrics**:

- Minimize average turnaround time
- Minimize average response time
- Fairly distribute CPU among interactive and batch jobs

An alternative approach: **Proportional Share (Lottery Scheduling)** ensures each process gets a **predictable percentage of CPU time** over time.

**Key Principle**: Rather than trying to be "fair" through complex rules, lottery scheduling is mathematically fair: each process's CPU share equals its ticket percentage.

### The Lottery Scheduling Concept

#### Basic Idea: Tickets Represent CPU Share

```
Suppose we have 400 total tickets distributed:
├─ Process A: 100 tickets (25% of CPU)
├─ Process B: 50 tickets  (12.5% of CPU)
└─ Process C: 250 tickets (62.5% of CPU)

Total: 400 tickets

Each scheduling decision:
1. Generate random number 0 to 399
2. Find which process "owns" that ticket
3. Run that process for one time slice
4. Repeat

Probability each gets selected:
- A: 100/400 = 25%
- B: 50/400 = 12.5%
- C: 250/400 = 62.5%
```

#### Why It Works: Law of Large Numbers

Individual scheduling decisions are random, but over many decisions, each process gets approximately its ticket fraction of CPU:

```
Over 100 scheduling decisions:
- A gets ~25 turns
- B gets ~12 turns
- C gets ~63 turns

Over 1,000,000 scheduling decisions:
- A gets ~250,000 turns
- B gets ~125,000 turns
- C gets ~625,000 turns

(Closer to exact ticket fraction)
```

### Lottery Scheduling Implementation

#### The Core Algorithm

```c
// Global state
struct {
    Process *processes[MAX_PROCS];
    int tickets[MAX_PROCS];
    int num_procs;
    int total_tickets;
} scheduler;

Process* lottery_schedule() {
    // Step 1: Pick random ticket from 0 to total_tickets-1
    int winner = random(0, scheduler.total_tickets - 1);

    // Step 2: Walk process list, accumulating tickets
    int counter = 0;
    for (int i = 0; i < scheduler.num_procs; i++) {
        counter += scheduler.tickets[i];
        if (counter > winner) {
            // Found winner
            return scheduler.processes[i];
        }
    }

    // Should never reach here
    return NULL;
}
```

**Example Walkthrough**:

```
Processes: A (100 tix), B (50 tix), C (250 tix)
Random number generated: 300

i=0: counter = 0 + 100 = 100; is 100 > 300? No
i=1: counter = 100 + 50 = 150; is 150 > 300? No
i=2: counter = 150 +250 = 400; is 400 > 300? Yes
     └─→ Return C

Result: C runs this time slice
```

#### Time Complexity

- **Picking random number**: O(1)
- **Walking process list**: O(n) where n = number of processes
- **Total per scheduling decision**: O(n)

This is reasonable even for systems with hundreds of processes.

### Ticket Mechanisms: Manipulating Shares

Lottery scheduling is flexible; processes can manipulate tickets:

#### Ticket Currency: Delegation Within User Domain

**Problem**: How do users A and B fairly share 200 tickets across their jobs?

**Solution**: Each user gets "currency" to allocate to their own jobs. System converts to global tickets.

**Example**:

```
System assigns:
├─ User A: 100 global tickets (50% of CPU)
└─ User B: 100 global tickets (50% of CPU)

User A runs 2 jobs: A1, A2
├─ A1: 600 A-currency tickets
└─ A2: 400 A-currency tickets
Total: 1000 A-currency

User B runs 1 job: B1
└─ B1: 10 B-currency tickets
Total: 10 B-currency

System converts to global:
├─ A1: 600/1000 × 100 = 60 global tickets
├─ A2: 400/1000 × 100 = 40 global tickets
└─ B1: 10/10 × 100 = 100 global tickets

Effective CPU allocation:
├─ A1: 60/300 = 20% of CPU
├─ A2: 40/300 ≈ 13% of CPU
└─ B1: 100/300 ≈ 33% of CPU

Result: A maintains 50% CPU (split between jobs),
        B maintains 50% CPU (one job)
```

#### Ticket Transfer: Temporary Delegation

**Scenario**: Client-server system where client blocks waiting for server

```
Without transfer:
├─ Client: 100 tickets (running I/O-bound code waiting for server)
└─ Server: 50 tickets (running server code)
Problem: Server only gets 50 tickets, can't serve client quickly

With transfer:
├─ Client: passes all 100 tickets to Server temporarily
└─ Server: 50 + 100 = 150 tickets
Result: Server gets priority boost while serving client
```

**Code Pattern**:

```c
// Client process
tickets_transfer(server_pid, my_tickets);
send_request_to_server(data);
wait_for_response();           // Blocks here
tickets_return(server_pid);    // Server returns tickets
```

**Benefit**: Server can process request quickly, unblocking client faster.

#### Ticket Inflation: Temporary Priority Increase

In **cooperative** environments (processes trust each other), a process can temporarily increase its tickets:

```c
// Process knows it needs more CPU (e.g., video rendering)
current_tickets *= 2;    // Double my tickets temporarily
do_intensive_work();
current_tickets /= 2;    // Return to normal
```

**Risk**: In adversarial environment (untrusted processes), could monopolize CPU by inflating infinitely.

### Lottery Scheduling: Advantages and Disadvantages

#### Advantages

| Advantage         | Explanation                                                       |
| :---------------- | :---------------------------------------------------------------- |
| **Predictable**   | Given tickets, CPU share is mathematically guaranteed (over time) |
| **Simple**        | Algorithm easy to understand and implement                        |
| **Flexible**      | Processes can exchange, transfer, or modify tickets               |
| **No starvation** | Every ticket-holder gets some CPU, eventually                     |
| **Responsive**    | Short jobs with few tickets still run (won't starve)              |

#### Disadvantages

| Disadvantage                      | Explanation                                                     |
| :-------------------------------- | :-------------------------------------------------------------- |
| **Not optimal for turnaround**    | Random selection doesn't minimize average turnaround (cf. SJF)  |
| **Not optimal for response time** | Random selection doesn't prioritize interactive jobs (cf. MLFQ) |
| **Random variation**              | Short-term behavior is unpredictable (not deterministic)        |
| **Ticket allocation complexity**  | Deciding initial ticket allocation is non-obvious               |
| **Efficiency assumption**         | Assumes processes don't care getting exact share vs. fair share |

### Stride Scheduling: Deterministic Alternative

**Limitation of Lottery**: Randomness means short-term CPU allocation is unpredictable.

**Stride Scheduling** provides deterministic proportional share:

#### Algorithm Concept

Instead of random draws, each process has a **stride** (inverse of ticket count), and the process with smallest "pass value" runs next.

```c
struct Process {
    int tickets;
    int stride;      // Stride = large_constant / tickets
    int pass_value;  // Track accumulated passes
};

void stride_init(Process *p, int tickets) {
    p->tickets = tickets;
    p->stride = LARGE_NUMBER / tickets;
    p->pass_value = 0;
}

Process* stride_schedule() {
    // Find process with minimum pass_value
    Process *min_process = NULL;
    int min_pass = INT_MAX;

    for (int i = 0; i < num_procs; i++) {
        if (processes[i].pass_value < min_pass) {
            min_pass = processes[i].pass_value;
            min_process = processes[i];
        }
    }

    // Run that process, then increment its pass value
    if (min_process) {
        run_process(min_process, time_slice);
        min_process->pass_value += min_process->stride;
    }

    return min_process;
}
```

#### Example: Stride Scheduling

```
Processes: A (100 tix), B (50 tix)
Large constant: 10,000

Init:
├─ A: stride = 10,000/100 = 100, pass = 0
└─ B: stride = 10,000/50 = 200, pass = 0

Scheduling sequence:

t=0:   A.pass=0 < B.pass=0? No (tie, break arbitrarily)
       Run A
       A.pass = 0 + 100 = 100

t=1:   A.pass=100 < B.pass=0? No
       Run B
       B.pass = 0 + 200 = 200

t=2:   A.pass=100 < B.pass=200? Yes
       Run A
       A.pass = 100 + 100 = 200

t=3:   A.pass=200 < B.pass=200? No (tie)
       Run A (arbitrary)
       A.pass = 200 + 100 = 300

t=4:   A.pass=300 < B.pass=200? No
       Run B
       B.pass = 200 + 200 = 400

Pattern: ABABAA BABAA ...
CPU share: A gets 2/3, B gets 1/3
Expected: A should get 100/150 = 2/3, B should get 50/150 = 1/3 ✓
```

#### Stride vs. Lottery

| Aspect                    | Lottery                   | Stride                            |
| :------------------------ | :------------------------ | :-------------------------------- |
| Guarantee                 | Probabilistic (over time) | Deterministic (exact ratio)       |
| Short-term predictability | Random                    | Predictable                       |
| Implementation complexity | Very simple               | Slightly more complex             |
| Flexibility               | Good (transfer tickets)   | Limited (modifying stride harder) |
| Practical OS use          | Less common               | Found in some research systems    |

---

#### Key Takeaways

- Lottery scheduling is alternative to priority-based scheduling
- Mathematical guarantee: CPU share equals ticket fraction
- Ticket mechanisms (currency, transfer, inflation) provide flexibility
- Stride scheduling improves on lottery with deterministic behavior
- Trade-off: Proportional share simplicity vs. optimizing specific metrics

#### Important Terms

| Term               | Meaning & Context                                                   |
| :----------------- | :------------------------------------------------------------------ |
| Lottery Scheduling | Scheduler giving process fraction of CPU proportional to tickets    |
| Tickets            | Abstract currency representing process's share of CPU               |
| Proportional Share | Guarantee that process gets certain percentage of CPU over time     |
| Ticket Currency    | Process group's local tickets, converted to global tickets          |
| Ticket Transfer    | Temporarily passing tickets to another process (esp. client-server) |
| Ticket Inflation   | Temporarily increasing tickets (only in cooperative environments)   |
| Stride             | Inverse of ticket count; used in deterministic stride scheduling    |
| Pass Value         | Running total incremented by stride; determines scheduling order    |
| Fairness           | Giving each process predictable, measurable CPU allocation          |

---

## Chapter 6: Comprehensive Scheduling Algorithms Summary

### Historical Timeline and Evolution

```
Timeline of Scheduler Development:

1950s-1960s
├─ FIFO / First-Come-First-Served (batch processing era)
│  └─ Problem: long jobs block system
│
1960s
├─ SJF and STCF (short job first and preemptive version)
│  └─ Problem: requires knowing job length in advance
│
├─ Round-Robin (time-sharing era)
│  └─ Problem: doesn't prioritize interactive jobs
│
1962
└─ MLFQ (Compatible Time-Sharing System)
   └─ Learning approach: no prior knowledge needed

1970s-1980s
├─ Priority Queues (refined MLFQ implementations)
│  └─ Multiple priority levels, longer quanta at lower levels
│
├─ Lottery Scheduling (proportional share research)
│  └─ Alternative: random selection based on tickets
│
└─ Stride Scheduling (deterministic proportional share)
   └─ Improvement: eliminate randomness

1990s-2000s
├─ Completely Fair Scheduler concept (CFS, Linux)
│  └─ Fair on per-task CPU time, not on interactions
│
├─ Virtual Time / Weighted Fair Queuing
│  └─ Fine-grained fairness tracking
│
└─ Multi-processor scheduling
   └─ Load balancing across cores

2000s-Now
├─ CFS (Completely Fair Scheduler, Linux)
├─ macOS: Hybrid thread priority scheduler
├─ Windows: Priority queue with dynamic adjustments
├─ Hybrid multi-queue approaches
└─ Energy-aware scheduling (mobile, data centers)
```

### Comprehensive Scheduling Algorithm Comparison

#### Feature Comparison Table

| Algorithm   | Year  | Preemptive | Requires Known Length | Interactive Friendly | Batch Friendly                 | Starvation Risk            |
| :---------- | :---- | :--------- | :-------------------- | :------------------- | :----------------------------- | :------------------------- |
| FIFO/FCFS   | 1950s | No         | No                    | ✗ Poor               | ✗ Convoy effect                | No                         |
| SJF         | 1960s | No         | **Yes**               | ○ Fair               | ✓ Excellent                    | Yes (when new jobs arrive) |
| STCF        | 1960s | ✓ Yes      | **Yes**               | ✓ Good               | ✓ Excellent                    | Yes (when new jobs arrive) |
| Round-Robin | 1960s | ✓ Yes      | No                    | ✓ Very good          | ✗ Poor (many context switches) | No                         |
| MLFQ        | 1962  | ✓ Yes      | No                    | ✓ Excellent          | ✓ Good                         | No (with priority boost)   |
| Lottery     | 1990s | ✓ Yes      | No                    | ✓ Fair               | ✓ Fair                         | No                         |
| Stride      | 1990s | ✓ Yes      | No                    | ○ Fair               | ○ Fair                         | No                         |
| CFS (Linux) | 2007  | ✓ Yes      | No                    | ✓ Fair               | ✓ Fair                         | No                         |

#### Performance Metrics Comparison

| Algorithm   | Avg Turnaround              | Response Time       | Context Switches               | Fairness                       |
| :---------- | :-------------------------- | :------------------ | :----------------------------- | :----------------------------- |
| FIFO        | **Poor** (convoy)           | Very poor (convoy)  | Minimal                        | Poor                           |
| SJF         | **Best** (if lengths known) | Good                | Minimal                        | Good                           |
| STCF        | **Best** (preemptive SJF)   | Good                | Moderate                       | Good                           |
| Round-Robin | Poor                        | **Good**            | Many (high overhead)           | Excellent                      |
| MLFQ        | Good (adaptive)             | **Good** (adaptive) | Moderate                       | Good (adaptive)                |
| Lottery     | Fair                        | Fair                | Moderate                       | **Guaranteed** (probabilistic) |
| Stride      | Fair                        | Fair                | Moderate                       | **Guaranteed** (deterministic) |
| CFS         | Fair                        | Good                | Many (tracking small vruntime) | **Guaranteed** (proportional)  |

#### Complexity Analysis

| Algorithm   | Time Complexity (pick next) | Space Complexity | Notes                                                   |
| :---------- | :-------------------------- | :--------------- | :------------------------------------------------------ |
| FIFO        | O(1)                        | O(n)             | Simple queue                                            |
| SJF         | O(n)                        | O(n)             | Must scan all, find shortest                            |
| STCF        | O(n)                        | O(n)             | On arrival, rescan for shortest remaining               |
| Round-Robin | O(1)                        | O(n)             | Simple queue, each gets turn                            |
| MLFQ        | O(m)                        | O(n + m)         | m = number of priority levels; usually O(1) in practice |
| Lottery     | O(n)                        | O(n)             | Accumulate tickets until winner found                   |
| Stride      | O(n)                        | O(n)             | Find minimum pass value                                 |
| CFS         | O(log n)                    | O(n)             | Red-black tree of vruntime values                       |

### Detailed Algorithm Specification

#### FIFO (First In, First Out)

**Best For**: Batch systems, all jobs known in advance

**How It Moves**: Queue only

```
Input queue: [A(100), B(10), C(10)]
│
├─ Run A, 0-100ms
├─ Run B, 100-110ms
├─ Run C, 110-120ms
│
Results: Avg turnaround = 110ms (high!)
```

**Pros**:

- Minimal overhead
- Trivial to implement

**Cons**:

- Convoy effect (short jobs behind long jobs)
- Unresponsive system
- Poor average turnaround with mixed workload

---

#### SJF (Shortest Job First)

**Best For**: Batch systems when job lengths known; theoretical comparison

**How It Moves**: Move up to front of queue based on length

```
Input jobs: A(100), B(10), C(10)
│
Sort by length: B(10), C(10), A(100)
│
├─ Run B, 0-10ms
├─ Run C, 10-20ms
├─ Run A, 20-120ms
│
Results: Avg turnaround = 50ms (much better!)
```

**Pros**:

- Minimizes average turnaround (optimal under assumptions)
- Simple to understand

**Cons**:

- **Requires knowing job length** (unrealistic)
- Doesn't handle late arrivals well
- Starvation risk (long jobs starved by stream of short jobs)

---

#### STCF (Shortest Time-to-Completion First)

**Best For**: Batch systems when lengths known and preemption allowed

**How It Moves**: Preempt current job if new shorter job arrives

```
Jobs:
- A: 100s, arrives t=0
- B: 10s, arrives t=10
- C: 10s, arrives t=10

Execution:
t=0-10:   Run A (B, C not here yet)
t=10:     B, C arrive
          Preempt A (remaining: A=90, B=10, C=10)
          Run B (shortest)
t=20:     B done; remaining: A=90, C=10
          Run C
t=30:     C done; remaining: A=90
          Run A
t=120:    Done

Results: Avg turnaround = 50ms
         A: 120  B: 10  C: 20
```

**Pros**:

- Handles late arrivals
- Optimal turnaround with preemption
- Better responsiveness than non-preemptive SJF

**Cons**:

- **Requires knowing job length**
- High context switch overhead (frequent preemptions)
- Still requires perfect knowledge

---

#### Round-Robin (RR)

**Best For**: Time-sharing systems, interactive workloads

**How It Moves**: Rotate through all runnable jobs

```
Jobs: A, B, C (all infinite)
Time quantum: 10ms

Schedule:
t=0-10:   A runs
t=10-20:  B runs
t=20-30:  C runs
t=30-40:  A runs
...

Each job gets 10ms per 30ms interval
CPU share: Each gets 1/3
```

**Pros**:

- **Fair** (all jobs get equal shares)
- Good response time
- No starvation

**Cons**:

- High context switches (overhead)
- Turnaround time not optimized
- Interactive jobs don't get better service than batch

---

#### MLFQ (Multi-Level Feedback Queue)

**Best For**: General-purpose OS (Unix, Linux variants, macOS, Windows)

**How It Moves**: Dynamic priority based on observed behavior

```
Initial state:
Q2 (high): [A]
Q1 (med):  []
Q0 (low):  []

After 10ms (A uses full slice):
Q2: []
Q1: [A]
Q0: []

Interactive job B arrives at t=20:
Q2: [B]
Q1: [A]
Q0: []

B yields (I/O) after 3ms:
Q2: [B]
Q1: [A]
Q0: []

At t=50, priority boost resets all:
Q2: [A, B]
Q1: []
Q0: []
```

**Pros**:

- Works without knowing job lengths
- Adaptive: learns interactive vs. batch
- **Standard in modern OS** (with refinements)
- No pure starvation (priority boost)

**Cons**:

- Complex (many parameters to tune)
- Can be gamed (yield before time slice ends)
- Voodoo constant S (boost interval)
- Still not provably optimal for any metric

---

#### Lottery Scheduling

**Best For**: Proportional CPU sharing; research/specialized systems

**How It Moves**: Random selection based on ticket count

```
Tickets: A=100, B=50, C=250 (total=400)
Random number generator picks 0-399

t=0:  Pick 150 → B's range, run B
t=1:  Pick 300 → C's range, run C
t=2:  Pick 50  → A's range, run A
t=3:  Pick 350 → C's range, run C
...

Over many draws:
A runs ~25% (100/400)
B runs ~12.5% (50/400)
C runs ~62.5% (250/400)
```

**Pros**:

- Mathematically fair (proportional guarantee)
- No starvation (every ticket eventually picked)
- Flexible (transfer, currency, inflate tickets)
- Simple algorithm

**Cons**:

- Doesn't optimize turnaround or response time
- Random behavior (short-term unpredictable)
- Requires careful ticket allocation
- Rarely used in mainstream OS

---

#### Stride Scheduling

**Best For**: Deterministic proportional sharing (research systems)

**How It Moves**: Pick process with minimum "pass value"

```
Tickets/Stride: A=100/100=1, B=50/200=0.5

Init: A.pass=0, B.pass=0

t=0:  min(0,0)=A, run A, A.pass=1
t=1:  min(1,0)=B, run B, B.pass=0.5
t=2:  min(1,0.5)=B, run B, B.pass=1
t=3:  min(1,1)=A, run A, A.pass=2
...

CPU share: A gets 2/3, B gets 1/3 (exact!)
```

**Pros**:

- Deterministic (reproducible scheduling)
- Exact proportional share (not probabilistic)
- Fair guarantee

**Cons**:

- More complex than lottery
- Doesn't optimize metrics like turnaround
- Rarely used in practice

---

#### CFS (Completely Fair Scheduler, Linux)

**Best For**: Modern general-purpose OS (Linux kernel 2.6.23+)

**How It Moves**: Track "virtual runtime" (vruntime); always run process with minimum

```c
struct Task {
    u64 vruntime;           // Virtual runtime
    u64 weight;             // Scheduling weight (priority-based)
};

// Scheduling decision:
// Run the task with minimum vruntime
// After time slice: vruntime += time_slice / weight
```

**Algorithm Overview**:

```
Init:
Task A: vruntime=0, weight=1 (normal priority)
Task B: vruntime=0, weight=2 (high priority)

t=0-4ms:  Run A (min vruntime)
          A.vruntime = 0 + 4/1 = 4

t=4-8ms:  Run B (min vruntime: B=0 < A=4)
          B.vruntime = 0 + 4/2 = 2

t=8-12ms: Run B (min vruntime: B=2 < A=4)
          B.vruntime = 2 + 4/2 = 4

t=12-16ms: Run A (min vruntime: A=4 = B=4, tie break)
           A vruntime = 4 + 4/1 = 8

Pattern: Higher priority (B) gets more frequent turns
Over 1000ms:
  A gets ~333ms (weight 1)
  B gets ~667ms (weight 2)
```

**Data Structure**: Red-black tree indexed by vruntime for O(log n) scheduling.

**Pros**:

- **Standard in Linux** (modern kernel)
- Fair CPU allocation (proportional to priority weight)
- Low context-switch overhead (tree-based, runs lowest vruntime via RB-tree)
- Handles priority changes smoothly

**Cons**:

- Complex implementation (red-black trees)
- Concept of "virtual time" non-intuitive
- Tuning group fairness vs. thread fairness is complex
- May not optimize for specific metrics

---

### Modern Linux Approaches (v5.x - v6.x+)

#### CFS (Completely Fair Scheduler)

**Still the default** for most CPU scheduling, with refinements.

**Key Concept**: vruntime ≈ "fair" CPU execution time

```c
// Simplified algorithm
Task with minimum vruntime gets CPU:
  vruntime < 4ms?  Always picked
  vruntime < 10ms? Very likely picked
  vruntime >= others? Picked when others sleep
```

#### Auto Group Scheduling (cgroup integration)

```
Problem: Multiple threads/processes from one app get priority

Solution: Group by parent process; fair share to groups first

Example:
Terminal window 1: 4 threads
Terminal window 2: 1 thread

Without grouping: Each thread competes equally
  => Window 2's thread gets 1/5 CPU = 20%
  => Total for window 2: 20%

With grouping:
  => Each group (window) gets 50%
  => Window 2's thread: 50% (100% within its group)
  => More responsive!
```

#### Energy-Aware Scheduling (SCHED_EAS)

Modern extension: Consider **power consumption** and **thermal state**.

```
Concepts:
- Heterogeneous CPUs (big cores, little cores)
- Power states (frequencies, sleep modes)
- Thermal limits (don't overheat)

Decision factors:
├─ Task requirements (performance vs. efficiency)
├─ CPU thermal state
├─ Current frequency
└─ Power budget

Example:
Interactive task → Use big core at high frequency
Background task → Use little core at low frequency
```

#### Real-Time Scheduling (SCHED_FIFO, SCHED_RR)

For hard real-time requirements:

```
SCHED_FIFO: Real-time processes run until completion or yield
SCHED_RR: Real-time processes with fixed time quantum

Priorities:
- Real-time tasks: 0-99 (fixed priority)
- Normal tasks: 100-139 (CFS scheduling with priorities)
```

#### Deadline Scheduling (SCHED_DEADLINE)

Explicit deadline support:

```c
struct sched_attr {
    u32 size;
    u32 sched_policy;
    u64 sched_runtime;      // Time allowed per period
    u64 sched_deadline;     // Relative deadline
    u64 sched_period;       // Period (runtime recharge)
};

// Scheduler: Never miss a deadline
// Preempt tasks if necessary to meet deadline
```

---

### Practical Considerations: Choosing a Scheduler

| Workload Type     | Recommended Algorithm         | Why                   |
| :---------------- | :---------------------------- | :-------------------- |
| Batch processing  | SJF (if lengths known) or CFS | Optimize turnaround   |
| Interactive shell | MLFQ or CFS                   | Responsiveness        |
| Real-time systems | SCHED_DEADLINE or SCHED_FIFO  | Guarantee timing      |
| Web server        | CFS or Lottery                | Fair resource sharing |
| Desktop/laptop    | CFS (Linux) or MLFQ (BSD/Mac) | Balance all workloads |
| Mobile/battery    | Energy-aware CFS variant      | Optimize power        |
| HPC/Cluster       | Custom (job-aware) + CFS      | Optimize throughput   |
| Virtual machines  | CFS + vCPU fair scheduling    | Isolate workloads     |

---

### Scheduling in Multi-Core Systems

#### Challenge: Load Balancing

With multiple CPUs, OS must:

1. Keep all CPUs busy (maximize throughput)
2. Minimize context switches (for cache efficiency)
3. Respect affinity (keep tasks on same CPU if possible)

**Strategy**:

```
Approach 1: Centralized queue
├─ Single global ready queue
├─ Lock contention under heavy scheduling load
└─ Good cache utilization if processes migrate cautiously

Approach 2: Per-CPU queues
├─ Each CPU has own ready queue
├─ No lock contention
├─ Potential imbalance (one CPU idle, another overloaded)
└─ Solution: Periodic load balancing (move tasks between queues)

Approach 3: Hierarchical (Linux)
├─ Per-core queues
├─ Per-socket domain
├─ NUMA domains
└─ Sophisticated balancing algorithm (considers distance, caches)
```

#### NUMA-Aware Scheduling (Linux)

```
System with 2 NUMA nodes:

Node 0        Node 1
├─ CPU 0-3    ├─ CPU 4-7
├─ RAM 0-4GB  ├─ RAM 4-8GB
└─ Cache      └─ Cache

Task T1 accessing memory in Node 0:
- Running on CPU 0: Local access (fast, ~40 cycles)
- Running on CPU 4: Remote access (slow, ~100+ cycles)

CFS + NUMA:
- Track task's preferred NUMA node
- Try to co-locate task with its memory
- Migrate task & memory together if necessary
```

---

### Summary: The Evolution of Fairness

```
Fairness concept evolution:

FIFO era (1950s):
└─ "First come, first served": Queue-based fairness

SJF era (1960s):
└─ "Shortest job first": Optimize total time, not fairness

Time-sharing era (1960s):
└─ Round-Robin: Equal time slices to all

MLFQ era (1970s-1980s):
└─ Adaptive priority: Learn and adapt

Proportional-share era (1990s):
├─ Lottery/Stride: Tickets = fair share
└─ Fair Queuing (fair net sharing) researchers

Modern era (2000s+):
├─ CFS (vruntime fairness)
├─ Group fairness (cgroups)
└─ Multi-dimensional: Fairness + Energy + Deadline
```

---

### Key Insights Across All Schedulers

1. **No perfect scheduler**: Every algorithm trades off something
2. **Context matters**: Workload determines best algorithm
3. **Learning is powerful**: MLFQ concept of predicting future from past is central to modern systems
4. **Hardware/software co-design**: Modern schedulers exploit hardware features (NUMA, multiple frequency levels)
5. **Fairness is hard**: Defining "fair" is non-trivial; different mechanisms (queues, tickets, vruntime) attempt fairness differently
6. **Predictable ≠ Optimal**: Stride scheduling is deterministic but doesn't optimize specific metrics like turnaround

---

## Master Takeaways: Core Operating Systems Concepts

### Process Management

- **Separation of fork() and exec()** enables shell features; process creation and transformation are independent
- **Limited Direct Execution** balances performance (direct hardware) with control (traps, timers)
- **System calls** are controlled entry points requiring explicit request numbers; not arbitrary jumps

### CPU Scheduling

- **Metrics matter**: Turnaround time, response time, and fairness are often at odds
- **Scheduling is adaptive**: Modern systems (MLFQ, CFS) predict future behavior from past
- **No free lunch**: Every policy trades off one goal for another
- **Hardware assists**: Timer interrupts, multiple CPUs, NUMA awareness directly impact scheduler design

### Design Principles

| Principle                | Application             | Example                                         |
| :----------------------- | :---------------------- | :---------------------------------------------- |
| **Learning from past**   | Predict job behavior    | MLFQ tracks CPU vs. I/O patterns                |
| **Layered protection**   | Privilege levels        | User mode ↔ Kernel mode transitions             |
| **Asynchronous control** | OS regains control      | Timer interrupts prevent runaway processes      |
| **Flexible primitives**  | Enable diverse policies | Tickets, weights, priorities customize fairness |
| **Measure everything**   | Drive decisions         | Turnaround, response time metrics guide design  |

---

# Final Summary and References

This study guide covers the foundational concepts of operating system process scheduling and virtualization. The progression from simple FIFO scheduling to sophisticated adaptive schedulers like MLFQ and modern Linux CFS demonstrates how operating systems evolve to balance multiple, often-competing goals:

- **Performance** (minimize turnaround time)
- **Responsiveness** (minimize response time for interactive users)
- **Fairness** (equitable resource allocation)
- **Practicality** (work without perfect information)

Modern operating systems like Linux, macOS, and Windows use refined variations of these core algorithms, adapted for multi-core hardware, NUMA architectures, and specialized workloads (real-time, mobile, cloud).

# Memory Virtualization: A Comprehensive Engineering Guide

## Table of Contents

1. [Chapter 13: The Abstraction - Address Spaces](#chapter-13-the-abstraction--address-spaces)
2. [Chapter 14: Interlude - Memory API](#chapter-14-interlude--memory-api)
3. [Chapter 15: Mechanism - Address Translation](#chapter-15-mechanism--address-translation)
4. [Chapter 16: Segmentation](#chapter-16-segmentation)
5. [Chapter 17: Free-Space Management](#chapter-17-free-space-management)
6. [Chapter 18: Paging - Introduction](#chapter-18-paging--introduction)
7. [Chapter 19: Paging - Faster Translations (TLBs)](#chapter-19-paging--faster-translations-tlbs)
8. [Chapter 20: Paging - Smaller Tables](#chapter-20-paging--smaller-tables)
9. [Chapter 21: Beyond Physical Memory - Mechanisms](#chapter-21-beyond-physical-memory--mechanisms)
10. [Chapter 22: Beyond Physical Memory - Policies](#chapter-22-beyond-physical-memory--policies)

---

## Chapter 13: The Abstraction - Address Spaces

### 13.1 Historical Context: Early Memory Management

In the earliest computer systems, memory management was straightforward but inflexible. The **physical memory layout** consisted of:

```
+-------------------+
| 0KB               |
| Operating System  |  (OS code, data, routines)
| (code, data)      |
+-------------------+
| 64KB              |
| Current Program   |  (Single running process)
| (code, data)      |
+-------------------+
| max               |
| (Empty)           |
+-------------------+
```

**Problem with Early Approach**: Only one process could run at a time. To switch processes, the OS had to save the entire memory contents to disk—an extraordinarily slow operation that made proper system sharing impossible.

### 13.2 Multiprogramming and Time Sharing

As machines became expensive resources, two key innovations emerged:

**Multiprogramming**: Multiple processes remained resident in memory simultaneously. When one process performed I/O (disk reads, waiting for input), the CPU could execute another ready process, improving CPU utilization.

**Time Sharing**: Processes shared the CPU by running for short time intervals, with the OS switching context frequently. This created the illusion that each user had exclusive access to the machine.

```
+-------------------+
| 0KB               |
| Operating System  |  (Fixed OS code)
+-------------------+
| 64KB              |
| (Free Space)      |
+-------------------+
| 128KB             |
| Process C         |  (in memory, not running)
+-------------------+
| 192KB             |
| Process B         |  (in memory, not running)
+-------------------+
| 256KB             |
| (Free Space)      |
+-------------------+
| 320KB             |
| Process A         |  (currently running)
+-------------------+
| 384KB             |
| (Free Space)      |
+-------------------+
| 512KB             |
+-------------------+
```

**Critical Issues Introduced**: With multiple processes in memory simultaneously, **isolation** and **protection** became paramount. A single errant or malicious process could read or write another process's memory, compromising the entire system.

### 13.3 The Address Space Abstraction

The **address space** is the OS's answer to these isolation and performance requirements. It represents the **running program's view of memory in the system**. The OS creates the illusion that each process has its own large, private, contiguous block of memory.

#### Key Properties of an Address Space

1. **Large**: Each process perceives a complete, independent memory space (e.g., 32-bit or 64-bit addressing)
2. **Private**: No other process can access its memory
3. **Contiguous**: From the process's perspective, memory appears as an unbroken linear block starting at address 0

#### Typical Components of an Address Space

A typical address space contains three major logical segments:

```
+-------------------+ 0KB (Low Address)
| Program Code      |  (Instructions, read-only)
+-------------------+
| (gap / free)      |  (Unused space between heap and stack)
+-------------------+
| Heap              |  (Dynamic memory - malloc(), grows upward)
|                   |
| (grows upward)    |
+-------------------+
|                   |
| (grows downward)  |
| Stack             |  (Local variables, return addresses, grows downward)
+-------------------+ 16KB (High Address)
```

**Example in a 16KB Address Space**:

```
0KB   - Code Segment (where instructions live)
1KB   - Heap Segment (malloc'd data, grows downward)
2KB   - Dynamic data structures
(free space in the middle - not allocated)
15KB  - Stack Segment (local variables, returns, grows upward from bottom)
16KB  - Maximum address
```

#### Why This Abstraction Matters

1. **Ease of Use**: Programmers don't need to worry about where variables are stored; they simply allocate and use memory
2. **Isolation**: The OS ensures each process's memory is protected from others
3. **Flexibility**: The OS can place address spaces anywhere in physical memory without the process knowing

---

## Chapter 14: Interlude - Memory API

### 14.1 Types of Memory in C Programs

C programs use two distinct types of memory, each with different allocation models and lifespans:

#### Stack Memory (Automatic Memory)

Stack memory is **managed implicitly by the compiler**. When you declare a local variable, the compiler automatically:

- Allocates space when entering a function (on the stack)
- Deallocates space when exiting the function (stack frame popped)

```c
void func() {
    int x;           // On stack - automatically allocated
    char buffer[128]; // On stack - automatically allocated
    // ... use x and buffer ...
} // Upon return: both x and buffer deallocated automatically
```

**Properties**:

- Fast allocation/deallocation (just pointer arithmetic)
- Limited lifetime (scope-dependent)
- Limited size (stack is typically small)
- No manual management required

#### Heap Memory (Dynamic Memory)

Heap memory requires **explicit programmer management**. Memory persists until the programmer explicitly frees it.

```c
void func() {
    int *ptr = malloc(sizeof(int) * 10);  // Must allocate
    // ... use ptr ...
    free(ptr);                              // Must deallocate
}
```

**Properties**:

- Programmer-controlled lifetime
- Larger available space than stack
- Slower allocation/deallocation
- More flexible but error-prone

### 14.2 The `malloc()` System Call

`malloc()` requests a contiguous block of memory from the heap and returns a pointer to it.

```c
#include <stdlib.h>

void *malloc(size_t size);  // Returns void pointer (generic address)
```

**Usage Pattern**:

```c
// Allocate space for a single integer
int *x = (int *) malloc(sizeof(int));
if (x == NULL) {
    // Allocation failed - no memory available
    perror("malloc failed");
} else {
    *x = 42;  // Use the allocated memory
}

// Allocate space for 100 integers
int *arr = (int *) malloc(100 * sizeof(int));

// Allocate space for a double
double *d = (double *) malloc(sizeof(double));
```

**Important Details**:

1. **Parameter**: Takes `size_t` (unsigned integer) for bytes requested
2. **Return**: Generic `void*` pointer (cast to correct type when assigning)
3. **Type Safety**: Always use `sizeof()` on the intended data type
4. **Strings**: Allocate `strlen(s) + 1` bytes (the +1 is for the null terminator '\0')

```c
char *str = (char *) malloc(strlen("hello") + 1);  // Correct
strcpy(str, "hello");
free(str);
```

### 14.3 The `free()` System Call

`free()` returns previously allocated memory to the heap, making it available for future allocations.

```c
int *x = malloc(sizeof(int));
// ... use x ...
free(x);
```

**Critical Point**: The size is NOT passed to `free()`. The memory allocation library **tracks the size internally** by storing metadata before the returned pointer.

### 14.4 Common Memory Management Errors

#### Error 1: Forgetting to Allocate

```c
char *src = "hello";
char *dst;              // Unallocated pointer!
strcpy(dst, src);       // SEGMENTATION FAULT - writing to invalid memory
```

**Solution**:

```c
char *src = "hello";
char *dst = (char *) malloc(strlen(src) + 1);  // Allocate first
strcpy(dst, src);                               // Now safe
free(dst);                                      // Clean up
```

#### Error 2: Buffer Overflow (Not Allocating Enough)

```c
char *src = "hello";
char *dst = (char *) malloc(strlen(src));  // BUG: forgot +1 for null terminator
strcpy(dst, src);  // Writes past allocated memory boundary
```

**Result**: Corrupts memory, may crash or create security vulnerability

#### Error 3: Uninitialized Memory Access

```c
int *x = (int *) malloc(sizeof(int));
printf("%d\n", *x);  // BUG: x contains garbage value (whatever was there before)
if (*x == 5) { }     // Random behavior - undefined!
```

**Solution**: Initialize after allocation

```c
int *x = (int *) malloc(sizeof(int));
*x = 0;  // Initialize to known value
```

#### Error 4: Memory Leaks

```c
void leaky_function() {
    int *x = (int *) malloc(sizeof(int));
    *x = 42;
    // BUG: forgot free(x)
    return;  // Memory is allocated but unreachable
}
```

In long-running programs/systems (like the OS itself), repeatedly leaving allocated memory unreachable eventually exhausts memory and forces a restart.

---

## Chapter 15: Mechanism - Address Translation

### 15.1 The Core Problem

When multiple processes reside in memory, the OS must:

1. **Place processes anywhere** in physical memory (not always at address 0)
2. **Protect each process's** memory from all other processes
3. **Do this transparently** - the process doesn't know its memory location

The process compiles assuming its address space starts at 0 and extends to some maximum. But the OS may load it anywhere in physical memory.

**Example**:

```
Virtual Address Space (from process perspective):
  0KB:    Process thinks its code is here
  15KB:   Process thinks its data is here

Physical Memory (OS perspective):
  32KB:   Process code is ACTUALLY here
  47KB:   Process data is ACTUALLY here
```

The process generates address 128 (for an instruction), but the actual instruction is at physical address 32896. How does the hardware know to translate 128 → 32896?

### 15.2 Hardware-Based Address Translation

Address translation is a technique where **the processor hardware transforms each virtual address into a physical address**.

```
Every memory access:
Virtual Address (from instruction)
        ↓
   [Hardware Translation]
        ↓
Physical Address (actual memory location)
```

The beauty: This is **transparent to the process**. The process never sees physical addresses.

### 15.3 Base and Bounds (Dynamic Relocation)

The simplest address translation technique uses two hardware registers per CPU:

1. **Base Register**: Physical address where the process is loaded
2. **Bounds Register**: Maximum size of the address space (for protection)

#### Hardware Translation Formula

```
Physical Address = Virtual Address + Base Register
```

#### Example Walkthrough

Suppose:

- Base Register = 32768 (32KB in bytes)
- Bounds Register = 16384 (16KB - max address space size)
- Process generates Virtual Address 128

**Translation**:

```
VirtualAddress:    128
Base Register:     32768
                  ------
PhysicalAddress:   32896
```

The hardware fetches from physical address 32896.

#### Bounds Checking for Protection

Before translation, the hardware checks:

```
if (VirtualAddress >= BoundsRegister) {
    RaiseException(PROTECTION_FAULT);  // Address out of bounds - kill process
} else {
    PhysicalAddress = VirtualAddress + BaseRegister;
}
```

**Example with bounds checking**:

- Bounds Register = 16384 (max 16KB)
- Process tries to access Virtual Address 20000
- 20000 >= 16384 → **PROTECTION FAULT** → OS terminates process

#### Hardware Component

These registers live in the **Memory Management Unit (MMU)** on the processor chip.

#### Example Process Execution

Instruction in assembly:

```assembly
128: movl 0x0(%ebx), %eax   ;load 0+ebx into eax
```

**Step 1**: Fetch instruction at address 128

```
VirtualAddress: 128
+ BaseRegister: 32768
= PhysicalAddress: 32896
→ Fetch instruction from physical address 32896
```

**Step 2**: Execute instruction (load from virtual address 15360 - where variable is)

```
VirtualAddress: 15360
+ BaseRegister: 32768
= PhysicalAddress: 48128
→ Fetch data from physical address 48128
```

### 15.4 Advantages and Limitations

#### Advantages

- **Simple**: Requires only addition and comparison
- **Fast**: Hardware can do translation in parallel with cache access
- **Trivial Relocation**: Process can be moved by just changing base register

#### Limitations

- **Internal Fragmentation Avoided**: Entire address space must be in physical memory
- **Inflexible**: One contiguous block per process
- **Sparse Address Spaces**: If a process only uses heap and stack (leaving gap in middle), that gap still takes physical memory
- **Pre-relocation compaction**: Software-based approach (static relocation) required pre-loading address rewriting

---

## Chapter 16: Segmentation

### 16.1 The Problem with Base-and-Bounds

Consider a 16KB address space:

```
0KB:    Code
1KB:    Gap (free)
2KB:    Heap (grows downward)
        ...
        ...
15KB:   Stack (grows downward from top)
```

With base-and-bounds, the entire 16KB must be in physical memory, even though the gap in the middle is unused. This wastes physical memory.

With 100 processes each with 16KB address spaces, we waste significant memory on unused gaps.

### 16.2 Segmentation: Multiple Base-and-Bounds Pairs

**Key Insight**: Different parts of the address space have different characteristics:

- **Code**: Read-only, doesn't grow
- **Heap**: Read-write, grows upward
- **Stack**: Read-write, grows downward

Instead of one base-and-bounds pair for the entire address space, use **multiple pairs—one per segment**.

#### Explicit Segmentation (Hardware Determines Segment by Address Bits)

The hardware uses the top bits of the virtual address to determine which segment register to use.

**Example with 2-bit segment selector** (3 segments: code, heap, stack):

```
Virtual Address: 14 bits
+--+  +--+  +--------+
Seg   (reserved)  Offset (12 bits)
 01   000000000010 0000 (binary)

Top 2 bits = 01 → Use Heap segment register
Remaining 12 bits = offset into segment
```

#### Segment-Based Address Translation Algorithm

```c
// Extract segment number from top bits
Segment = (VirtualAddress & SEG_MASK) >> SEG_SHIFT;

// Extract offset within segment
Offset = VirtualAddress & OFFSET_MASK;

// Check bounds
if (Offset >= Bounds[Segment])
    RaiseException(PROTECTION_FAULT);
else {
    // Translate using segment's base
    PhysicalAddress = Base[Segment] + Offset;
    data = AccessMemory(PhysicalAddress);
}
```

#### Concrete Example

Suppose:

```
Virtual Address: 4200 (binary: 01 0000 0110 1000)
Segment bits (top 2):    01 → Heap segment
Offset bits (bottom 12): 0000 0110 1000 = 104 decimal

Heap Base Register: 34KB (34816 bytes)
Heap Bounds:        2KB

Translation:
Offset (104) < Bounds (2048) ✓ OK
PhysicalAddress = 34816 + 104 = 34920
```

#### Hardware Segment Register State

```
| Segment | Base   | Size | Grows Up? | Protection |
|---------|--------|------|-----------|------------|
| Code    | 32KB   | 2KB  | 1 (yes)   | Read-Exec  |
| Heap    | 34KB   | 3KB  | 1 (yes)   | Read-Write |
| Stack   | 28KB   | 2KB  | 0 (no)    | Read-Write |
```

### 16.3 Handling Backward-Growing Segments (Stack)

The stack grows **downward** (toward lower addresses), which requires special handling.

**Problem**: Stack uses negative offsets from its base.

**Example**:

- Stack Base Register = 28KB = 28672 bytes
- To access virtual address 15KB from a 16KB address space:
  - Top 2 bits: 11 (stack segment)
  - Remaining offset: 3KB into the segment
  - But stack grows downward! So actual offset is: 3KB - 4KB (max segment size) = -1KB
  - Physical address: 28KB + (-1KB) = 27KB ✓

**Algorithm with negative growth**:

```c
Segment = (VirtualAddress & SEG_MASK) >> SEG_SHIFT;
Offset = VirtualAddress & OFFSET_MASK;  // 3KB

if (Segment_GrowsPositive[Segment]) {
    // Normal case: heap
    if (Offset >= Bounds[Segment])
        RaiseException(PROTECTION_FAULT);
    PhysicalAddress = Base[Segment] + Offset;
} else {
    // Backward case: stack
    // Convert to negative offset
    NegativeOffset = Offset - MaxSegmentSize;  // 3KB - 4KB = -1KB

    if (AbsoluteValue(NegativeOffset) > Bounds[Segment])
        RaiseException(PROTECTION_FAULT);

    PhysicalAddress = Base[Segment] + NegativeOffset;  // 28KB - 1KB = 27KB
}
```

### 16.4 Support for Code Sharing

**Motivation**: Shared libraries (like libc) shouldn't be replicated in memory for each process.

**Solution**: Use **protection bits** on segments

```
| Segment | Base | Size | Prot      |
|---------|------|------|-----------|
| Code    | 32KB | 2KB  | R-X       | (Read-Execute only)
| Heap    | 34KB | 3KB  | RW-       | (Read-Write)
| Stack   | 28KB | 2KB  | RW-       | (Read-Write)
```

When a segment is marked **read-execute** (R-X), it can be:

- **Read**, but not **written**
- **Executed** (fetched as instructions)
- **Shared** across multiple processes safely

The hardware checks protection bits during translation:

```c
if (CanAccess(ProtectionBits, AccessType)) {
    PhysicalAddress = Base[Segment] + Offset;
} else {
    RaiseException(PROTECTION_FAULT);
}
```

### 16.5 Coarse-Grained vs. Fine-Grained Segmentation

#### Coarse-Grained (Modern systems)

- Few segments (3-4): Code, Heap, Stack
- Simple, low hardware overhead
- Less flexible

#### Fine-Grained (Multics, B5000)

- Many segments (thousands) provided by hardware
- Requires segment table in memory
- More flexible but complex

### 16.6 The Fragmentation Problem

#### External Fragmentation

When variable-sized segments are allocated in physical memory, free space becomes scattered into small unusable pieces.

```
Before:
[Process A: 6KB] [Free: 8KB] [Process B: 4KB] [Free: 6KB] [Process C: 8KB]

New process wants 20KB segment.
24KB total free space, but not contiguous!
OS can reject allocation even though space exists.

After Compaction:
[Process A: 6KB] [Process B: 4KB] [Process C: 8KB] [Free: 20KB]√ Now satisfied!
```

**Problems with compaction**: Expensive (requires copying all data), time-consuming, can still leave fragmentation.

#### Allocation Strategies

Various free-list algorithms attempt to minimize external fragmentation:

| Algorithm     | Strategy                           | Result                                                   |
| ------------- | ---------------------------------- | -------------------------------------------------------- |
| **Best Fit**  | Return smallest chunk that fits    | Minimizes wasted space, but leaves many small chunks     |
| **Worst Fit** | Return largest available chunk     | Leaves larger free chunks, but more wasted space overall |
| **First Fit** | Return first chunk that fits       | Fast, but fragments beginning of free list               |
| **Next Fit**  | Continue search from last position | Spreads fragmentation more uniformly                     |

**None are perfect**: External fragmentation is fundamental when using variable-sized allocation.

---

## Chapter 17: Free-Space Management

### 17.1 Heap Data Structure: Free List

The most common approach to managing heap free space is the **free list**—a linked list of free memory regions.

#### Free List with Headers

Each allocated or free chunk stores metadata before the actual data:

```
Allocated Chunk:
+--------+--------+
| Header | Data   |
+--------+--------+
  (size)

Free Chunk:
+--------+        +----------+
| Header | ....   | Next Ptr |
+--------+        +----------+
(size)             (pointer to next free chunk)
```

**Header Contents**:

- **Size**: Bytes in this chunk
- **Magic Number**: Sanity check (ensures pointer is valid)
- **Next Pointer** (for free chunks): Points to next free chunk

#### Example Heap Lifecycle

**Initial State** (one 4096-byte free chunk):

```
Head → [size: 4088, next: NULL, ..................... 4088 bytes .....................]
```

**After allocating 100 bytes**:

```
Allocated:  [size: 100, magic: 1234567, ....100 bytes data....]
Free:       [size: 3980, next: NULL, ........ 3980 bytes free ...........]
```

**After allocating 3 x 100-byte chunks**:

```
Allocated:  [size: 100, ....100...]  [size: 100, ....100...]  [size: 100, ....100...]
Free:       [size: 3764, next: NULL, ......... 3764 bytes ...........]
```

**After freeing the middle chunk**:

```
Free:       [size: 100, next: → tail, ...100...]
Allocated:  [size: 100, ....100...]
Free:       [size: 3764, next: NULL, ......... 3764 ...........]
```

Without **coalescing**, you get fragmentation:

```
Free:   [100] → Free: [100] → Free: [100] → Free: [3764]
```

All memory is free but fragmented into small pieces.

**With coalescing**, merge adjacent free chunks:

```
Free:   [3964] (merged into one)
```

### 17.2 Basic Allocation Strategies

#### Best Fit

```
Strategy:
1. Scan entire free list
2. Find all chunks ≥ request size
3. Return smallest chunk among candidates

Example:
Free list: [10] → [30] → [20] → NULL
Request: 15 bytes

Candidates: [30], [20]
Best fit: [20] (smallest that fits)
Result: [10] → [30] → [5] → NULL
```

**Pros**: Minimizes wasted space  
**Cons**: Slow (O(n) search), leaves many small chunks

#### Worst Fit

```
Strategy:
1. Scan entire free list
2. Find largest chunk
3. Return requested amount; keep remainder

Example:
Free list: [10] → [30] → [20] → NULL
Request: 15

Largest: [30]
Result: [10] → [15 used] → [15] → [20] → NULL
```

**Pros**: Leaves large free chunks  
**Cons**: Still requires O(n) search; empirically performs worst (high fragmentation)

#### First Fit

```
Strategy:
1. Scan free list from beginning
2. Return first chunk ≥ request size

Example:
Free list: [10] → [30] → [20] → NULL
Request: 15

First fit: [30] (first that's big enough)
Result: [10] → [15 used] → [15] → [20] → NULL
```

**Pros**: Fast (O(1) average), simple  
**Cons**: Can fragment the beginning of the list

#### Next Fit

```
Strategy:
Like First Fit, but remember last allocation position
Resume search from there next time

Benefits: Spreads allocations more evenly throughout list
```

### 17.3 Advanced Approaches

#### Segregated Lists

Maintain **separate free lists for different size classes**:

```
Size 8:      [8] → [8] → [8] → NULL
Size 16:     [16] → [16] → NULL
Size 32:     [32] → [32] → [32] → NULL
General:     [64] → [128] → [256] → NULL
```

**Advantages**:

- No fragmentation within a size class
- Fast allocation for common sizes
- Reduced search time

**Example**: Java virtual machine uses this approach

#### Slab Allocator (Jeff Bonwick, Solaris)

Hierarchical allocation:

1. **Object caches**: Pre-initialized caches for kernel objects (locks, inodes)
2. **Slabs**: Groups of pages for each cache
3. **General allocator**: Provides slabs to specialized caches

```
Object Cache for "inodes"
  ↓
Slab 1: [initialized inode] [initialized inode] [initialized inode]
Slab 2: [initialized inode] [initialized inode] ...

When cache runs low: Request more slabs from general allocator
When objects unused: Return slabs to general allocator
```

**Benefits**:

- Re-initialized objects are already in proper state
- Reduces initialization/destruction overhead
- Scales well for kernel allocations

#### Buddy Allocation

For systems where **coalescing** is critical:

```
Memory: 2^N bytes
Request: k bytes

Algorithm:
1. Find minimum power of 2 ≥ k
2. Recursively split in half until chunk is exactly right size
3. On free: Recursively merge with buddy chunks

Example:
Total: 256 bytes

Request 32 bytes:
256 → [128|128] → [64|64|128] → [32|32|64|128]
      return 32

Request 64 bytes later:
We have buddy [32|32] → merge to [64] → return 64
```

**Advantage**: Coalescing is perfectly efficient  
**Disadvantage**: Internal fragmentation (can only allocate power-of-2 sizes)

### 17.4 Growing the Heap

When a heap allocator runs out of free space, it must request more from the OS using `sbrk()` or similar:

```c
// Heap allocator runs low on memory
new_memory = sbrk(num_pages);

// sbrk() triggers:
// 1. OS finds free physical pages
// 2. OS maps them into process address space
// 3. OS returns new heap end
// 4. Allocator treats new memory as free space
```

---

## Chapter 18: Paging - Introduction

### 18.1 Limitations of Segmentation

Despite solving fragmentation in some cases, segmentation has fundamental issues:

1. **Sparse Address Spaces**: If code and heap are small but stack is large, the entire stack must be in memory even if only top few frames are used
2. **Variable-Sized Allocation**: Still requires contiguous regions; compaction is expensive
3. **Inflexibility**: Model doesn't match actual memory usage patterns

**Solution**: Use **fixed-sized units** called **pages**

### 18.2 Paging: Basic Concept

Instead of variable-sized segments, divide both virtual and physical memory into **fixed-size chunks**:

- **Page**: Fixed-size unit of virtual address space (typically 4KB)
- **Page Frame** (or Physical Frame): Fixed-size unit of physical memory (same size as page)
- **Page Table**: Mapping from virtual page numbers to physical frame numbers

#### Address Space Division

```
Virtual Address (32-bit):
[ VPN (20 bits) ][ Offset (12 bits) ]
   1,048,576         4,096
   possible         bytes per
   pages            page

Physical Address:
[ PFN (20 bits) ][ Offset (12 bits) ]
```

#### Key Insight

With 4KB pages:

- A 32-bit address space requires 2^20 = ~1 million page table entries (PTEs)
- Each PTE ≈ 4 bytes → ~4MB per page table
- With 100 processes → 400MB just for page tables

But this overhead is acceptable because:

1. Not all pages are allocated (sparse address spaces)
2. We can use hierarchical page tables (multi-level)
3. We can use the TLB to cache frequent translations

### 18.3 Page Table Structure and Contents

A **page table** is a data structure stored in memory (typically in kernel space) that maps virtual page numbers to physical page frames.

#### Linear Page Table (Simple Array)

```c
PageTableEntry = array[VirtualPageNumber]
PFN = PageTableEntry.PageFrameNumber
```

#### Page Table Entry (PTE) Fields

Each PTE typically contains (example: x86):

| Bit(s) | Name                     | Purpose                                        |
| ------ | ------------------------ | ---------------------------------------------- |
| 0      | P (Present)              | Is page in physical memory or swapped to disk? |
| 1      | R/W (Read/Write)         | Is page writable?                              |
| 2      | U/S (User/Supervisor)    | Can user-mode processes access?                |
| 3      | PWT (Page Write Through) | Caching policy                                 |
| 4      | PCD (Page Cache Disable) | Caching policy                                 |
| 5      | A (Accessed)             | Was page recently accessed?                    |
| 6-7    | (Reserved)               | -                                              |
| 8      | D (Dirty)                | Was page recently written?                     |
| 9-11   | (Available)              | OS can use for its own purposes                |
| 12-31  | PFN (Page Frame Number)  | Physical page frame address                    |

#### Meaning of Key Bits

**Present Bit (P)**:

- P=1: Page is in physical memory
- P=0: Page is not in memory (on disk - swapped out)

**Protection Bits (R/W, U/S)**:

- Control read/write/execute permissions
- Violation raises trap to OS

**Dirty Bit (D)**:

- D=1: Page has been written since loaded
- Used for page replacement (dirty pages are expensive to evict)

**Accessed Bit (A)**:

- A=1: Page has been read or written recently
- Used for page replacement (recently accessed pages often stay)

### 18.4 Address Translation with Paging

When the CPU generates a virtual address, the hardware must:

1. Extract the Virtual Page Number (VPN) from the virtual address
2. Use VPN to index into the page table
3. Get the Physical Frame Number (PFN) from the PTE
4. Check protection bits
5. Combine PFN with offset to form physical address
6. Fetch data from physical memory

```
Virtual Address: 21 (binary: 0001 0101)
  VPN = 21 >> 4 = 1 (shifted by 4 bits; page size is 2^4 = 16 bytes)
  Offset = 21 & 0xF = 5 (lower 4 bits)

Page Table Base Register (PTBR) = 1024 (physical address)
Lookup: PageTableEntry = Memory[1024 + 1*sizeof(PTE)]

PTE contains: PFN = 7, Valid=1, Prot=readable

Physical Address = (7 << 4) | 5 = 112 + 5 = 117
Fetch data from physical address 117
```

#### Algorithm

```c
// Extract VPN
VPN = (VirtualAddress & VPN_MASK) >> SHIFT;

// Calculate PTE location in memory
PTEAddr = PTBR + (VPN * sizeof(PTE));

// Fetch PTE from memory
PTE = AccessMemory(PTEAddr);

// Check validity
if (PTE.Valid == False)
    RaiseException(SEGMENTATION_FAULT);  // Page not allocated

// Check protection
else if (CanAccess(PTE.ProtectBits) == False)
    RaiseException(PROTECTION_FAULT);   // Permission denied

// Translate
else {
    offset = VirtualAddress & OFFSET_MASK;
    PhysicalAddress = (PTE.PFN << SHIFT) | offset;
    data = AccessMemory(PhysicalAddress);
}
```

### 18.5 The Paging Slowdown Problem

**Critical Issue**: Every memory access now requires TWO memory accesses:

1. **First access**: Fetch the PTE from the page table (in memory)
2. **Second access**: Fetch the actual data

```
Example: movl 21, %eax

Without paging (1 memory access):
VirtualAddress 21 → Fetch directly from physical location

With paging (2 memory accesses):
VirtualAddress 21
  → [Memory access 1] Lookup PTE at PTBR + VPN
  → Get PFN from PTE
  → [Memory access 2] Fetch data from physical address (PFN << SHIFT | offset)
```

This would slow programs by **2x or more**! Solution: Use the **TLB**.

### 18.6 Example Memory Trace

Code:

```c
int array[1000];
for (i = 0; i < 1000; i++)
    array[i] = 0;
```

Compiled to:

```assembly
1024: movl $0x0, (%edi, %eax, 4)   // Move 0 to array[i]
1028: incl %eax                     // Increment i
1032: cmpl $0x03e8, %eax            // Compare i to 1000
1036: jne 0x1024                    // Jump if not equal
```

Assumptions:

- Virtual address space: 64KB
- Page size: 1KB
- Code at real address 1024-1028 (VPN=1, maps to PFN=4)
- Array at virtual addresses 40000-44000
  - VPN 39 → PFN 7
  - VPN 40 → PFN 8
  - VPN 41 → PFN 9
  - VPN 42 → PFN 10

---

## Chapter 19: Paging - Faster Translations (TLBs)

### 19.1 The TLB: Translation Lookaside Buffer

The **TLB** is a hardware cache that stores **recent virtual-to-physical address translations**.

Instead of accessing memory for every page table lookup, the TLB provides fast in-cache lookups for frequently-accessed pages.

```
Virtual Address
        ↓
[Check TLB Cache]
        ↓
TLB Hit (90%)→ Instant translation
TLB Miss (10%)→ Fetch from page table in memory
```

### 19.2 TLB Organization

A typical TLB:

- **Size**: 32, 64, or 128 entries
- **Associativity**: Fully associative (any entry can hold any translation)
- **Search**: Parallel search of all entries

#### TLB Entry Format

Each entry contains:

```
| VPN | PFN | Valid | Protection | ASID | Dirty | ... |
```

- **VPN**: Virtual page number (the key)
- **PFN**: Physical frame number (the result)
- **Valid Bit**: Whether this entry is valid
- **Protection Bits**: Read/write/execute permissions
- **ASID**: Address Space ID (identifies which process owns this entry)
- **Dirty Bit**: Has page been modified?

#### TLB Lookup Algorithm (Hardware Managed)

```c
VPN = (VirtualAddress & VPN_MASK) >> SHIFT;

(Hit, TLBEntry) = TLB_Lookup(VPN);

if (Hit == True) {  // TLB Hit
    if (CanAccess(TLBEntry.ProtectBits) == True) {
        offset = VirtualAddress & OFFSET_MASK;
        PhysicalAddress = (TLBEntry.PFN << SHIFT) | offset;
        data = AccessMemory(PhysicalAddress);
    } else {
        RaiseException(PROTECTION_FAULT);
    }
} else {  // TLB Miss
    // Hardware exception triggers OS to load TLB
    // (details depend on hw vs sw managed TLB)
}
```

### 19.3 Who Handles TLB Misses?

Two approaches exist, reflecting the CISC vs RISC divide:

#### Hardware-Managed TLB (x86, older systems)

The hardware itself handles the TLB miss:

```
1. TLB miss exception
2. Hardware knows where page table is (CR3 register)
3. Hardware walks page table (multi-level on x86)
4. Hardware loads translation into TLB
5. Hardware retries instruction
```

**Advantage**: Simple from OS perspective  
**Disadvantage**: Hardware must understand page table format; less flexibility

#### Software-Managed TLB (MIPS, SPARC)

The OS handles the TLB miss:

```c
// On TLB miss, hardware raises exception
TLB_Miss_Exception() {
    // OS handler
    VPN = (FaultingAddress & VPN_MASK) >> SHIFT;
    PTE = PageTable[VPN];

    if (PTE.Valid == False)
        RaiseException(SEGMENTATION_FAULT);

    // Load into TLB using privileged instruction
    TLB_Insert(VPN, PTE.PFN, PTE.ProtectBits);

    // Return from trap - hardware retries instruction
    RTF();  // Special: retries faulting instruction
}
```

**Advantage**: Flexible (OS can use any page table structure)  
**Disadvantage**: More work for OS on each miss

**Key Detail**: Return-from-trap after TLB miss must **retry the faulting instruction**, not continue to the next instruction (unlike normal traps).

### 19.4 Context Switches and the TLB Problem

**Problem**: TLB entries contain virtual-to-physical translations that are **process-specific**.

When switching processes, old TLB entries become invalid.

```
Process P1 running:
  VPN 10 → PFN 100 (in TLB)

Switch to Process P2:
  P2's VPN 10 → PFN 170 (different mapping!)

If TLB still has P1's entry:
  VPN 10 → Could translate to wrong PFN or wrong address space!
```

#### Solution 1: Flush TLB on Context Switch

On every context switch, invalidate all TLB entries:

```c
ContextSwitch(ProcessOld, ProcessNew) {
    SaveState(ProcessOld);             // Save registers, etc.
    Load_PTBR(ProcessNew.PageTable);   // Set page table base
    TLB_FlushAll();                    // Clear all TLB entries
    RestoreState(ProcessNew);
    Run(ProcessNew);
}
```

**Cost**: All TLB entries become misses, slowing down new process initially

#### Solution 2: Address Space Identifier (ASID)

Add ASID field to each TLB entry to tag which process owns it:

```
TLB Entry: [VPN | PFN | Valid | ASID | ...]

On context switch:
  Load_ASID(ProcessNew.ASID);  // Just set ASID register, don't flush TLB

TLB lookup now matches both VPN and ASID
```

**Advantage**: Keep TLB entries across context switches  
**Disadvantage**: Requires hardware ASID support; limited ASID space

### 19.5 TLB Performance Impact

With a good TLB:

```
Hit Rate: ~99%
  Hit latency: ~1 cycle (cache)
  Miss latency: ~100 cycles (page table + TLB load)

Average latency = 0.99 * 1 + 0.01 * 100 = 2 cycles
```

**Conclusion**: TLB hit rate is critical to performance

---

## Chapter 20: Paging - Smaller Tables

### 20.1 The Page Table Size Problem

With linear (single-level) page tables for a 32-bit address space:

```
Address Space: 2^32 bytes
Page Size: 4KB = 2^12 bytes
VPN bits: 32 - 12 = 20 bits
PTEs: 2^20 = ~1 million
PTE size: 4 bytes
Total: 4MB per process

With 100 processes: 400MB just for page tables!
```

For 64-bit address spaces, this becomes catastrophic (millions of gigabytes).

### 20.2 Multi-Level Page Tables

Instead of one flat page table, use a **hierarchical structure** with multiple levels.

#### Two-Level Page Table (x86)

```
Virtual Address (32-bit):
[ PDI(10) ][ PTI(10) ][ Offset(12) ]
  Page Dir    Page Tbl   in page
  Index       Index
```

**Structure**:

```
                 Page Directory
              [PDE 0 (points to PT 0)]
              [PDE 1 (points to PT 1)]
                ...
              [PDE 1023 (points to PT 1023)]
                  ↓
         Page Table 0
      [PTE 0→PFN 5]
      [PTE 1→PFN 17]
      ...
      [PTE 1023→...]
```

**Translation Process**:

```
1. PDI = (VirtualAddress >> 22) & 0x3FF  // Top 10 bits
2. PTI = (VirtualAddress >> 12) & 0x3FF  // Middle 10 bits
3. PDEAddr = PDBR + (PDI * sizeof(PDE))
4. PDE = Memory[PDEAddr]                  // Fetch page table pointer
5. PTEAddr = PDE.BaseAddr + (PTI * sizeof(PTE))
6. PTE = Memory[PTEAddr]                  // Fetch page table entry
7. PFN = PTE.PageFrameNumber
8. PhysicalAddress = (PFN << 12) | (VirtualAddress & 0xFFF)
```

#### Advantages of Multi-Level

1. **Sparse address spaces work well**: Only allocate page tables for used regions
2. **Smaller memory footprint**: Page directory small; page tables allocated on-demand

#### Example: Sparse Address Space

A 32-bit process using only code (0-2MB) and stack (4GB-2MB to 4GB):

```
Single-level: 4MB page table (entire address space)

Two-level:
  Page Directory: 4KB (always allocated)
  Page tables needed:
    - For code: ~1 page table = 4KB
    - For stack: ~1 page table = 4KB
  Total: 12KB vs 4MB → 333x savings!
```

#### Three-Level and Beyond

Many modern systems use three or more levels:

```
Virtual Address (Simplified):
[ L1(10) ][ L2(10) ][ L3(10) ][ Offset(2) ]
  Level 1   Level 2   Level 3
  (PML4)    (PDPT)    (PD)
```

Each level trades off:

- **Depth** (more memory accesses per translation)
- **Space** (larger sparse regions can be left unallocated at upper levels)

### 20.3 Hybrid Approach: Paging and Segmentation

Some systems combine both techniques:

```
Segment Base/Bounds Registers (Hardware):
- Base[0] → Page Table for Code Segment
- Base[1] → Page Table for Heap Segment
- Base[2] → Page Table for Stack Segment
```

**Translation**:

```c
// Extract segment
Segment = (VirtualAddress & SEG_MASK) >> SEG_SHIFT;

// Get VPN within segment
VPN = (VirtualAddress & VPN_MASK) >> VPN_SHIFT;

// Use segment's page table base
PageTableBase = Base[Segment];
PTEAddr = PageTableBase + (VPN * sizeof(PTE));
PTE = Memory[PTEAddr];

// Translate
PhysicalAddress = (PTE.PFN << 12) | (VirtualAddress & OFFSET_MASK);
```

**Benefit**: Unused portions of address space don't need page tables!

---

## Chapter 21: Beyond Physical Memory - Mechanisms

### 21.1 The Problem: Address Spaces Larger Than Physical Memory

Modern workloads often have virtual address spaces (32-64 bits) much larger than physical memory (e.g., 8GB).

**Solution**: Use **swap space** on disk.

The OS can page out rarely-used pages to disk, freeing physical frames for other uses. When a page is needed, page it back in.

### 21.2 Page Faults and the Present Bit

The **Present Bit (P)** in a PTE indicates if a page is in physical memory:

- P=1: Page is in physical memory
- P=0: Page is not in memory (swapped to disk); access triggers a **page fault**

#### Page Fault Handling

When a process accesses a page that's not in memory:

```c
MemoryAccess(VirtualAddress) {
    VPN = GetVPN(VirtualAddress);
    PTE = PageTable[VPN];

    if (PTE.Invalid)
        RaiseException(SEGMENTATION_FAULT);  // Page never allocated

    else if (PTE.Present == False)
        RaiseException(PAGE_FAULT);  // Page is on disk

    else if (CanAccess(PTE.ProtectBits) == False)
        RaiseException(PROTECTION_FAULT);

    else {
        PhysicalAddress = Translate(VirtualAddress);
        data = Memory[PhysicalAddress];
    }
}

PageFault_Handler(VirtualAddress) {
    // Find a free physical frame
    frame = FindFreeFrame();
    if (NoFreeFrames())
        frame = EvictPage();  // Replace an existing page

    // Load page from disk
    DiskRead(SwapSpace[VPN], frame);

    // Update PTE
    PageTable[VPN].PFN = frame;
    PageTable[VPN].Present = True;

    // Retry instruction
    RetryInstruction();
}
```

### 21.3 What If Memory Is Full?

When all physical frames are occupied, the OS must **evict** a page to disk to make room for the new page.

The choice of which page to evict is determined by the **page replacement policy** (see next chapter).

```
Physical Memory Before:
[Page A] [Page B] [Page C] [Page D] (all used, no free space)

Page Fault for new page:
1. Choose page to evict (policy-dependent, e.g., B)
2. Write Page B to disk (if dirty)
3. Free frame
4. Load new page into freed frame
5. Update page tables

Physical Memory After:
[Page A] [New Page] [Page C] [Page D]
```

### 21.4 Page Fault Control Flow

Complete flow of a memory access:

```c
// TLB Hit (90% of time):
1. Check TLB
2. Found → get PFN immediately
3. Access memory

// TLB Miss with Page in Memory (9% of time):
1. TLB miss
2. Fetch PTE from page table
3. Check PTE.Present == True
4. Load PTE into TLB
5. Retry instruction (now TLB hits)
6. Access memory

// TLB Miss with Page on Disk (1% of time):
1. TLB miss
2. Fetch PTE from page table
3. Check PTE.Present == False
4. Page fault exception
5. OS finds free frame (or evicts one)
6. OS reads page from disk (slow! ~10ms)
7. Process blocked while I/O completes
8. Other processes run
9. I/O completes, OS updates PTE
10. Update TLB with new translation
11. Retry instruction
12. TLB hit, access memory
```

### 21.5 Swap Space Management

The OS reserves disk space for virtual memory:

```
Disk Layout:
+------------------+
| File System       |  (regular files, directories)
+------------------+
| Swap Space        |  (virtual memory for pages)
| [empty slots...100m...occupied slots...50m...]
+------------------+
```

**Swap Management**:

- OS maintains free list of swap slots
- When paging out: Write PTE.VPN → Swap[PTEOffset] mapping
- When paging in: Know swap location from previous page-out
- Can be significant bottleneck if thrashing occurs

---

## Chapter 22: Beyond Physical Memory - Policies

### 22.1 Page Replacement Policies

The **page replacement policy** decides which page to evict when physical memory is full.

"Wrong" decisions cause programs to run 10,000-100,000x slower (swapping at disk speed vs. memory speed).

### 22.2 The Optimal Policy

**OPT**: Evict the page that will **not be used for the longest time**.

```
Access pattern: A B A C B D A B C ...
            Time: 0-1-2-3-4-5-6-7-8-...

Currently in memory: [A] [B] [C]
New access: D (not in memory, page fault)

Future accesses:
  A needed at: 2 (relative: 2 time units away)
  B needed at: 4 (relative: 4 time units away)
  C needed at: 8 (relative: 8 time units away)

OPT evicts C (furthest in future) and loads D
```

**Optimal hit rate**: ~95% on many workloads

**Problem**: Requires future knowledge (impossible to implement!).

Used as a benchmark to evaluate other policies.

### 22.3 FIFO (First In First Out)

**FIFO**: Evict the page that has been in memory the **longest** (oldest arrival time).

```
Load order: A, B, C, D
Memory:     [A] [B] [C]  (A oldest)

Page fault on E:
Evict A (oldest)
Result: [E] [B] [C]
```

**Advantages**: Simple, O(1) with queue  
**Disadvantages**: Often poor hit rate; may evict frequently-used old pages

**Anomaly**: Increasing cache size can actually **decrease** hit rate (Belady's Anomaly)

### 22.4 LRU (Least Recently Used)

**LRU**: Evict the page **least recently accessed**.

```
Access sequence: A B C A B D E K ...
                 0 1 2 3 4 5 6 7 ...
Memory:          [A] [B] [C]

At time 5, D causes fault.
Recent access times:
  A accessed at: 3 (4 time units ago)
  B accessed at: 4 (1 time unit ago)  ← Most recent
  C accessed at: 2 (3 time units ago)

Evict A (least recently accessed)
```

**Intuition**: Pages accessed recently are more likely to be accessed again soon (temporal locality)

**Hit rate**: ~98% on many workloads

**Implementation Problem**: Perfect LRU requires tracking every access. With modern CPUs doing billions of accesses/second, exact LRU tracking is too expensive.

```
Memory overhead for perfect LRU:
- Timestamp every access
- Update on every memory operation (huge overhead!)
```

### 22.5 Approximating LRU: The Clock Algorithm

Since perfect LRU is too expensive, use an approximation:

**Hardware support**: Add a **use bit** (reference bit) per page:

- Set to 1 whenever page is accessed
- Reset to 0 by OS periodically

**Clock Algorithm**:

1. Keep all pages arranged in a circular list with a clock hand pointer
2. On page fault:
   - Check current page (pointed by clock hand)
   - If use bit == 1: Clear it, move hand to next page
   - If use bit == 0: Evict this page, load new page
3. Clock hand rotates around the list

```
Clock of 4 pages with use bits:
     [A:1] ← hand
     [B:0]
     [C:1]
     [D:0]

Page fault:
A has use bit 1 → clear to 0, advance hand
     [A:0]
     [B:0] ← hand
     [C:1]
     [D:0]

Another fault:
B has use bit 0 → evict B, load new page, advance
     [A:0]
     [New:0]
     [C:1] ← hand
     [D:0]
```

**Result**: Approximates LRU well; hits are ~95% vs ~98% perfect LRU

**Advantage**: Practical (minimal OS overhead)

### 22.6 Considering Dirty Pages

One optimization: Prefer to evict **clean** (unmodified) pages over **dirty** (modified) pages.

**Why**: Dirty pages must be written to disk (expensive I/O), clean pages can be discarded immediately.

**Modified Clock Algorithm**:

1. First scan: Look for clean, unreferenced pages
2. If none: Look for unreferenced pages (even if dirty)
3. If none: Reset all reference bits and scan again

```
Hardware support:
  - Use bit: Page was recently accessed
  - Dirty bit: Page was recently modified

Preference to evict:
1. Unreferenced AND Clean (best)
2. Unreferenced AND Dirty (less good)
3. Referenced AND Clean (not ideal)
4. Referenced AND Dirty (last resort)
```

---

#### Key Takeaways

- **Virtual memory abstracts physical memory**: Each process sees its own large, private, contiguous address space
- **Address translation is fundamental**: Hardware transforms virtual addresses to physical addresses
- **Segmentation handles variable-sized regions**: Reduces memory waste for sparse address spaces
- **Paging uses fixed-size pages**: Simpler, enables virtual memory larger than physical memory
- **The TLB is critical**: Caches translations; without it, paging would be too slow; present in all modern CPUs
- **Multi-level page tables scale**: Allow page tables themselves to be paged; handle large address spaces efficiently
- **Swapping and page replacement are mechanisms**: OS can run programs larger than physical memory; policy choices heavily impact performance
- **LRU is near-optimal**: Performance-cost tradeoff; clock algorithm provides practical approximation

#### Important Terms

| Term                                   | Meaning & Context                                                                                                    |
| :------------------------------------- | :------------------------------------------------------------------------------------------------------------------- |
| **Virtual Address**                    | Address generated by instruction; not a real memory location; must be translated to physical address                 |
| **Physical Address**                   | Real address in physical memory; where data actually resides                                                         |
| **Address Translation**                | Hardware mechanism that converts virtual addresses to physical addresses using page tables or segmentation           |
| **Address Space**                      | The complete memory view of a process; includes code, heap, and stack; appears as 0 to max address                   |
| **Page**                               | Fixed-size unit of virtual memory (typically 4KB); smallest unit managed by virtual memory system                    |
| **Page Frame**                         | Fixed-size unit of physical memory; corresponds one-to-one with pages                                                |
| **Page Table**                         | Kernel data structure that maps virtual page numbers to physical page frames                                         |
| **Page Table Entry (PTE)**             | Single entry in page table; contains physical frame number and metadata (valid, dirty, protection bits)              |
| **Translation Lookaside Buffer (TLB)** | Hardware cache of recent virtual-to-physical translations; critical for performance                                  |
| **TLB Hit**                            | Translation found in TLB cache; very fast (~1 cycle)                                                                 |
| **TLB Miss**                           | Translation not in TLB; requires fetching PTE from memory (~100 cycles)                                              |
| **Segment**                            | Logical region of address space (code, heap, stack); different segments may have different permissions               |
| **Base Register**                      | Hardware register holding physical address where segment/process is loaded; used in translation                      |
| **Bounds Register**                    | Hardware register limiting segment size; used for protection checks                                                  |
| **Heap**                               | Dynamic memory region; allocated with malloc(); grows upward                                                         |
| **Stack**                              | Automatic memory region for function calls; grows downward; deallocated on return                                    |
| **malloc()**                           | C library function for allocating heap memory; returns pointer or NULL                                               |
| **free()**                             | C library function for deallocating heap memory; must be paired with malloc()                                        |
| **Memory Leak**                        | Allocated memory never freed; in long-running programs, exhausts memory                                              |
| **Fragmentation**                      | Wasted or unusable free space; external (between segments) or internal (within allocated block)                      |
| **Coalescing**                         | Merging adjacent free blocks into larger free block; reduces fragmentation                                           |
| **Present Bit**                        | PTE flag indicating if page is in physical memory (1) or swapped to disk (0)                                         |
| **Dirty Bit**                          | PTE flag indicating if page has been written (modified) since loaded into memory                                     |
| **Access/Use Bit**                     | PTE flag indicating if page has been recently read or written; used for page replacement                             |
| **Protection Bits**                    | PTE flags controlling read/write/execute permissions for page                                                        |
| **Page Fault**                         | Exception raised when process accesses page not in physical memory (on disk)                                         |
| **Page Replacement**                   | Selection of victim page to evict from memory when space needed; policy critical to performance                      |
| **Swap Space**                         | Reserved disk area used for pages evicted from physical memory                                                       |
| **Spatial Locality**                   | Principle that data near recently-accessed address is likely to be accessed soon                                     |
| **Temporal Locality**                  | Principle that recently-accessed data is likely to be accessed again soon                                            |
| **Context Switch**                     | OS stops running one process, runs another; must update page table base, often TLB                                   |
| **ASID (Address Space ID)**            | Hardware tag in TLB; allows cached translations to persist across context switches                                   |
| **Multi-Level Page Table**             | Hierarchical page table structure; inner tables allocated only for used regions; enables sparse address spaces       |
| **Fully Associative**                  | Every entry in cache (TLB) can hold any translation; requires parallel search                                        |
| **LRU (Least Recently Used)**          | Page replacement policy; evicts page least recently accessed; near-optimal ~98% hit rate                             |
| **Clock Algorithm**                    | Practical approximation to LRU; uses reference bits and circular scanning; ~95% hit rate                             |
| **OPT (Optimal)**                      | Theoretical page replacement policy; evicts page used furthest in future; unimplementable but represents upper bound |
| **First Fit**                          | Memory allocation strategy; return first free block that fits; fast but can fragment                                 |
| **Best Fit**                           | Memory allocation strategy; return smallest free block that fits; minimizes waste but slower                         |
| **Internal Fragmentation**             | Wasted space within allocated block (e.g., allocating 32B when only 24B needed)                                      |
| **External Fragmentation**             | Unusable free space scattered between allocated blocks; causes allocation failures despite total free space          |

---

## Summary: The Complete Memory Virtualization Picture

### The Three Layers

**1. Virtual Address Space (Process View)**

- Each process sees 0 to 2^N (contiguous, private)
- Contains code, heap, stack

**2. Hardware Translation**

- **Simple**: Base/bounds (entire address space contiguous in physical memory)
- **Intermediate**: Segmentation (multiple variable-sized regions)
- **Advanced**: Paging (fixed-size pages, flexible placement)
- **Cached**: TLB (translations cached for speed)

**3. Physical Memory (OS View)**

- Actual hardware memory
- Managed by OS, shared among all processes
- Supplemented by swap space on disk

### Why Multiple Techniques?

- **Base/Bounds**: Simplest, but inflexible; wastes memory on sparse address spaces
- **Segmentation**: Handles sparse spaces better; but external fragmentation, compaction required
- **Paging**: Solves fragmentation, enables overcommitting; but multiple memory accesses per translation
- **TLB**: Solves paging slowdown; essential for modern performance
- **Multi-level tables**: Solves page table size problem; scales to 64-bit; allows sparse allocation
- **Swap**: Allows programs larger than physical memory; but disk access is slow

### Performance Equation

```
Average Memory Access Time =
    (TLB Hit Rate × 1 cycle) +
    (TLB Miss Rate × (Page Table Lookup + TLB Load)) +
    (Page Fault Rate × Disk I/O Time)

Modern system with good TLB:
    = 0.99 × 1 + 0.01 × 100 + 0.00001 × 10,000,000
    = 0.99 + 1 + 100
    ≈ 102 cycles (page faults dominate if they occur)
```

### Practical Implementation

Real systems combine techniques:

- **Paging** as base (fixed-size units)
- **Multi-level page tables** (hierarchical structure)
- **TLB** (performance optimization)
- **Swapping** (physical memory overcommitment)
- **LRU/Clock** (page replacement policy)
- **Protection bits** (isolation and security)

This layered approach provides:

- **Illusion**: Large private address spaces
- **Performance**: Fast access via TLB
- **Flexibility**: Virtual > physical memory
- **Protection**: Processes can't interfere
- **Transparency**: Process doesn't know actual memory layout

# Concurrency and Synchronization: A Complete Study Guide

## Table of Contents

- [Chapter 26: Concurrency and Introduction](#chapter-26-concurrency-an-introduction)
- [Chapter 28: Locks](#chapter-28-locks)
- [Chapter 29: Lock-Based Concurrent Data Structures](#chapter-29-lock-based-concurrent-data-structures)
- [Chapter 30: Condition Variables](#chapter-30-condition-variables)
- [Chapter 31: Semaphores](#chapter-31-semaphores)

---

## Chapter 26: Concurrency: An Introduction

### 26.1 Introduction to Threads and Concurrency

#### What Are Threads?

A **thread** is an execution context within a process that runs independently of other threads. Think of a thread as a lightweight unit of execution that shares the same memory space (address space, heap, data segments) with other threads in the same process, but each thread has its own:

- **Program Counter (PC)** — tracks which instruction is being executed
- **Register set** — CPU registers that store temporary values during execution
- **Stack** — local variables and function call information specific to that thread's execution

Threads are created using system calls like `pthread_create()` on POSIX systems (Linux, macOS, BSD).

#### Why Threads Matter: The Concurrency Problem

**Concurrency** refers to multiple threads (or processes) executing code at overlapping time periods. On **single-CPU systems**, this is achieved through **time-slicing** — the OS rapidly switches between threads. On **multi-CPU systems**, threads truly execute in parallel.

The challenge of concurrency is that thread scheduling is non-deterministic. The OS scheduler decides which thread runs at any given moment. A programmer **cannot predict the exact interleaving of thread execution**, which leads to subtle but critical bugs.

### 26.2 Thread Creation and Execution

#### Basic POSIX Thread Example

```c
#include <stdio.h>
#include <pthread.h>

static volatile int counter = 0;

void *mythread(void *arg) {
    printf("%s: begin\n", (char *) arg);
    int i;
    for (i = 0; i < 1e7; i++) {
        counter = counter + 1;
    }
    printf("%s: done\n", (char *) arg);
    return NULL;
}

int main(int argc, char *argv[]) {
    pthread_t p1, p2;
    printf("main: begin (counter = %d)\n", counter);

    // Create two threads
    pthread_create(&p1, NULL, mythread, "A");
    pthread_create(&p2, NULL, mythread, "B");

    // Wait for both threads to finish
    pthread_join(p1, NULL);
    pthread_join(p2, NULL);

    printf("main: done with both (counter = %d)\n", counter);
    return 0;
}
```

**Code Breakdown:**

- `pthread_create(&p1, NULL, mythread, "A")` — Creates a new thread that executes `mythread("A")`. The thread ID is stored in `p1`.
- `pthread_join(p1, NULL)` — Blocks until thread `p1` completes execution.
- `static volatile int counter` — The `volatile` keyword tells the compiler not to cache the variable in a register; it must always be read from memory.

**Expected Behavior:**
Each thread increments `counter` 10 million times, so the final value should be 20,000,000. However, due to race conditions (explained below), the actual result is often incorrect.

#### Thread Execution Traces

The scheduler determines the order in which threads execute. Here are possible execution sequences:

**Scenario 1: Sequential Execution (Lucky Case)**

```
Thread 1        Thread 2         Main
starts running
prints "A"
finishes
                starts running
                prints "B"
                finishes
                                 prints "done"
```

In this case, the final counter value is correct: 20,000,000.

**Scenario 2: Interleaved Execution (Lucky case)**

```
Thread 1        Thread 2         Counter Value
Load counter    -                50
Add 1           -                -
Store counter   -                51
(context switch)
-               Load counter     51
-               Add 1            -
-               Store counter    52
```

Still correct because operations don't overlap at critical moments.

**Scenario 3: Interleaved Execution (Unlucky Case)**

```
Thread 1        Thread 2         Counter Value
Load counter    -                50
Add 1           -                -
(context switch)
-               Load counter     50 (T2 sees same value!)
-               Add 1            -
-               Store counter    51
(context switch)
Store counter   -                51 (wrong! should be 52)
```

This is the race condition.

### 26.3 Race Conditions and the Critical Section Problem

#### The Race Condition

A **race condition** (specifically, a **data race**) occurs when:

1. Multiple threads access the same shared variable
2. At least one thread modifies the variable
3. The execution order of these accesses is not synchronized

The result is **indeterminate** — the final value depends on unpredictable thread interleaving and varies from run to run.

#### Low-Level Assembly: Why This Happens

The seemingly simple line `counter = counter + 1;` compiles to multiple x86 assembly instructions:

```asm
movl    0x8049a1c, %eax      # Load counter from memory into register eax
addl    $0x1, %eax           # Add 1 to eax
movl    %eax, 0x8049a1c      # Store eax back to memory
```

This is a **Read-Modify-Write (RMW)** sequence. If a context switch happens between any two instructions, another thread can interfere.

#### Detailed Execution Trace

Assume counter = 50 initially. Instructions are at memory addresses 100, 105, and 108.

```
Point   Thread 1 Instruction      Thread 2 Instruction    Counter  T1 eax  T2 eax
(0)     -                         -                       50       -       -
(1)     movl 0x8049a1c, %eax [100]  -                      50       50      -
(2)     addl $0x1, %eax [105]       -                      50       51      -
        **CONTEXT SWITCH**
(3)     (paused)                  movl 0x8049a1c, %eax    50       51      50
(4)     (paused)                  addl $0x1, %eax         50       51      51
(5)     (paused)                  movl %eax, 0x8049a1c    51       51      51
        **CONTEXT SWITCH**
(6)     movl %eax, 0x8049a1c      (paused)                51       51      51
```

Final counter value: **51 (WRONG!)**

Expected: 52 (both increments succeeded)

#### The Critical Section

A **critical section** is a code region that accesses a shared resource and must execute **atomically** — that is, without interruption from other threads. The goal is **mutual exclusion (mutex)**: only one thread can execute the critical section at a time.

### 26.4 The Heart of the Problem: Uncontrolled Scheduling

The fundamental issue is that **the OS scheduler makes all decisions** about which thread runs when, and the programmer has no control. Even on a single CPU:

- Timer interrupts force context switches at unpredictable times
- The kernel saves the interrupted thread's state (registers, PC, etc.) into its **Thread Control Block (TCB)**
- Another thread is scheduled to run
- When the interrupted thread resumes, it continues from where it left off

This lack of synchronization between threads creates opportunities for race conditions.

---

## Chapter 28: Locks

### 28.1 Locks: The Basic Idea

#### What Is a Lock?

A **lock** (or **mutex** in POSIX terminology) is a synchronization primitive that enforces **mutual exclusion** — exactly one thread can hold the lock at a time.

#### Lock Semantics

A lock has two states:

- **FREE/UNLOCKED** — no thread holds the lock
- **HELD/LOCKED** — exactly one thread holds the lock

**Lock operations:**

- `lock()` — Try to acquire the lock. If free, immediately acquire it and enter the critical section. If held by another thread, the calling thread **blocks** (waits) until the lock becomes free.
- `unlock()` — Release the lock. If other threads are waiting, one is chosen to acquire the lock and proceed.

#### Example: Using a Lock to Protect a Critical Section

```c
lock_t mutex;  // Global lock

// Protected code:
lock(&mutex);
balance = balance + 1;  // Critical section
unlock(&mutex);
```

Now, even if context switches occur, only one thread executes `balance = balance + 1` at a time.

#### POSIX Mutex Example

```c
#include <pthread.h>

pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

// Code to execute atomically:
pthread_mutex_lock(&lock);
balance = balance + 1;
pthread_mutex_unlock(&lock);
```

### 28.2 Evaluating Locks

Good lock implementations must satisfy three properties:

| Property             | Meaning                                                                          | Importance                                 |
| :------------------- | :------------------------------------------------------------------------------- | :----------------------------------------- |
| **Mutual Exclusion** | Only one thread holds the lock at a time                                         | Correctness—prevents race conditions       |
| **Fairness**         | Threads waiting for a lock eventually acquire it; no thread starves indefinitely | Liveness—ensures all threads make progress |
| **Performance**      | Low overhead in uncontended cases; good scaling with many threads                | Efficiency—doesn't slow down the system    |

### 28.3 Building Locks: Hardware Support

#### Challenge: Disabling Interrupts (Single-CPU Only)

One naive solution for single-CPU systems is to disable hardware interrupts:

```c
void lock() {
    DisableInterrupts();
}

void unlock() {
    EnableInterrupts();
}
```

**Why this works:**

- While interrupts are disabled, no context switch can occur
- The critical section executes uninterrupted
- When interrupts are re-enabled, the OS can resume normal scheduling

**Why this is bad:**

1. **Not portable to multi-CPU systems** — other CPUs still execute threads; disabling interrupts on one CPU doesn't prevent concurrency on others
2. **Dangerous** — a buggy program that forgets to enable interrupts can hang the entire system
3. **Poor performance** — interrupts are needed for I/O, system responsiveness, and other OS functions
4. **Privilege required** — user programs typically cannot disable interrupts

#### Test-and-Set (Atomic Instruction)

Modern CPUs provide atomic instructions that execute as a single, indivisible operation. One example is **test-and-set**:

```c
int TestAndSet(int *ptr, int new) {
    int old = *ptr;        // Load old value
    *ptr = new;             // Store new value
    return old;             // Return old value (all atomically!)
}
```

**Hardware guarantee:** The entire load-modify-store sequence is atomic; no other thread can intervene.

**Using test-and-set to build a simple spin-lock:**

```c
void lock(int *flag) {
    while (TestAndSet(flag, 1) == 1)
        ;  // Spin: keep looping until we acquire the lock
}

void unlock(int *flag) {
    *flag = 0;
}
```

**How it works:**

- Each thread calls `TestAndSet(flag, 1)` repeatedly
- When `flag == 0` (lock is free), `TestAndSet` returns 0, the while condition is false, and the thread acquires the lock
- While holding the lock, the thread sets `flag = 1`
- When done, it calls `unlock()` to set `flag = 0`, allowing another thread to proceed

**Problem:** This is a **spin-lock** — threads waste CPU cycles repeatedly checking the flag. On a single CPU, if one thread is interrupted while holding the lock, other threads spin uselessly until the lock holder gets scheduled again.

#### Compare-and-Swap (CAS)

Another common atomic instruction is **compare-and-swap** (CAS):

```c
int CompareAndSwap(int *ptr, int expected, int new) {
    int actual = *ptr;
    if (actual == expected) {
        *ptr = new;
        return 1;  // Success
    }
    return 0;  // Failure
}
```

**Semantics:** Atomically compare the value at `ptr` with `expected`. If they match, update `ptr` to `new` and return success; otherwise, do nothing and return failure.

**Using CAS to build a lock:**

```c
void lock(int *flag) {
    while (CompareAndSwap(flag, 0, 1) == 0)
        ;  // Spin until we successfully change 0 → 1
}

void unlock(int *flag) {
    *flag = 0;
}
```

#### Fetch-and-Add and Ticket Locks

**Fetch-and-add** is an atomic instruction that increments a value and returns the old value:

```c
int FetchAndAdd(int *ptr) {
    int old = *ptr;
    *ptr = old + 1;
    return old;  // Return old value (all atomically)
}
```

**Ticket locks** use fetch-and-add to ensure fairness:

```c
typedef struct {
    int ticket;  // Next ticket to be issued
    int turn;    // Ticket of thread that should proceed
} lock_t;

void lock_init(lock_t *lock) {
    lock->ticket = 0;
    lock->turn = 0;
}

void lock(lock_t *lock) {
    int myturn = FetchAndAdd(&lock->ticket);  // Get a ticket number
    while (lock->turn != myturn)
        ;  // Spin until it's my turn
}

void unlock(lock_t *lock) {
    lock->turn = lock->turn + 1;  // Advance to next ticket holder
}
```

**Key advantage:** Guarantees **FIFO fairness**. Each thread gets a ticket in order; threads are served in ticket order.

**Problem:** Spinning is still inefficient, especially on single-CPU systems.

### 28.4 OS Support for Efficient Locks

#### The Spinning Problem

Spinning wastes CPU cycles. Consider two threads on a single CPU:

```
Time    Thread 1           Thread 2         CPU State
(0)     Acquires lock      -                T1 running
(1)     Executes critical  -                T1 running
        section
(2)     (Running)          Tries lock()      Timer interrupt
(3)     (in kernel)        spin-loops        T2 running (wasting cycles!)
(4)     (in kernel)        spin-loops        T2 running (wasting cycles!)
(5)     Returns from       spin-loops        Timer interrupt
        critical section
(6)     unlock()           spin-loops        T1 running
(7)     (returns)          Acquires lock!    Context switch
(8)     -                  Executes section  T2 running (finally productive)
```

With N threads contending for one lock, a single context switch can waste N-1 entire time slices.

#### Yield-Based Locking

A simple improvement: if the lock is held, yield the CPU instead of spinning:

```c
void lock() {
    while (TestAndSet(&flag, 1) == 1)
        yield();  // Give up the CPU voluntarily
}

void unlock() {
    flag = 0;
}
```

`yield()` is a system call that moves the current thread from RUNNING → READY state, allowing the OS scheduler to run another thread.

**Problem:** With 100 threads contending for a lock, a thread might yield 99 times before getting its turn. Each yield/reschedule cycle has overhead (context switch). Plus, there's no guarantee against **starvation** — a thread might get stuck in a partial-yield loop while others repeatedly acquire and release the lock.

#### Queue-Based Locking with park() and unpark()

The solution: the OS maintains a per-lock queue of waiting threads. Threads don't spin or yield; they **sleep** (are blocked).

**Solaris provides:**

- `park()` — Put the current thread to sleep, waiting to be woken up
- `unpark(threadID)` — Wake a specific sleeping thread

**Implementation:**

```c
typedef struct {
    int flag;           // 0 = free, 1 = held
    int guard;          // Guards flag and queue (spin-lock)
    queue_t *q;         // Queue of waiting threads
} lock_t;

void lock(lock_t *m) {
    // First acquire a guard lock (spin briefly)
    while (TestAndSet(&m->guard, 1) == 1)
        ;  // Spin on guard only (very short critical section)

    if (m->flag == 0) {
        // Lock is free
        m->flag = 1;
        m->guard = 0;
    } else {
        // Lock is held; add ourselves to the queue and sleep
        queue_add(m->q, gettid());  // gettid() returns current thread ID
        m->guard = 0;
        park();  // Sleep; OS will wake us when lock is free
    }
}

void unlock(lock_t *m) {
    // Acquire guard lock
    while (TestAndSet(&m->guard, 1) == 1)
        ;

    if (queue_empty(m->q)) {
        // No one is waiting
        m->flag = 0;
    } else {
        // Wake the next thread in the queue
        unpark(queue_remove(m->q));
        // Don't release flag; the woken thread holds it
    }
    m->guard = 0;
}
```

**Why this is better:**

1. **No spinning on the main lock** — threads sleep instead of spinning
2. **Deterministic fairness** — the queue ensures FIFO order
3. **No starvation** — each waiting thread will eventually be woken

**Subtle issue — the wakeup/waiting race:**
Between adding to the queue and calling `park()`, another thread might call `unpark()`. If that happens, `park()` sleeps forever (waked before it sleeps).

Solution: Use `setpark()` to indicate "I'm about to park"; if `unpark()` is called between `setpark()` and `park()`, the subsequent `park()` returns immediately.

```c
queue_add(m->q, gettid());
setpark();     // Signal: "I'm about to sleep"
m->guard = 0;
park();        // If unpark() already happened, return immediately
```

#### Linux Futexes

Linux provides a more efficient **futex** (fast userspace mutex) mechanism:

- Each futex is associated with a memory location and an in-kernel counter
- Threads can wait/wake using atomic operations on that memory location
- The kernel avoids context switches when possible by checking atomicity

A futex provides further optimization: fast path (uncontended case) avoids system calls entirely.

### 28.5 Summary: Building Efficient Locks

| Approach                      | Mechanism                   | Pros                             | Cons                                          |
| :---------------------------- | :-------------------------- | :------------------------------- | :-------------------------------------------- |
| **Disable Interrupts**        | `DisableInterrupts()`       | Simple, works on single CPU      | Not portable; dangerous; poor performance     |
| **Spin-Lock (Test-and-Set)**  | Hardware atomic instruction | Simple, fast on multi-CPU        | Wastes CPU on single-CPU; starvation possible |
| **Yield-Based**               | `yield()` system call       | Reduces wasted CPU               | Context-switch overhead; starvation possible  |
| **Sleep-Based (park/unpark)** | Queue + `park()`/`unpark()` | Fair; efficient; no spinning     | Requires OS support; more complex             |
| **Futex**                     | Memory-based wait queue     | Fast uncontended path; efficient | Requires OS support; platform-specific        |

---

## Chapter 29: Lock-Based Concurrent Data Structures

### 29.1 Concurrent Counter

The simplest concurrent data structure is a **counter** protected by a lock:

```c
typedef struct {
    int value;
    pthread_mutex_t lock;
} counter_t;

void counter_init(counter_t *c) {
    c->value = 0;
    pthread_mutex_init(&c->lock, NULL);
}

void counter_increment(counter_t *c) {
    pthread_mutex_lock(&c->lock);
    c->value++;
    pthread_mutex_unlock(&c->lock);
}

int counter_get(counter_t *c) {
    pthread_mutex_lock(&c->lock);
    int val = c->value;
    pthread_mutex_unlock(&c->lock);
    return val;
}
```

**Lesson:** Lock the entire operation (read is also a critical section; without the lock, you might read while another thread is incrementing).

### 29.2 Concurrent Linked List

#### Naive Approach (Coarse-Grained Locking)

Lock the entire list for any operation:

```c
typedef struct __node_t {
    int key;
    struct __node_t *next;
} node_t;

typedef struct __list_t {
    node_t *head;
    pthread_mutex_t lock;
} list_t;

void List_Init(list_t *L) {
    L->head = NULL;
    pthread_mutex_init(&L->lock, NULL);
}

int List_Insert(list_t *L, int key) {
    pthread_mutex_lock(&L->lock);

    node_t *new = malloc(sizeof(node_t));
    if (new == NULL) {
        pthread_mutex_unlock(&L->lock);
        return -1;  // Failure
    }
    new->key = key;
    new->next = L->head;
    L->head = new;

    pthread_mutex_unlock(&L->lock);
    return 0;  // Success
}

int List_Lookup(list_t *L, int key) {
    pthread_mutex_lock(&L->lock);

    node_t *curr = L->head;
    while (curr) {
        if (curr->key == key) {
            pthread_mutex_unlock(&L->lock);
            return 0;  // Found
        }
        curr = curr->next;
    }

    pthread_mutex_unlock(&L->lock);
    return -1;  // Not found
}
```

**Issue:** `malloc()` is slow; we hold the lock during memory allocation, preventing other threads from accessing the list.

#### Optimized Approach: Lock Only the Critical Section

```c
void List_Insert(list_t *L, int key) {
    // Allocate OUTSIDE the lock
    node_t *new = malloc(sizeof(node_t));
    if (new == NULL) {
        return;
    }
    new->key = key;

    // Lock only the critical section (updating pointers)
    pthread_mutex_lock(&L->lock);
    new->next = L->head;
    L->head = new;
    pthread_mutex_unlock(&L->lock);
}

int List_Lookup(list_t *L, int key) {
    int rv = -1;
    pthread_mutex_lock(&L->lock);

    node_t *curr = L->head;
    while (curr) {
        if (curr->key == key) {
            rv = 0;
            break;  // Common exit path
        }
        curr = curr->next;
    }

    pthread_mutex_unlock(&L->lock);
    return rv;  // Single return point
}
```

**Improvement:** Free the lock as soon as possible; allocate memory outside the critical section.

#### Fine-Grained Locking: Hand-Over-Hand Locking

Instead of one lock for the entire list, add a lock per node:

```c
typedef struct __node_t {
    int key;
    struct __node_t *next;
    pthread_mutex_t lock;
} node_t;
```

When traversing:

1. Acquire the next node's lock
2. Release the current node's lock (hand-over-hand)
3. Continue to the next node

**Theoretically:** Enables multiple threads to traverse different parts of the list simultaneously.

**In practice:** Overhead of acquiring/releasing a lock per node often exceeds the benefit of allowing concurrent traversals, especially if the list is not massive.

### 29.3 Concurrent Queue

A queue has two ends: head (dequeue) and tail (enqueue). A simple optimization is to use **two locks**:

```c
typedef struct __node_t {
    int value;
    struct __node_t *next;
} node_t;

typedef struct __queue_t {
    node_t *head;
    node_t *tail;
    pthread_mutex_t head_lock, tail_lock;
} queue_t;

void Queue_Init(queue_t *q) {
    node_t *tmp = malloc(sizeof(node_t));  // Dummy node
    tmp->next = NULL;
    q->head = q->tail = tmp;
    pthread_mutex_init(&q->head_lock, NULL);
    pthread_mutex_init(&q->tail_lock, NULL);
}

void Queue_Enqueue(queue_t *q, int value) {
    node_t *tmp = malloc(sizeof(node_t));
    tmp->value = value;
    tmp->next = NULL;

    pthread_mutex_lock(&q->tail_lock);
    q->tail->next = tmp;
    q->tail = tmp;
    pthread_mutex_unlock(&q->tail_lock);
}

int Queue_Dequeue(queue_t *q, int *value) {
    pthread_mutex_lock(&q->head_lock);

    node_t *tmp = q->head;
    node_t *new_head = tmp->next;

    if (new_head == NULL) {
        pthread_mutex_unlock(&q->head_lock);
        return -1;  // Queue is empty
    }

    *value = new_head->value;
    q->head = new_head;
    pthread_mutex_unlock(&q->head_lock);
    free(tmp);  // Free the old dummy/node

    return 0;  // Success
}
```

**Key insight:** In the common case, `enqueue()` only needs `tail_lock`, and `dequeue()` only needs `head_lock`. They can operate concurrently.

**Dummy node:** A sentinel node at initialization facilitates separation of head and tail operations.

### 29.4 Concurrent Hash Table

A hash table is an array of buckets (lists). Each bucket can have its own lock:

```c
#define BUCKETS 101

typedef struct __hash_t {
    list_t lists[BUCKETS];  // Array of concurrent lists
} hash_t;

void Hash_Init(hash_t *H) {
    for (int i = 0; i < BUCKETS; i++)
        List_Init(&H->lists[i]);
}

int Hash_Insert(hash_t *H, int key) {
    return List_Insert(&H->lists[key % BUCKETS], key);
}

int Hash_Lookup(hash_t *H, int key) {
    return List_Lookup(&H->lists[key % BUCKETS], key);
}
```

**Scalability:** Multiple threads can work on different buckets simultaneously. With 101 buckets and 4 CPUs, up to 101 concurrent operations are possible (versus 1 with a single lock).

**Performance:** Hash tables with per-bucket locks scale dramatically better than linked lists with a single lock, especially as the number of operations and threads increases.

---

## Chapter 30: Condition Variables

### 30.1 The Need for Condition Variables

Sometimes threads must wait for a **condition** — a specific state of program execution — before proceeding.

#### Example: Producer-Consumer Problem

A producer generates data; a consumer processes it. They communicate via a bounded buffer.

**Naive approach:\*\***

```c
int buffer[MAX];
int fill = 0;  // Number of items in buffer
int empty = MAX;  // Number of empty slots

void put(int value) {
    buffer[fill] = value;
    fill = (fill + 1) % MAX;
}

int get() {
    int tmp = buffer[fill];
    fill = (fill + 1) % MAX;
    return tmp;
}

void *producer(void *arg) {
    for (int i = 0; i < LOOPS; i++) {
        // **Problem:** No way to wait for empty slots
        put(i);
        empty--;
    }
}

void *consumer(void *arg) {
    for (int i = 0; i < LOOPS; i++) {
        // **Problem:** No way to wait until buffer has data
        int tmp = get();
        fill--;
    }
}
```

**Issue:**

- The producer needs to wait if the buffer is full
- The consumer needs to wait if the buffer is empty
- Using a spin-loop (`while (buffer_full) ;`) wastes CPU

**Solution:** **Condition variables** allow threads to sleep until a condition becomes true.

### 30.2 Condition Variables: Core Semantics

A **condition variable (CV)** is a queue of threads. It has two main operations:

#### `wait(lock)`

```c
void wait(cond_t *cv, mutex_t *lock) {
    // Atomically:
    // 1. Release the lock
    // 2. Add this thread to the CV's wait queue
    // 3. Sleep (block)
    // When woken up:
    // 4. Reacquire the lock
    // 5. Return
}
```

**Semantics:** The calling thread releases the lock and sleeps on the CV. The lock is released atomically with sleep to prevent a race condition where the thread sleeps before another thread signals.

#### `signal(lock)`

```c
void signal(cond_t *cv) {
    // Wake ONE thread sleeping on this CV
    // (If multiple threads are sleeping, the choice is arbitrary)
}
```

**Note:** Signaling does NOT automatically make the woken thread run. It moves the thread from blocked state to ready; the descheduler decides when it runs.

#### `broadcast()`

```c
void broadcast(cond_t *cv) {
    // Wake ALL threads sleeping on this CV
}
```

### 30.3 Example: Producer-Consumer with Condition Variables

```c
int buffer[MAX];
int fill = 0;
int use = 0;
int count = 0;

cond_t empty, full;  // Two condition variables
mutex_t lock;

void put(int value) {
    buffer[fill] = value;
    fill = (fill + 1) % MAX;
    count++;
}

int get() {
    int tmp = buffer[use];
    use = (use + 1) % MAX;
    count--;
    return tmp;
}

void *producer(void *arg) {
    for (int i = 0; i < LOOPS; i++) {
        mutex_lock(&lock);

        // Wait while buffer is full
        while (count == MAX)
            cond_wait(&full, &lock);

        put(i);

        // Signal that buffer is no longer empty
        cond_signal(&empty);

        mutex_unlock(&lock);
    }
}

void *consumer(void *arg) {
    for (int i = 0; i < LOOPS; i++) {
        mutex_lock(&lock);

        // Wait while buffer is empty
        while (count == 0)
            cond_wait(&empty, &lock);

        int tmp = get();

        // Signal that buffer is no longer full
        cond_signal(&full);

        mutex_unlock(&lock);

        printf("%d\n", tmp);
    }
}
```

#### Execution Timeline

```
Producer Thread      Consumer Thread      Buffer State (count)
                                          0 (empty)
lock(&lock)
while (count==MAX)   -> FALSE, continue
put(i=0)
count = 1
signal(&empty)
unlock(&lock)                             1

                     lock(&lock)
                     while (count==0)     -> FALSE, continue
                     get() returns 0
                     count = 0
                     signal(&full)
                     unlock(&lock)        0

lock(&lock)
while (count==MAX)
put(i=1)
count = 1
signal(&empty)
unlock(&lock)                             1
                                          ...
```

#### Why Use `while` Instead of `if`?

**Mesa semantics** vs. **Hoare semantics**:

- **Mesa:** Signaling a thread only moves it to ready; it doesn't guarantee the condition still holds when it runs.
- **Hoare:** Signaling a thread runs immediately before returning from signal.

Most systems (Linux, Unix, etc.) use Mesa semantics. Therefore, **always recheck the condition with a `while` loop** after waking up, in case:

1. Another thread modified the condition between wake and run
2. The condition was never true (spurious wakeup)

**Problem**: Single condition variable with multiple sleepers on different conditions

If both producer and consumer threads sleep on the _same_ condition variable, a signaling thread doesn't know which type to wake:

```c
void *producer(void *arg) {
    for (int i = 0; i < LOOPS; i++) {
        mutex_lock(&lock);
        while (count == MAX)
            cond_wait(&cond, &lock);  // Both sleep here!
        put(i);
        cond_signal(&cond);  // Could wake another producer instead of consumer!
        mutex_unlock(&lock);
    }
}
```

**Solution:** Use two condition variables:

- `empty` — consumer sleeps here; producer signals
- `full` — producer sleeps here; consumer signals

Alternatively, use `broadcast()` to wake all threads and let them check conditions:

```c
cond_broadcast(&cond);  // Wake everyone; they'll recheck while loops
```

### 30.4 Important Rules for Condition Variables

1. **Always hold the lock when calling `wait()`, `signal()`, or `broadcast()`**
2. **Recheck conditions in a `while` loop** (not `if`)
3. **Use multiple CVs for different conditions** when possible
4. Understand **Mesa semantics** — signaling is a hint, not a guarantee

---

## Chapter 31: Semaphores

### 31.1 Introduction to Semaphores

A **semaphore** is a low-level synchronization primitive that combines aspects of locks and condition variables. It has:

- An integer value (the semaphore counter)
- Two atomic operations: `wait()` (also called `P()` or `down()`) and `post()` (also called `V()` or `up()`)

#### Semaphore Operations

**`wait()` (P)**

```
if (value > 0) {
    value--;
} else {
    sleep and add to waiting queue;
}
```

**`post()` (V)**

```
value++;
if (any threads waiting) {
    wake one waiting thread;
}
```

**Key:** Both operations are atomic (indivisible).

### 31.2 Using Semaphores as Mutexes (Locks)

A semaphore initialized to 1 acts as a binary lock:

```c
sem_t mutex;
sem_init(&mutex, 0, 1);  // Initialize to value 1

// Critical section:
sem_wait(&mutex);     // Decrement to 0; if already 0, wait
critical_section();
sem_post(&mutex);     // Increment to 1; wake waiter if any
```

**How it works:**

- First thread calls `sem_wait()` → value becomes 0 → enters critical section
- Second thread calls `sem_wait()` → value is 0 → sleeps
- First thread calls `sem_post()` → value becomes 1 → wakes second thread
- Second thread resumes from `sem_wait()` → value becomes 0 → enters critical section

### 31.3 Using Semaphores for Ordering

A semaphore initialized to 0 can enforce ordering between threads:

```c
sem_t s;
sem_init(&s, 0, 0);  // Initialize to 0

void *parent(void *arg) {
    printf("parent: begin\n");
    // ... do parent stuff ...
    sem_post(&s);  // Signal child
    printf("parent: end\n");
}

void *child(void *arg) {
    sem_wait(&s);  // Wait for parent
    printf("child: begin\n");
    printf("child: end\n");
}
```

**Execution:**

```
parent: start
parent: begin
parent: do stuff
parent: signal child (s becomes 1)
parent: end

child: block on wait (s becomes 0)
child: resume
child: begin
child: end
```

The child cannot proceed until the parent chooses to signal.

### 31.4 Producer-Consumer Problem with Semaphores

#### Initial Attempt (Incorrect)

```c
sem_t empty, full;

void *producer(void *arg) {
    for (int i = 0; i < LOOPS; i++) {
        sem_wait(&empty);  // Wait for empty slot
        put(i);
        sem_post(&full);   // Signal that slot is full
    }
}

void *consumer(void *arg) {
    for (int i = 0; i < LOOPS; i++) {
        sem_wait(&full);   // Wait for full slot
        int tmp = get();
        sem_post(&empty);  // Signal that slot is empty
        printf("%d\n", tmp);
    }
}

int main() {
    sem_init(&empty, 0, MAX);  // MAX empty slots initially
    sem_init(&full, 0, 0);     // 0 full slots initially
    // ... create and join threads ...
}
```

**Problem:** This works for a single producer and single consumer, but with multiple producers or consumers, a **data race** can occur. Two producers might both see an empty slot and both write to it simultaneously.

#### Corrected Solution (with Mutex)

```c
sem_t empty, full, mutex;

void *producer(void *arg) {
    for (int i = 0; i < LOOPS; i++) {
        sem_wait(&empty);       // Wait for empty slot
        sem_wait(&mutex);       // Acquire lock
        put(i);
        sem_post(&mutex);       // Release lock
        sem_post(&full);        // Signal full slot
    }
}

void *consumer(void *arg) {
    for (int i = 0; i < LOOPS; i++) {
        sem_wait(&full);        // Wait for full slot
        sem_wait(&mutex);       // Acquire lock
        int tmp = get();
        sem_post(&mutex);       // Release lock
        sem_post(&empty);       // Signal empty slot
        printf("%d\n", tmp);
    }
}

int main() {
    sem_init(&empty, 0, MAX);  // MAX empty slots
    sem_init(&full, 0, 0);     // 0 full slots
    sem_init(&mutex, 0, 1);    // Mutual exclusion
    // ... create and join threads ...
}
```

**Order of operations:**

1. Wait for availability (empty or full)
2. Acquire the mutex
3. Perform the operation
4. Release the mutex
5. Signal availability (full or empty)

This ensures that `put()` and `get()` are protected.

### 31.5 Initializing Semaphores Correctly

**Principle:** Initialize the semaphore to the number of resources you can immediately give away.

| Use Case                              | Initial Value | Reasoning                                                        |
| :------------------------------------ | :------------ | :--------------------------------------------------------------- |
| **Binary lock (mutex)**               | 1             | You can give the lock away immediately (it's free)               |
| **Ordering (child waits for parent)** | 0             | You have no resources initially; parent creates one by signaling |
| **Bounded buffer (N slots)**          | N             | You have N empty slots available initially                       |
| **Bounded buffer (initial data)**     | 0             | You have no full slots initially                                 |

### 31.6 Semaphores vs. Condition Variables vs. Locks

| Primitive              | Purpose                      | Typical Use                         | Complexity                               |
| :--------------------- | :--------------------------- | :---------------------------------- | :--------------------------------------- |
| **Mutex**              | Mutual exclusion             | Protecting shared data              | Simple; just lock/unlock                 |
| **Condition Variable** | Wait for condition           | Signaling between threads           | Moderate; requires lock; use while loops |
| **Semaphore**          | Counting resource / ordering | Multiple resources; thread ordering | Flexible; can replace both locks and CVs |

**Practical note:** Condition variables + mutexes are often clearer and preferred in modern code. Semaphores are lower-level and more error-prone if used incorrectly (e.g., wrong initialization, wrong signal/wait order).

---

## Key Takeaways

### Chapter 26: Concurrency and Introduction

- Multiple threads in the same process share memory but have independent execution contexts (registers, stack, PC)
- **Race conditions** occur when multiple threads access shared data without synchronization; results are **indeterminate** and vary across runs
- A **critical section** is code that accesses shared resources and must execute atomically
- Even simple operations like `counter++` compile to multiple instructions; interrupts between instructions cause race conditions
- **Mutual exclusion** is needed to ensure only one thread executes the critical section at a time

### Chapter 28: Locks

- A **lock** (or **mutex**) enforces mutual exclusion; only one thread holds it at a time
- **Hardware atomic instructions** (test-and-set, compare-and-swap) are necessary to build locks
- **Spin-locks** waste CPU cycles; on a single CPU with preemption, they are highly inefficient
- **OS support** is essential for efficient locks: `park()`/`unpark()` put threads to sleep instead of spinning
- **Ticket locks** ensure FIFO fairness; modern systems use more sophisticated mechanisms (futexes, condition variables)
- **Lock contention** and overhead must be minimized; lock only the critical section, not surrounding code

### Chapter 29: Lock-Based Concurrent Data Structures

- **Coarse-grained locking** (one lock for the entire data structure) is simple but limits concurrency
- **Fine-grained locking** (locks per element or per section) improves concurrency but adds complexity and overhead
- **Hash tables** with per-bucket locks scale far better than single-locked lists
- **Allocate memory outside the lock** to minimize time spent holding it
- **Never forget to unlock** — use proper control flow (single exit points) to avoid bugs

### Chapter 30: Condition Variables

- Condition variables allow threads to **sleep and wake based on conditions**, avoiding wasteful spin-loops
- `wait()` atomically releases the lock and sleeps; upon waking, it reacquires the lock
- `signal()` wakes one waiting thread; `broadcast()` wakes all
- **Always use `while` loops** to recheck conditions after waking (Mesa semantics)
- Multiple condition variables enable clearer code (one per distinct condition/result)
- Must hold a lock when calling wait/signal/broadcast

### Chapter 31: Semaphores

- A **semaphore** is an atomic counter with `wait()` (decrement and sleep if ≤ 0) and `post()` (increment and wake)
- Semaphore value = 1 acts as a mutex; value = 0 enforces ordering
- For the producer-consumer problem with multiple threads, a **mutex semaphore is essential** to protect shared buffer operations
- Initialize semaphores to the number of immediately-available resources
- Semaphores are powerful but lower-level; condition variables + mutexes are often clearer

---

## Important Terms

| Term                             | Meaning & Context                                                                                                                                                                      |
| :------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Thread**                       | A unit of execution within a process that shares memory with other threads but has its own registers, stack, and program counter. Created with `pthread_create()`.                     |
| **Race Condition / Data Race**   | Occurs when multiple threads access shared data without synchronization, and at least one writes. Results are indeterminate and vary across runs.                                      |
| **Critical Section**             | A region of code that accesses shared resources. Must execute atomically (uninterrupted) to prevent race conditions.                                                                   |
| **Mutual Exclusion (mutex)**     | A property guaranteeing only one thread executes the critical section at a time. Achieved with locks.                                                                                  |
| **Lock / Mutex**                 | A synchronization primitive with two states (free/held). `lock()` acquires it; `unlock()` releases it. Enforces mutual exclusion.                                                      |
| **Atomic Instruction**           | A CPU instruction that executes indivisibly—it cannot be interrupted mid-execution. Examples: test-and-set, compare-and-swap.                                                          |
| **Spin-Lock**                    | A lock implemented by repeatedly checking a flag (spinning). Wastes CPU on single-CPU systems if the lock is held by a preempted thread.                                               |
| **Starvation**                   | A thread is indefinitely prevented from making progress. Occurs with poor lock scheduling. Ticket locks prevent this.                                                                  |
| **Context Switch**               | OS kernel saves a thread's state (registers, PC, etc.) and loads another thread's state, allowing multithreading on a single CPU.                                                      |
| **Thread Control Block (TCB)**   | Kernel data structure storing a thread's saved state: registers, PC, stack pointer, etc. Used during context switches.                                                                 |
| **park() / unpark()**            | OS calls: `park()` puts a thread to sleep; `unpark(threadID)` wakes it. Used to build efficient locks without spinning.                                                                |
| **Condition Variable (CV)**      | A queue of threads. `wait(lock)` releases the lock and sleeps; `signal()` wakes one. Enables efficient waiting on conditions.                                                          |
| **Mesa Semantics**               | Signaling a thread only makes it ready; no guarantee the condition still holds when it runs. Requires rechecking condition with `while`, not `if`.                                     |
| **Semaphore**                    | An atomic counter with `wait()` (decrement; sleep if negative) and `post()` (increment; wake if sleeping). Value = 1 acts as a lock; value = 0 enforces ordering.                      |
| **Bounded Buffer**               | A fixed-size queue shared between producers and consumers. Needs synchronization: consumers wait if empty; producers wait if full.                                                     |
| **Producer-Consumer Problem**    | Synchronization problem: producer generates data (put); consumer processes it (get). Requires coordination to avoid buffer underflow/overflow.                                         |
| **Fine-Grained Locking**         | Using multiple locks for different parts of a data structure (e.g., per-node locks in a linked list) to improve concurrency.                                                           |
| **Coarse-Grained Locking**       | Using a single lock for an entire data structure. Simple but reduces concurrency.                                                                                                      |
| **Hand-Over-Hand Locking**       | Fine-grained technique: acquire next node's lock, release current node's lock during list traversal. Theoretically better but often slower in practice due to overhead.                |
| **Futex (Fast Userspace Mutex)** | Linux mechanism combining memory-based wait queue with kernel support for efficient lock implementation. Avoids system calls on the fast path.                                         |
| **Priority Inversion**           | A higher-priority thread blocked waiting for a lower-priority thread. Occurs with spin-locks if the lock holder is preempted. Solved with priority inheritance or avoiding spin-locks. |
| **Volatile**                     | Compiler keyword indicating a variable must always be read from memory, not cached in registers. Used for thread-shared variables.                                                     |
| **Atomic Operation**             | A sequence of instructions that appear to execute as a single indivisible unit, unaffected by interleaving with other threads.                                                         |

---

## ASCII Diagrams: Visual Reference

### Thread Interleaving and Context Switches

```
Timeline: How a single CPU context-switches between threads

CPU Time →

Thread 1:  [Running] [Running] [Paused] [Paused] [Running] [Running] [Paused]

Thread 2:  [Paused] [Paused] [Running] [Running] [Paused] [Paused] [Running]

Kernel:    Context-switch events indicated above

Note: Only one thread has CPU at any moment. OS decides scheduling.
```

### Race Condition: Counter Example

```
Initial: counter = 50

Thread 1 Execution:          Thread 2 Execution:
─────────────────          ─────────────────
1. Load counter → eax = 50
2. Add 1 → eax = 51
                            3. Load counter → eax = 50 (sees old value!)
                            4. Add 1 → eax = 51
                            5. Store eax → counter = 51
6. Store eax → counter = 51 (overwrites T2's write!)

Final: counter = 51 ❌ (should be 52)
```

### Thread States and Transitions

```
                  ┌───────────────┐
                  │   CREATED     │
                  └───────┬───────┘
                          │ pthread_create()
                          ▼
        ┌─────────────────────────────────────┐
        │                                      │
        │   ┌────────────────────┐            │
        │   │     RUNNING        │            │
        │   │ (on CPU)           │            │
        │   └────────────────────┘            │
        │          ▲         ▼                │
        │          │ context switch
        │          │ (timer interrupt)
        │          │         ▼
        │   ┌────────────────────┐            │
        │   │     READY          │            │
        │   │ (in queue, waiting │            │
        │   │  for CPU)          │            │
        │   └────────────────────┘            │
        │                                      │
        └─────────────────────────────────────┘
                  ▲              ▲
                  │              │
                  └──────┬───────┘
                    (other events)
                         ▼
                ┌──────────────────┐
                │   BLOCKED        │
                │ (wait for lock,   │
                │  I/O, etc.)      │
                └──────────────────┘
                    (condition met / signal)
                         ▲
                         │
        When condition is met or thread is signaled,
        BLOCKED → READY
```

### Lock Acquisition and Release

```
Two threads competing for one lock:

TIME    Thread A              Lock State       Thread B
─────   ────────────────      ──────────      ────────────────
  0     Calls lock()          LOCKED          Calls lock()
        Returns (owns lock)   (by A)          → Blocked
        Enters critical section

  1     (in critical section) LOCKED
        (in critical section) (by A)
        (in critical section)

  2     Calls unlock()        FREE            (still blocked)
        Returns
                              FREE            Wakes up

  3     Exits critical section              Calls lock() (retry)
                              LOCKED          Returns (owns lock)
                              (by B)

  4     (outside critical     LOCKED          Enters critical section
        section)              (by B)

  5     ...                   LOCKED          (in critical section)
                              (by B)

  6     ...                   LOCKED          Calls unlock()
                              (by B)

  7     ...                   FREE            Returns
```

### Condition Variable Synchronization

```
Producer-Consumer Scenario:

Buffer is empty initially.
Consumer wants to consume; Producer wants to produce.

INITIAL STATE:
Producer: can produce (count < MAX)
Consumer: must wait (count == 0)
Condition Variable 'empty': Consumer sleeping here

TIMELINE:

Time  Producer              Buffer        Consumer
────  ────────────────      ──────────    ────────────────
  0   -                     [empty]       Calls wait(&empty)
                                          → Sleeps, releases lock

  1   Acquires lock         [empty]       (sleeping)
      Adds item
      count = 1
      Signals 'empty'
      (wakes Consumer)
      Releases lock
                            [1 item]      Wakes up

  2   -                     [1 item]      Acquires lock
                                          (returning from wait)

  3   -                     [1 item]      Removes item
                                          count = 0
                                          Releases lock
                                          Returns from wait

  ...continues...
```

### Semaphore State: Binary Lock (N=1)

```
Semaphore initialized to 1 (like a binary lock):

Initial: sem_value = 1

Thread A:              Semaphore        Thread B
────────────────      ────────────     ────────────────
sem_wait(&s)          1 → 0            Calls sem_wait(&s)
value decremented     (held by A)      → Blocks
Returns (owns)                         (waits in queue)
[Critical Section]
sem_post(&s)          0 → 1            (still blocked)
value incremented     (free)           (in queue)
Returns
                                       Wakes up
                                       Decrements: 1 → 0
                                       Returns (owns)
                                       [Critical Section]
                                       sem_post(&s)
                                       Increments: 0 → 1
```

### Semaphore State: Ordering (N=0)

```
Semaphore initialized to 0 (for ordering):

Initial: sem_value = 0

Parent Thread                Semaphore        Child Thread
────────────────────        ────────────     ────────────────
Start                       value = 0
(do work)
sem_post(&s)                0 → 1            Calls sem_wait(&s)
value incremented           (available)      0: blocks in queue
Returns
(continue work)
                                             Wakes up
                                             Decrements: 1 → 0
                                             Returns
                                             (do child work)
```

### Lock Contention and Scaling

```
Performance vs. Threads (Concurrent Hash Table vs. Linked List):

Throughput
    ▲
    │                    Hash Table
    │                  (per-bucket locks) ╱╱╱
    │                                  ╱╱
    │                              ╱╱
    │                          ╱╱
    │                      ╱╱
    │      Linked List ─────────────── (single lock)
    │    (plateaus early)
    │
    └─────────────────────────────────────────────────► Threads
      1      2      4      8     16     32

With per-bucket locking, hash table scales nearly linearly (up to number
of buckets). Single-locked linked list doesn't scale beyond 1-2 threads
due to contention.
```

---

## Summary and Study Recommendations

This study guide covers the fundamental synchronization primitives and patterns for concurrent programming:

1. **Understand the Problem First** — Know what data races are, why they occur, and how to recognize them.

2. **Start with Locks** — Mutexes are the foundational synchronization primitive. Understand how they force mutual exclusion.

3. **Recognize Inefficiency** — Spinning, yielding, and context switches all have costs. Understand why OS support (park/unpark, futexes) is essential.

4. **Advance to Condition Variables** — For signaling and waiting on conditions more efficiently than polling.

5. **Understand Semaphores** — A more general primitive that can replace both locks and condition variables, though often less intuitive.

6. **Apply to Data Structures** — Learn to protect shared data (counters, linked lists, queues, hash tables) with appropriate locking strategies.

7. **Think About Scale** — Fine-grained locking and per-bucket/per-node protection scales better than global locks, though with added complexity.

8. **Test and Measure** — Concurrency bugs manifest under load. Use tools like helgrind (valgrind) to detect races, and always measure performance — intuition about concurrency is often wrong.

# I/O, Storage & Persistence: Comprehensive OS Study Guide

**Operating Systems: Three Easy Pieces (OSTEP) - Chapters 35-45**

---

## Table of Contents

1. [Chapter 36: I/O Devices](#chapter-36-io-devices)
2. [Chapter 37: Hard Disk Drives](#chapter-37-hard-disk-drives)
3. [Chapter 38: RAID](#chapter-38-raid)
4. [Chapter 39: Files and Directories](#chapter-39-files-and-directories)
5. [Chapter 40: File System Implementation](#chapter-40-file-system-implementation)
6. [Chapter 41: Locality and the Fast File System](#chapter-41-locality-and-ffs)
7. [Chapter 42: Crash Consistency & Journaling](#chapter-42-crash-consistency-journaling)
8. [Chapter 43: Log-Structured File Systems](#chapter-43-log-structured-file-systems)
9. [Chapter 44: Flash-Based SSDs](#chapter-44-flash-based-ssds)

---

## Chapter 36: I/O Devices

### Overview

I/O devices are the bridge between the CPU/memory system and the outside world. The core challenge is managing the vastly different speeds between fast processors and slow peripherals (disks, networks, etc.).

### System Architecture

#### I/O Bus Hierarchy

The typical system uses a hierarchical bus structure:

```
                    ┌─ Memory Bus (fast) ─┐
                    │  (10s of GB/s)      │
                ┌────────────────────────────┐
                │        CPU/Cache           │
                └────────────────────────────┘
                    │ (North Bridge)
            ┌───────────────────────┐
            │  General I/O Bus      │
            │  (PCI: MB/s speeds)   │
            └───────────────────────┘
                    │ (South Bridge)
        ┌───────────────────────────────┐
        │  Peripheral Buses (USB,       │
        │  SCSI, SATA, Ethernet)        │
        │  (KB/s to MB/s)               │
        └───────────────────────────────┘
```

**Key Components**:

- **Memory bus**: High-speed connection between CPU and main memory
- **General I/O bus** (e.g., PCI): Connects graphics, SCSI controllers, etc.
- **Peripheral buses** (USB, SATA, SCSI): Connect actual devices

### I/O Communication Methods

#### 1. Polling

The CPU repeatedly checks the device status register until data is ready.

```c
// Simple polling example
while ((status_reg & DEVICE_READY) == 0) {
    // Busy-wait: consume CPU cycles
}
data = data_reg;
```

**Advantages**: Simple to implement  
**Disadvantages**: Wastes CPU cycles; poor for slow devices

#### 2. Interrupt-Driven I/O

The device signals completion via interrupt, allowing CPU to do other work.

```c
// Device raises interrupt when data ready
// CPU context switches to interrupt handler
handle_interrupt() {
    data = read(data_reg);
    process_data(data);
    // Return to previous task
}
```

**Advantages**: CPU can multitask while waiting  
**Disadvantages**: More complex; interrupt overhead for fast devices

#### 3. Direct Memory Access (DMA)

A specialized hardware controller moves data between device and memory without CPU involvement.

```c
// CPU initiates DMA transfer
dma_controller.set_source(disk_address);
dma_controller.set_dest(memory_address);
dma_controller.set_count(num_bytes);
dma_controller.start();

// CPU continues other work
// Device raises interrupt when transfer complete
```

**How it works**:

1. CPU programs DMA controller with source, destination, and size
2. DMA controller handles all data movement
3. Device raises interrupt when complete
4. CPU reads interrupt status

**Advantages**: Frees CPU from data movement; excellent for large transfers  
**Disadvantages**: More complex hardware; memory bus contention

#### Performance Trade-offs

- **High-speed devices** (fast networks): Polling may be better to avoid interrupt overhead
- **Slow devices** (disk): Interrupts essential to avoid wasting CPU
- **Bulk transfers** (disk to memory): DMA is ideal

### Example: xv6 IDE Driver (Disk Interface)

Modern disks use IDE/SATA interface with memory-mapped I/O ports:

```c
// Port addresses for IDE disk interface
#define DISK_BASE    0x1F0    // Base I/O port
#define DISK_STATUS  (DISK_BASE + 7)   // Status register
#define DISK_DATA    (DISK_BASE + 0)   // Data port
#define IDE_BSY      0x80      // Busy bit
#define IDE_DRDY     0x40      // Drive ready bit
#define IDE_DRQ      0x08      // Data request ready

// 1. Polling for disk readiness
void ide_wait_ready(void) {
    while ((inb(DISK_STATUS) & IDE_BSY) ||
           !(inb(DISK_STATUS) & IDE_DRDY))
        ;  // Busy-wait until drive ready and not busy
}

// 2. Issue read/write command to disk
void ide_start_request(uint32_t sector, uint8_t *data,
                       int nsects, int write) {
    outb(DISK_BASE + 2, nsects);           // Sector count
    outb(DISK_BASE + 3, sector & 0xFF);    // LBA low byte
    outb(DISK_BASE + 4, (sector >> 8) & 0xFF);
    outb(DISK_BASE + 5, (sector >> 16) & 0xFF);
    outb(DISK_BASE + 6, 0xE0 | ((sector >> 24) & 0x0F));

    if (write) {
        outb(DISK_BASE + 7, 0x30);  // Write command
        outsl(DISK_DATA, data, BSIZE/4);  // Write data
    } else {
        outb(DISK_BASE + 7, 0x20);  // Read command
    }
}

// 3. Asynchronous I/O: Queue request and sleep process
void ide_rw(bufs *b) {
    // Queue the request
    acquire(&idelock);
    b->status = B_PENDING;
    q_push(&request_queue, b);
    release(&idelock);

    // If this is the first request, start the disk
    if (request_queue_was_empty) {
        start_next_request();
    }

    // Sleep until I/O completes
    sleep(b, &idelock);
}

// 4. Interrupt handler for disk completion
void ide_intr(void) {
    bufs *b;

    // Get completed request
    if (!(b = request_queue_pop()))
        return;  // Stray interrupt

    // Mark complete based on operation type
    if (b->flags & B_WRITE) {
        // Write already completed; just mark done
        b->status = B_DONE;
    } else {
        // Read: fetch data from disk
        insl(DISK_DATA, b->data, BSIZE/4);
        b->status = B_DONE;
    }

    // Wake waiting process and start next request
    wakeup(b);
    if (!request_queue_empty())
        start_next_request();
}
```

**Key Insights**:

- **Asynchronous I/O**: Process sleeps while disk works; other processes can run
- **Interrupt-driven**: Device signals completion; no polling
- **Queuing**: Multiple requests can be queued for the disk to process
- **Buffering**: BSIZE (typically 512 bytes) chunks of data

---

## Chapter 37: Hard Disk Drives

### Disk Mechanics

#### Physical Structure

```
        Spindle (motor)
           ↓    ↓
      ┌─────────────┐
      │   Platter   │  ← Data surface
      │  (spinning) │
      └─────────────┘
         ↑     ↑
    Disk Arm  Head (read/write)
    (moves)   (actuator)
```

**Components**:

- **Platters**: Rotating magnetic storage media (spinning at 7200-15000 RPM)
- **Tracks**: Concentric circles on platter
- **Sectors**: 512-byte (or 4KB) addressable units within tracks
- **Cylinders**: Tracks at same radius across different surfaces
- **Disk Arm & Head**: Actuator that positions head to read/write

#### Disk Address Mapping

Data is addressed via **Logical Block Addressing (LBA)**:

```
Logical Block Address → Physical Location

LBA 0, 1, 2, 3, ... → Converted via disk firmware to:
  Cylinder, Head (surface), Sector

Example layout:
C  H  S (Cylinder, Head, Sector)
0  0  0 → LBA 0
0  0  1 → LBA 1
...
0  1  0 → LBA (sectors per track)
1  0  0 → LBA (sectors per track × heads)
```

### Disk Performance Model

#### I/O Time Formula

The time to complete a disk I/O consists of three components:

$$T_{I/O} = T_{seek} + T_{rotation} + T_{transfer}$$

**Components**:

1. **Seek time** ($T_{seek}$): Time for actuator to move head to desired cylinder
   - Range: 0 ms (already at cylinder) to ~10 ms (full stroke)
   - Typical: 2-5 ms
   - Modern: 1-2 ms

2. **Rotational delay** ($T_{rotation}$): Time to wait for desired sector to reach head
   - Average: Half a rotation (disk spins but we arrive at random sector)
   - Formula: $T_{rotation} = \frac{1}{2 \times RPM} \times 60 \text{ seconds}$
   - At 7200 RPM: ~4.2 ms
   - At 15000 RPM: ~2 ms

3. **Transfer time** ($T_{transfer}$): Time to read/write data
   - Formula: $T_{transfer} = \frac{\text{bytes to transfer}}{\text{bandwidth}}$
   - For 512-byte sector at 100 MB/s: ~0.005 ms (negligible)
   - For 4 MB at 100 MB/s: ~0.04 ms

#### Example Calculation

```
Reading one 512-byte sector from typical 7200 RPM disk:

T_seek = 3 ms (average)
T_rotation = 1/2 * (60,000 ms / 7200) = 4.17 ms
T_transfer = 512 bytes / 100 MB/s ≈ 0.005 ms

T_I/O ≈ 3 + 4.17 + 0.005 ≈ 7.2 ms total
```

**Key Insight**: Mechanical delays dominate; data transfer is nearly free!

### Disk Scheduling Algorithms

The order in which the disk services requests significantly affects performance. Seek and rotation times are minimized by smart scheduling.

#### 1. Shortest-Seek-Time-First (SSTF)

Service the request closest to the current head position.

```
Requests: 5, 12, 25, 3, 10
Queue: [5, 12, 25, 3, 10]

Head at 1:
  Choose 3 (closest) → Head moves to 3
  Remaining: [5, 12, 25, 10]

From 3:
  Choose 5 (closest) → Head moves to 5
  Remaining: [12, 25, 10]

From 5:
  Choose 10 (closest) → Head moves to 10
  Remaining: [12, 25]

From 10:
  Choose 12 (closest) → Head moves to 12
  Remaining: [25]

From 12:
  Choose 25 → Head moves to 25

Schedule: 1 → 3 → 5 → 10 → 12 → 25
Total distance: 2 + 2 + 5 + 2 + 13 = 24
```

**Advantages**:

- Minimizes average seek time
- Simple to implement

**Disadvantages**:

- **Starvation**: Requests at disk edges may never be served if new requests keep arriving in the middle

```
Example starvation:
Queue: [0, 99]   (request at track 0 and track 99)
Head at 50
SSTF chooses 99 (closest)
New request arrives at 51
SSTF chooses 51 instead of 0
0 keeps starving!
```

#### 2. SCAN ("Elevator Algorithm")

Sweep across the disk like an elevator: move from one end to the other, servicing requests along the way.

```
Requests: 5, 12, 25, 3, 10
Head at 1, moving right

Head 1 → visits 3 (on path)
       → visits 5 (on path)
       → visits 10 (on path)
       → visits 12 (on path)
       → visits 25 (on path)
       → reaches end, reverses

Schedule: 1 → 3 → 5 → 10 → 12 → 25
Total distance: 2 + 2 + 5 + 2 + 13 = 24 (same as SSTF in this case)
```

**Advantages**:

- Prevents starvation: all requests eventually served
- Smooth, predictable performance
- Good fairness

**Disadvantages**:

- Requests at far end may wait while nearer requests served multiple times
- Can cause "biased" service (near/far ends treated unequally)

#### 3. C-SCAN (Circular SCAN)

Like SCAN but only sweep in one direction; when reaching end, return to start without servicing requests.

```
Requests: 5, 12, 25, 3, 10
Head at 1, moving right

Head 1 → visits 3, 5, 10, 12, 25
       → reaches end, jumps back to 0 without stopping
       → now ready for requests coming to track 0

Schedule: 1 → 3 → 5 → 10 → 12 → 25 → (jump to 0)
Total seek distance: 24
```

**Advantages**:

- More uniform wait times than SCAN
- Prevents "clustering" of requests at far end

#### 4. SPTF (Shortest-Positioning-Time-First)

Choose request with smallest $T_{seek} + T_{rotation}$ (not just seek time).

**Advantages**:

- Accounts for rotational delay, not just seek
- More accurate model of disk performance

**Disadvantages**:

- Requires knowing disk rotational position (modern disks hide this)
- Rarely implemented on real disks

### Comparison Table

| Algorithm | Fairness  | Performance | Starvation          |
| --------- | --------- | ----------- | ------------------- |
| SSTF      | Poor      | Good        | Yes (starves edges) |
| SCAN      | Good      | Good        | No                  |
| C-SCAN    | Excellent | Good        | No                  |
| SPTF      | Moderate  | Best        | Yes                 |

### Write Buffering & Caching

Modern disks include:

- **Write cache**: Buffers writes before committing to disk
- **Read-ahead cache**: Prefetches expected sectors
- **Reordering**: Reorders requests to minimize seek distance

**Durability Trade-off**: Write caching speeds up writes but risks data loss on power failure.

---

## Chapter 38: RAID

### Motivation

A single disk has limited:

- **Performance**: ~100-200 MB/s
- **Reliability**: Mean-time-to-failure of ~5 years
- **Capacity**: Upgrade path is discrete (buy bigger disk)

**RAID** (Redundant Array of Inexpensive/Independent Disks) addresses these through parallelism and redundancy.

### RAID Levels

#### RAID-0: Striping (No Redundancy)

Data split across multiple disks in round-robin fashion.

```
File: [D0, D1, D2, D3, D4, D5]

With 3 disks:
Disk 0: [D0, D3]
Disk 1: [D1, D4]
Disk 2: [D2, D5]

Layout:
┌─────────────┬─────────────┬─────────────┐
│     Disk 0  │     Disk 1  │     Disk 2  │
├─────────────┼─────────────┼─────────────┤
│ [D0][D3]    │ [D1][D4]    │ [D2][D5]    │
└─────────────┴─────────────┴─────────────┘
```

**Advantages**:

- **Full capacity use**: N disks × disk capacity = total capacity
- **Peak performance**: All N disks work in parallel
  - Sequential read: N × single-disk bandwidth
  - Random read/write: N × single-disk throughput

**Disadvantages**:

- **No redundancy**: Single disk failure = total data loss
- **Reliability worse than single disk**: MTBF = (single MTBF) / N

**When to use**: Temporary storage, caches, non-critical data

#### RAID-1: Mirroring

Every disk is duplicated (mirrored). Reads can come from either copy; writes go to both.

```
File: [D0, D1, D2, D3]

With 2 disks:
Disk 0: [D0, D1, D2, D3]
Disk 1: [D0, D1, D2, D3]  (exact copy)

Layout:
┌─────────────────────────────┐
│  Disk 0 (Primary)           │
│  [D0][D1][D2][D3]           │
└─────────────────────────────┘
           ║
        (mirror)
           ║
┌─────────────────────────────┐
│  Disk 1 (Backup)            │
│  [D0][D1][D2][D3]           │
└─────────────────────────────┘
```

**Capacity Analysis**:

- Total usable capacity: Disk capacity (50% of raw)
- Reason: One full copy is redundancy

**Reliability**:

- Can tolerate any single disk failure
- With independent failure: MTBF from one disk becomes years\*N (much better)

**Performance**:

- **Sequential reads**: 2× throughput (read from either disk)
- **Random reads**: 2× throughput
- **Sequential writes**: 1× throughput (both disks written)
- **Random writes**: 1× throughput

**When to use**: Critical data, database systems, high-reliability requirement

#### RAID-4: Block-Level Striping with Dedicated Parity

Data striped across N disks; parity on dedicated disk (often called "parity disk").

```
Data blocks:    [D0, D1, D2, D3, D4, D5, D6, D7]

With 3 data disks + 1 parity disk:
Disk 0: [D0, D4]
Disk 1: [D1, D5]
Disk 2: [D2, D6]
Disk 3: [P0, P1]  (parity disk)

Where:
P0 = D0 ⊕ D1 ⊕ D2   (XOR of data blocks)
P1 = D4 ⊕ D5 ⊕ D6
```

**XOR Parity Mechanism**:

XOR has useful property: A ⊕ B ⊕ C ⊕ P = 0 (if P = A ⊕ B ⊕ C)

This means: **Any missing block can be recovered by XORing all others**

```
Example with 3 data blocks:
P = D0 ⊕ D1 ⊕ D2

If D1 lost:
D1 = D0 ⊕ D2 ⊕ P

If D2 lost:
D2 = D0 ⊕ D1 ⊕ P
```

**Capacity Analysis**:

- With N data disks + 1 parity disk:
  - Raw capacity: (N+1) × disk capacity
  - Usable: N × disk capacity (one disk is parity)
  - Overhead: 1/(N+1) of storage

**Reliability**:

- Can tolerate ONE disk failure (any disk)
- Two failures = data loss

**Performance Problem - Small Writes**:

Writing one small block requires updating parity:

```
Old state:          New state (if changing D0 to D0'):
P = D0 ⊕ D1 ⊕ D2   P' = D0' ⊕ D1 ⊕ D2

We can compute:
P' = (D0 ⊕ D0') ⊕ P  (XOR the changed bit and old parity)
```

A small write requires:

1. Read old data block (D0)
2. Read old parity block (P)
3. Compute: P_new = (D0_old ⊕ D0_new) ⊕ P_old
4. Write new data block (D0_new)
5. Write new parity block (P_new)

**Total: 4 I/Os for 1 data I/O!** (2 reads + 2 writes = "small write problem")

**RAID-4 Performance Summary**:

| Operation        | I/Os | Notes                       |
| ---------------- | ---- | --------------------------- |
| Sequential read  | ~1   | All disks accessed          |
| Sequential write | ~2   | Data + parity written       |
| Random read      | ~1   | One disk                    |
| Random write     | ~4   | 2 reads + 2 writes (parity) |

#### RAID-5: Distributed Parity

Like RAID-4, but parity is distributed across all disks (no dedicated parity disk).

```
Data blocks:    [D0, D1, D2, D3, D4, D5, D6, D7]

With 3 disks (stripe width = 2, parity rotates):
Stripe 0:
  Disk 0: D0
  Disk 1: D1
  Disk 2: P0 = D0 ⊕ D1

Stripe 1:
  Disk 0: D2
  Disk 1: P1 = D2 ⊕ D3   (parity on different disk!)
  Disk 2: D3

Stripe 2:
  Disk 0: P2 = D4 ⊕ D5
  Disk 1: D4
  Disk 2: D5
```

**Advantages**:

- **Load balancing**: No single parity bottleneck
- **Better write performance**: Parity writes distributed
- **More capacity efficient**: Only one disk used for parity info

**Disadvantages**:

- More complex (which disk has parity for which stripe?)
- Still can't handle 2 disk failures

**Performance** (similar to RAID-4):

- Sequential read/write: ~(N-1) disks (one is parity)
- Random read: ~1 disk
- Random write: still ~4 I/Os (2 reads, 2 writes)

### RAID Comparison Table

| Metric           | RAID-0 | RAID-1  | RAID-4/5 |
| ---------------- | ------ | ------- | -------- |
| Capacity         | N×C    | N×C / 2 | (N-1)×C  |
| Min. disks       | 2      | 2       | 3        |
| Fault tolerance  | 0      | 1       | 1        |
| Seq. read BW     | N×B    | N×B     | (N-1)×B  |
| Seq. write BW    | N×B    | N×B/2   | (N-1)×B  |
| Rand. read IOPS  | N×I    | N×I     | I        |
| Rand. write IOPS | N×I    | N×I/2   | I/4      |
| Cost/reliability | Poor   | High    | Good     |

### When to Use Each

- **RAID-0**: Temporary data, caching, performance-critical non-critical
- **RAID-1**: Databases, high availability, reliability critical
- **RAID-5**: General-purpose, balance of performance and reliability
- **RAID-6**: Like RAID-5 but survives 2 failures (uses 2 parity blocks)

---

## Chapter 39: Files and Directories

### File Abstraction

A file is a linear array of bytes, typically stored on persistent media (disk). The OS provides system calls to:

- Create, open, close, and delete files
- Read from and write to files
- Query file metadata

### File System Calls

#### Opening a File

```c
int fd = open(pathname, flags, mode);
```

**Parameters**:

- `pathname`: Path to file (absolute or relative)
- `flags`: Control behavior
  - `O_RDONLY`: Read-only
  - `O_WRONLY`: Write-only
  - `O_RDWR`: Read and write
  - `O_CREAT`: Create if doesn't exist (with mode)
  - `O_TRUNC`: Truncate to zero bytes (for writing)
  - `O_APPEND`: Append writes to end
- `mode`: Permissions for new files (e.g., `S_IRUSR|S_IWUSR` for owner read+write)

**Returns**: File descriptor (small int) or -1 on error

**Internally**:

1. Kernel allocates entry in per-process file descriptor table
2. Creates or looks up file's inode
3. Performs permission checks
4. Returns file descriptor

**Example**:

```c
// Create file for writing
int fd = open("data.txt", O_CREAT | O_WRONLY | O_TRUNC,
              S_IRUSR | S_IWUSR);  // rw-------

// Open existing file for reading
int fd = open("data.txt", O_RDONLY);

// Open file for appending
int fd = open("log.txt", O_WRONLY | O_APPEND);
```

#### Reading from a File

```c
ssize_t n = read(int fd, void *buffer, size_t count);
```

**Operation**:

1. Read up to `count` bytes from file descriptor `fd`
2. Store in `buffer`
3. Advance file offset by number of bytes read
4. Return number of bytes actually read (may be less than `count`)

**Semantics**:

- File offset is maintained per file descriptor
- Each read advances offset automatically
- Reading past EOF returns 0

**Example**:

```c
char buf[1024];
ssize_t nread = read(fd, buf, sizeof(buf));
if (nread > 0) {
    printf("Read %ld bytes\n", nread);
}
```

#### Writing to a File

```c
ssize_t n = write(int fd, const void *buffer, size_t count);
```

**Operation**:

1. Write up to `count` bytes from `buffer` to file `fd`
2. Advance file offset
3. Return number of bytes written

**Important**: Short writes possible if disk full or buffer full.

#### Seeking Within a File

```c
off_t offset = lseek(int fd, off_t offset, int whence);
```

**Parameters**:

- `offset`: Byte offset to move to
- `whence`: Interpretation of offset
  - `SEEK_SET`: Absolute position (0 = start of file)
  - `SEEK_CUR`: Relative to current position
  - `SEEK_END`: Relative to end of file

**Examples**:

```c
// Go to start
lseek(fd, 0, SEEK_SET);

// Go to 100 bytes from end
lseek(fd, -100, SEEK_END);

// Go forward 4096 bytes
lseek(fd, 4096, SEEK_CUR);

// Create sparse file (seek past EOF then write)
lseek(fd, 1000000, SEEK_SET);
write(fd, "data", 4);  // Gap of 1MB filled with zeros
```

#### Forcing Durability

```c
int fsync(int fd);
```

**Operation**:

- Forces file data and metadata to persistent storage
- Returns only after write reaches disk
- Necessary for critical data (databases, transactions)

**Important for safety**:

```c
// Dangerous: data may be lost on crash
write(fd, important_data, size);
// Return immediately, data still in cache

// Safe: data is durable
write(fd, important_data, size);
fsync(fd);  // Wait for disk write
// Now safe if system crashes
```

### Directory Operations

A directory is a special file containing (name, inode_number) mappings.

#### Listing Directory Contents

```c
#include <dirent.h>

DIR *dir = opendir(path);
struct dirent *entry;

while ((entry = readdir(dir)) != NULL) {
    printf("%s\n", entry->d_name);
}

closedir(dir);
```

**Dirent Structure** (simplified):

```c
struct dirent {
    ino_t d_ino;           // Inode number
    char d_name[256];      // Filename
    unsigned char d_type;  // File type (file, dir, link, etc.)
};
```

#### Creating Directories

```c
int mkdir(const char *path, mode_t mode);
```

**Creates** new directory with specified mode.

#### Directory Listing Example

```c
void list_dir(const char *path) {
    DIR *dir = opendir(path);
    if (!dir) {
        perror("opendir");
        return;
    }

    struct dirent *entry;
    while ((entry = readdir(dir)) != NULL) {
        // Skip . and ..
        if (strcmp(entry->d_name, ".") == 0 ||
            strcmp(entry->d_name, "..") == 0)
            continue;

        printf("%s", entry->d_name);
        if (entry->d_type == DT_DIR)
            printf("/");  // Mark directories
        printf("\n");
    }

    closedir(dir);
}
```

### File Metadata: Stat

```c
#include <sys/stat.h>

int stat(const char *path, struct stat *buf);
int fstat(int fd, struct stat *buf);
```

**Stat Structure** (simplified version):

```c
struct stat {
    ino_t st_ino;           // Inode number
    mode_t st_mode;         // Permissions and type
    nlink_t st_nlink;       // Number of hard links
    uid_t st_uid;           // User ID (owner)
    gid_t st_gid;           // Group ID
    off_t st_size;          // Size in bytes
    blkcnt_t st_blocks;     // Blocks allocated
    time_t st_atime;        // Last access time
    time_t st_mtime;        // Last modification time
    time_t st_ctime;        // Last change time (metadata)
};
```

**File Type** (from `st_mode`):

```c
#define S_ISREG(m)  ((m & S_IFMT) == S_IFREG)  // Regular file
#define S_ISDIR(m)  ((m & S_IFMT) == S_IFDIR)  // Directory
#define S_ISLNK(m)  ((m & S_IFMT) == S_IFLNK)  // Symbolic link
```

**Permissions** (from `st_mode`):

```
High bits: File type (regular, directory, link, etc.)
Low 9 bits: Permissions (rwxrwxrwx for owner-group-other)

  9 8 7   6 5 4   3 2 1
┌─────┬─────────┬─────────┐
│ FT  │ Owner   │ Group   │ Other
└─────┬─────────┬─────────┘
      r w x     r w x     r w x
      4 2 1     4 2 1     4 2 1
```

**Example**:

```c
// Check if file exists
struct stat sb;
if (stat(path, &sb) == -1) {
    perror("File not found");
}

// Check if regular file
if (S_ISREG(sb.st_mode)) {
    printf("Regular file, %ld bytes\n", sb.st_size);
}

// Check permissions
if (sb.st_mode & S_IRUSR) {
    printf("Owner can read\n");
}
```

### Hard Links and Symbolic Links

#### Hard Links

Hard link creates another name for the same inode (same data).

```c
int link(const char *oldpath, const char *newpath);
```

**Effect**:

```
Initial:
  filename: "file1"
  inode 42: [data]

After: link("file1", "file2")

  filename "file1" → inode 42
  filename "file2" → inode 42 (same inode!)

Inode 42 now has link count = 2
```

**Important properties**:

- Both names refer to same data
- Deleting one name doesn't delete data (as long as other names exist)
- File deleted only when link count reaches 0
- Can't create hard links to directories (would create loops)
- Can't cross filesystems

**Deleting**:

```c
unlink(filename);  // Decrements link count
```

#### Symbolic Links (Soft Links)

Symbolic link is a special file containing a pathname.

```c
int symlink(const char *target, const char *linkpath);
```

**Effect**:

```
symlink("file1", "link1")

Creates:
  filename "link1" → [special file containing string "file1"]

When you access "link1":
  1. Kernel reads the link
  2. Sees it points to "file1"
  3. Resolves "file1"
  4. Returns that file's data
```

**Advantages over hard links**:

- Can point to directories
- Can cross filesystems
- Can point to non-existent files (not immediately broken)
- Can be circular (kernel prevents infinite loops)

**Disadvantages**:

- Extra indirection (slower)
- Broken if target deleted
- Takes up inode even if target is directory

#### Comparison

| Feature                   | Hard Link          | Symlink            |
| ------------------------- | ------------------ | ------------------ |
| Same inode?               | Yes                | No (contains path) |
| Cross filesystem?         | No                 | Yes                |
| Can link directories?     | No                 | Yes                |
| Extra indirection?        | No                 | Yes                |
| Broken if target deleted? | No (name survives) | Yes                |

### Path Name Resolution

When opening `/home/user/file.txt`:

1. **Read root inode** (inode #2 in most UNIX systems)
2. **Look up "home" in root directory data**
   - Find: ("home" → inode 5000)
3. **Read inode 5000 data** (directory entries in /home)
4. **Look up "user" in /home data**
   - Find: ("user" → inode 7200)
5. **Read inode 7200 data** (directory entries in /home/user)
6. **Look up "file.txt" in /home/user data**
   - Find: ("file.txt" → inode 9500)
7. **Read inode 9500** (the file's metadata)

**Performance**: Multiple disk I/Os even for single file open!

**Optimization**: Kernel caches inodes and directory entries to reduce I/Os.

---

## Chapter 40: File System Implementation

### VSFS: Very Simple File System

A simplified file system design to illustrate concepts used in real systems (ext2, ext3, etc.).

### On-Disk Layout

```
┌──────┬───────┬───────┬─────────┬──────────┐
│      │ Inode │ Data  │ Inode   │   Data   │
│Super │Bitmap │Bitmap │  Table  │  Region  │
│block │       │       │         │          │
├──────┼───────┼───────┼─────────┼──────────┤
│ 0    │ 1     │ 2     │ 3-7     │ 8-63     │
└──────┴───────┴───────┴─────────┴──────────┘

Examples:
- Superblock (block 0): FS metadata, inode count, data block count
- Inode bitmap (block 1): 1 bit per inode (1=allocated, 0=free)
- Data bitmap (block 2): 1 bit per data block
- Inode table (blocks 3-7): Array of inode structures
- Data region (blocks 8-63): User data
```

### Inode Structure

Each inode contains file metadata:

```c
struct inode {
    ino_t i_number;          // Inode number
    mode_t i_mode;           // Permissions + file type
    uid_t i_uid;             // Owner user ID
    gid_t i_gid;             // Owner group ID
    off_t i_size;            // File size in bytes
    time_t i_atime;          // Access time
    time_t i_mtime;          // Modification time
    time_t i_ctime;          // Change time
    unsigned int i_links;    // Hard link count
    uint32_t i_blocks[12];   // Direct block pointers (direct)
    uint32_t i_indirect;     // Indirect block pointer
    uint32_t i_dindirect;    // Double indirect pointer
    uint32_t i_tindirect;    // Triple indirect pointer
};
```

### Multi-Level Index: Block Pointers

To support large files with reasonable inode size, use indirect blocks.

#### Direct Blocks

```
Small file (≤ 48 KB):
inode.i_blocks[0-11] = [LBA 100, LBA 101, ..., LBA 111]

File data:
Block 0 at LBA 100: bytes 0-4095
Block 1 at LBA 101: bytes 4096-8191
...
Block 11 at LBA 111: bytes 45056-49151
```

#### Indirect Blocks

For file > 48 KB:

```
inode.i_indirect = pointer to indirect block (e.g., LBA 500)

LBA 500 (indirect block):
[LBA 200, LBA 201, LBA 202, ..., LBA 1223]
(1024 pointers, 4 bytes each in 4KB block)

This adds: 1024 × 4 KB = 4 MB more capacity
Total file size: 48 KB + 4 MB ≈ 4 MB
```

#### Double-Indirect Blocks

For file > 4 MB:

```
inode.i_dindirect = pointer to double-indirect block

LBA 5000 (double-indirect block):
[LBA 600, LBA 601, ..., LBA 1623]

Each entry points to an indirect block:
  LBA 600: [LBA 300, LBA 301, ..., LBA 1323]
  LBA 601: [LBA 1400, LBA 1401, ..., LBA 2423]
  ...

Total capacity: 1024 × 4 MB = 4 GB
```

#### Triple-Indirect Blocks

For file > 4 GB:

```
inode.i_tindirect = pointer to triple-indirect block

Triple-indirect → Double-indirect → Indirect → Data

Total capacity: 1024 × 4 GB = 4 TB
```

### Example: Multi-Level Index Calculation

**Find 5th block of file with inode 10**:

```c
// Assuming 4KB blocks, 4-byte addresses
#define BLOCKSIZE 4096
#define PTRS_PER_BLOCK (BLOCKSIZE / sizeof(uint32_t))  // 1024

// Read inode
inode *in = read_inode(10);

// Block 5 is within direct pointers (blocks 0-11)
if (block_num < 12) {
    lba = in->i_blocks[block_num];  // LBA 105
    read_block(lba);  // Get block 5
}

// Or if block 5000 (after all directs and some indirects):
// Block 5000 - 12 (direct) = 4988
// 4988 / 1024 = 4 (which indirect block)
// 4988 % 1024 = 872 (which pointer in indirect block)

int block_offset = 5000 - 12;          // 4988
int indirect_index = block_offset / 1024;  // 4
int pointer_index = block_offset % 1024;   // 872

// Read 4th indirect block
indirect_block = read_block(in->i_indirect + indirect_index);

// Read target block pointer
lba = indirect_block[pointer_index];  // LBA of block 5000
read_block(lba);
```

### Directory Organization

A directory is a special file containing (name, inode_number) mappings:

```
Directory data (in disk blocks):

├─ Entry 1:
│  ├─ Name: "."
│  ├─ Inode #: 10 (self-reference)
│  └─ Reclen: 12 bytes
│
├─ Entry 2:
│  ├─ Name: ".."
│  ├─ Inode #: 2 (parent directory)
│  └─ Reclen: 12 bytes
│
├─ Entry 3:
│  ├─ Name: "file1"
│  ├─ Inode #: 45
│  └─ Reclen: 12 bytes
│
└─ Entry 4:
   ├─ Name: "longfilename"
   ├─ Inode #: 67
   └─ Reclen: 20 bytes (extra space for deleted entries)
```

**On-disk format** (simplified):

```c
struct dirent {
    ino_t inode;           // Inode number
    uint16_t reclen;       // Record length (for variable-length names)
    uint8_t namelen;       // Length of name
    char name[255];        // Filename (variable length)
};
```

### Free Space Management

#### Bitmaps

Simple: Use bit per object (inode or data block).

```
Inode bitmap (block 1):
11001010_00110001_...
│││││││
└────────→ Inode 0: allocated (1)
 └───────→ Inode 1: allocated (1)
  └──────→ Inode 2: free (0)
   └─────→ Inode 3: allocated (1)
    ...

Advantages:
- Simple to implement
- Easy to find free blocks
- Compact representation

Disadvantages:
- Requires scanning to find free space
- Can be slow for large filesystems

Optimization: Keep track of "first free block", start scanning there
```

### File System Access Paths

#### Reading a File: `/foo/bar` (3 blocks)

```
Timeline:

1. open("/foo/bar", O_RDONLY)
   ├─ Read root inode (inode #2) [1 I/O]
   ├─ Read root directory data [1 I/O]
   │  └─ Find "foo" → inode 44
   ├─ Read foo inode [1 I/O]
   ├─ Read foo directory data [1 I/O]
   │  └─ Find "bar" → inode 5
   └─ Read bar inode [1 I/O]
   [Total: 5 I/Os just to open!]

2. read(fd, buf, 4096)  // Read 1st block
   ├─ Read bar's inode (cache hit) [0 I/Os]
   ├─ Get 1st block LBA from inode
   └─ Read block 0 from disk [1 I/O]

3. read(fd, buf, 4096)  // Read 2nd block
   ├─ Get 2nd block LBA from inode (cache hit) [0 I/Os]
   └─ Read block 1 from disk [1 I/O]

4. read(fd, buf, 4096)  // Read 3rd block
   └─ Read block 2 from disk [1 I/O]

5. close(fd)
   [0 I/Os]

Total I/Os: 5 (open) + 3 (read) = 8 I/Os
```

**Without caching**: Would revisit inodes multiple times, much worse

**With caching**: Inodes and directories cached in memory, major optimization

#### Writing a File (Creating `/foo/bar`)

```
Creating file involves multiple steps:

1. Allocate inode
   ├─ Read inode bitmap [1 I/O]
   ├─ Mark inode 44 as allocated
   ├─ Write inode bitmap [1 I/O]
   └─ Write new inode [1 I/O]

2. Update parent directory (/foo)
   ├─ Read parent inode (cached)
   ├─ Read parent directory data [1 I/O]
   ├─ Add new entry: ("bar" → inode 44)
   ├─ Write updated directory data [1 I/O]
   └─ Write updated parent inode [1 I/O]

3. Write data (allocating block)
   ├─ Read data bitmap [1 I/O]
   ├─ Mark block allocated
   ├─ Write data bitmap [1 I/O]
   ├─ Fetch updated parent inode
   ├─ Write new inode with data block pointer [1 I/O]
   └─ Write data block [1 I/O]

Total: ~10 I/Os for a single small file creation!

MASSIVELY EXPENSIVE compared to simple operations.
```

### Caching and Buffering

Modern file systems use aggressive caching:

#### Unified Page Cache

Instead of separate file system cache and virtual memory page cache:

- Use single **page cache** for both VM and filesystem
- Dynamically allocate memory between uses
- File data once in memory serves both roles

**Benefits**:

- Avoid double caching
- Better memory utilization
- Sequential file access benefits from paging

**Example**:

```
Without unified cache:
┌─────────────────┐
│  File cache:    │
│  Recently used  │
│  file blocks    │
└─────────────────┘ ← Static size, wasteful if unused

┌─────────────────┐
│  Page cache:    │
│  Virtual memory │
│  pages          │
└─────────────────┘ ← Also static size

With unified cache:
┌─────────────────────────────────┐
│  Unified page cache:            │
│  File blocks + VM pages         │
│  Dynamic allocation based on    │
│  usage patterns                 │
└─────────────────────────────────┘
```

#### Write Buffering

The file system doesn't write immediately:

- Buffers writes in memory for ~5-30 seconds
- Batches multiple writes together
- Reduces I/O load significantly

**Trade-off**:

- **Performance**: Writes appear instant, aggregated to disk when efficient
- **Durability**: If crash occurs, writes in buffer are lost

**Example**:

```
Rapid writes:
write(fd, buf1, 4096);    // Buffered in memory
write(fd, buf2, 4096);    // Buffered in memory
write(fd, buf3, 4096);    // Buffered in memory
// After 5 seconds, all 3 blocks written in single I/O sequence
```

**For critical data**, use `fsync()`:

```c
write(fd, critical_data, size);
fsync(fd);  // Force to disk immediately
```

---

## Chapter 41: Locality and the Fast File System

### The Problem: Fragmentation in Old UNIX FS

Early UNIX file system suffered severe performance degradation:

```
Old file system performance: ~2% of disk peak bandwidth!

Reasons:
1. Inodes stored far from data blocks
   └─ Opens require expensive seeks

2. Free space fragmentation
   └─ Files spread across disk
   └─ Sequential access becomes random

Example fragmentation:
Initial layout:
[A1][A2][B1][B2][C1][C2][D1][D2]

After deleting B and D:
[A1][A2][--][--][C1][C2][--][--]

Creating file E (4 blocks):
[A1][A2][E1][E2][C1][C2][E3][E4]

File E is fragmented!
Sequential read: seek after reading E2, then seek after C reading C2
```

### Solution: FFS (Fast File System)

Key insight: **File systems must be disk-aware**

Organize file system to minimize seeks and respect disk geometry.

### Cylinder Groups

FFS divides disk into **cylinder groups** (sets of consecutive cylinders).

```
Physical disk:
┌─────────────────────────────────────────────────┐
│ Platter with multiple surfaces                  │
│                                                  │
│ Surface 0: [Track 0][Track 1][Track 2]...       │
│ Surface 1: [Track 0][Track 1][Track 2]...       │
│ Surface 2: [Track 0][Track 1][Track 2]...       │
│ Surface 3: [Track 0][Track 1][Track 2]...       │
│                                                  │
│ Cylinder 0 = Track 0 on all surfaces            │
│ Cylinder 1 = Track 1 on all surfaces            │
│ Cylinder 2 = Track 2 on all surfaces            │
│ ...                                              │
└─────────────────────────────────────────────────┘

FFS grouping:
┌─────────────────┬──────────────────┬──────────────────┐
│ Cylinder Group 0│ Cylinder Group 1 │ Cylinder Group 2 │
│ (Cylinders 0-9) │ (Cylinders 10-19)│ (Cylinders 20-29)│
│                 │                  │                  │
│ ┌──────────────┐│┌──────────────┐ │┌──────────────┐ │
│ │ Superblock   │││ Superblock   │ ││ Superblock   │ │
│ │ (replicated) │││ (replicated) │ ││ (replicated) │ │
│ ├──────────────┤│├──────────────┤ ├──────────────┤ │
│ │ Inode bitmap │││ Inode bitmap │ ││ Inode bitmap │ │
│ ├──────────────┤│├──────────────┤ ├──────────────┤ │
│ │ Data bitmap  │││ Data bitmap  │ ││ Data bitmap  │ │
│ ├──────────────┤│├──────────────┤ ├──────────────┤ │
│ │ Inode table  │││ Inode table  │ ││ Inode table  │ │
│ ├──────────────┤│├──────────────┤ ├──────────────┤ │
│ │ Data blocks  │││ Data blocks  │ ││ Data blocks  │ │
│ └──────────────┘│└──────────────┘ │└──────────────┘ │
└─────────────────┴──────────────────┴──────────────────┘
```

### FFS Allocation Policies

**Philosophy**: Keep related stuff together; separate unrelated.

#### Directory Allocation

When creating a new directory:

1. Find cylinder group with low number of allocated directories
2. Ensure high number of free inodes available
3. Allocate directory inode and data in that group

**Rationale**: Spread directories to balance load; group files with their directory.

#### File Allocation

When creating a new file:

1. Place inode in same cylinder group as parent directory's inode
2. When allocating data blocks, use same group as inode

**Rationale**:

- Parent directory frequently accessed with files
- Nearby blocks minimize seek distance
- Preserve directory locality

#### Example Layout

```
Initial setup: 3 directories (/, /a, /b), 4 files in same groups

Allocation:
Group 0: Inode for /,     containing (/a, /b, /c entries)
         File data for /dir

Group 1: Inode for /a,    containing (/a/file1, /a/file2 entries)
         File inodes and data for files in /a
         (/a/file1, /a/file2, etc.)

Group 2: Inode for /b,    containing (/b/file3 entries)
         File inodes and data for /b/file3

Result: Accessing /a/file1 requires minimal seeking
        All /a data close together
        /b data isolated from /a
```

### Handling Large Files

Large files could fill an entire cylinder group:

```
Without special handling:
File "/large" with 50 MB allocated to Group 0:

Group 0: [entire 50 MB of /large]
Group 1: [empty - no room for other files in group]
Group 2: [empty]

Problem: Other files can't use Group 0; locality lost
```

**Solution**: Large File Exception

After allocating first chunk (direct pointers, ~48 KB), allocate subsequent blocks in different groups:

```
With large file exception:

File "/large" (4 MB total)
├─ Blocks 0-11 (48 KB direct) → Group 0
├─ Next 1 MB (first indirect) → Group 1 (chosen for low utilization)
├─ Next 1 MB (second indirect) → Group 2
├─ Next 1 MB (third indirect) → Group 3
└─ Final part → Group 4

Result: File spans groups but each group not overwhelmed
        Sequential read still works (chunk size large enough)
        Other files can use remaining space in groups
```

**Chunk Size Calculation**:

To amortize positioning overhead and achieve desired bandwidth utilization:

Let F = desired fraction of peak bandwidth (e.g., F = 0.9 for 90%)

$$D = \frac{F \times R_{peak} \times T_{position}}{1 - F}$$

**Example**:

```
Disk: 100 MB/s transfer, 10 ms seek/rotation overhead
Target: 90% of peak bandwidth

D = (0.9 × 100 × 0.01) / (1 - 0.9)
  = (0.9 × 100 × 0.01) / 0.1
  = 0.9 / 0.1
  = 9 MB

Conclusion: Allocate 9 MB per chunk to achieve 90% of peak bandwidth
            (spend 10 ms seeking, 90 ms transferring, ratio 1:9)
```

### Disk Layout Optimization

Modern disks with track buffering handle this automatically, but historically FFS used **parameterization**:

```
Parameterized block placement (historical):

Problem: Disk rotates between requests
         If reading consecutive blocks, 2nd block rotates past head

Solution: Skip blocks during layout

Disk sectors arranged:
Physical:  [0][1][2][3][4][5][6][7][8][9][10][11]
Logical:   [0][2][4][6][8][10][1][3][5][7][9][11]

By time processor interacts with disk, rotation moves head to next
block's logical position.

Result: Can read consecutive logical blocks without extra rotations
        (though only ~50% peak bandwidth without track buffer)

Modern disks: Internal track buffer makes this unnecessary
```

### FFS Features

#### Sub-block Allocation

Small files waste space in 4KB blocks:

```
Problem: Creating 1KB file in 4KB blocks
         3 KB wasted per file

Solution: 512-byte sub-blocks
├─ Allocate 512-byte blocks for small files
├─ As file grows to 4KB, write all data to full 4KB block
├─ Free sub-blocks

Optimization: libc buffers writes, avoids fragmentation

Result: Small files use appropriate size; no massive waste
```

#### Features for Usability

FFS introduced:

- **Long filenames**: Up to 255 characters (vs. 14 byte limit)
- **Symbolic links**: Can point to directories and cross filesystems
- **Atomic rename**: `rename()` operation for safe file replacement

### FFS Measurements

Empirical study (SEER traces) shows namespace locality:

```
File access patterns:
- ~7% of accesses are to same file (distance 0)
- ~33% of accesses are to files in same directory (distance 1)
- ~25% of accesses are to files in nearby directories (distance 2)

Conclusion: FFS's locality assumptions well founded!
           Files accessed in directory groups as expected
```

---

## Chapter 42: Crash Consistency & Journaling

### The Crash Consistency Problem

When system crashes mid-update, file system can be left inconsistent.

### Example: Appending to File

To append one block to file, three structures must be updated:

```
Initial state:
┌────────┬───────┬────────┐
│ Inode  │ Bitmap│ Data   │
├────────┼───────┼────────┤
│[v1]    │ [v1]  │ [D_a]  │
│size=1  │ 00001011
│ptr→4   │
└────────┴───────┴────────┘
  Block 2  Block 1  Block 4

Goal state (append to block 5):
┌────────┬───────┬────────┐
│ Inode  │ Bitmap│ Data   │
├────────┼───────┼────────┤
│[v2]    │ [v2]  │[D_a][D_b]
│size=2  │ 00001100
│ptr→4,5 │
└────────┴───────┴────────┘
  Block 2  Block 1  Blocks 4,5

Modifications needed:
1. I[v2]: Update inode (new size, new pointer)
2. B[v2]: Update bitmap (mark block 5 as allocated)
3. D_b:   Write new data block
```

### Crash Scenarios

If system crashes after only 1-2 of 3 writes complete:

#### Case 1: Only D_b written

```
Disk state: Inode still points to block 4, bitmap says 4 is allocated
Result: No corruption, append lost (acceptable)
```

#### Case 2: Only I[v2] written

```
Disk state: Inode points to block 5, bitmap says 5 NOT allocated
Result: INCONSISTENT! Inode-bitmap mismatch!
        Reading file may get garbage from block 5
```

#### Case 3: Only B[v2] written

```
Disk state: Bitmap says block 5 allocated, no inode points to it
Result: INCONSISTENT! Space leak - block 5 never reused
        (in lost+found or lost forever)
```

#### Case 4: I[v2] and B[v2] written, not D_b

```
Disk state: Inode and bitmap agree, but block 5 contains garbage
Result: Consistent metadata, but user gets garbage data!
        (subtle corruption)
```

### Solution 1: fsck (File System Checker)

Scan entire disk and repair inconsistencies:

#### FSCK Phases

1. **Superblock Check**: Verify FS metadata is reasonable
2. **Free Block Scan**: Read all inodes, build correct bitmap
3. **Inode State**: Check each inode for corruption
4. **Inode Link Count**: Verify link counts match directory tree
5. **Duplicate Pointers**: Find inodes pointing to same block
6. **Bad Pointers**: Remove pointers outside valid range
7. **Directory Checks**: Verify directory structure

#### Example FSCK Recovery

```
Crash left:
Inode 5: points to block 5
Bitmap: block 5 marked free (0)

FSCK:
1. Scans all inodes, finds inode 5 points to block 5
2. Reads bitmap, sees block 5 is free
3. Decides to trust inode (marks block 5 as allocated in new bitmap)
4. Writes corrected bitmap to disk

Result: Consistent state, but:
   - If block 5 was actually free, just corrupted inode recovery
   - If crash happened during different operation, recovery wrong
   - Can't distinguish "what operation was in progress?"
```

#### FSCK Problems

- **Slow**: Must scan entire disk (can take hours on modern disks)
- **O(disk_size)**: Recovery time scales with disk capacity
- **Limited repair capability**: Can't recover garbage data

### Solution 2: Journaling (Write-Ahead Logging)

Before modifying on-disk structures, write log entry describing update.

If crash, replay log entries to finish or discard incomplete operation.

#### Basic Protocol

```
Journal (on disk, safe location):

TxB ← Transaction begin
│    (TID=1, describes update)
├─ I[v2]  ← Inode data
├─ B[v2]  ← Bitmap data
├─ D_b    ← Data block content
│
TxE ← Transaction end
    (TID=1, confirms ready to apply)

Then checkpoint:
I[v2] → Block 2
B[v2] → Block 1
D_b → Block 5
```

#### Why Order Matters

**Unsafe ordering**:

```
Write all to journal at once:
TxB I[v2] B[v2] TxE D_b

Disk might reorder writes!
Result: TxB I[v2] B[v2] TxE written before D_b

Recovery sees complete transaction, replays it
Inode points to D_b, but D_b not written yet (garbage!)
```

**Safe ordering**:

```
Step 1: Write transaction content (TxB, data, metadata)
        TxB I[v2] B[v2] D_b written

Step 2: Write transaction commit (TxE)
        TxE written
        (atomicity guaranteed: 512-byte block write is atomic)

Recovery:
- If TxE present: Transaction committed, apply it
- If TxE missing: Transaction incomplete, skip it
```

#### Data Journaling Example

```
Timeline:

┌─────────────────┬────────────────┬──────────────┐
│  Journal Write  │ Checkpoint    │ Free         │
├─────────────────┼────────────────┼──────────────┤
│ [1] Write TxB   │ [5] Write I[v2]│ [7] Update  │
├─ I[v2]         │ [6] Write B[v2]│    journal   │
├─ B[v2]         │ [7] Write D_b  │    superblock
├─ D_b           │                │              │
│ [2] Write TxE  │                │              │
│ [3] Wait complete
│                │
│ Journal stable │ Checkpoint starts after (3)
└─────────────────┴────────────────┴──────────────┘
```

**Crash scenarios with journaling**:

- **Crash before TxE**: Transaction incomplete, skipped on recovery
- **Crash after TxE**: Transaction complete but not checkpointed
  - Recovery replays transactions from journal
  - Checkpoint writes them to final location
  - Effective recovery: O(log_size) instead of O(disk_size)

#### Metadata Journaling (Ordered Journaling)

Data journaling writes data twice (to journal, then to disk) - expensive.

**Optimization**: Only journal metadata, not user data.

```
Protocol:
1. Write data block D_b to disk (not journaled)
2. Journal: TxB I[v2] B[v2] TxE
3. Checkpoint: I[v2] B[v2] to disk

Ordering requirement: D_b MUST REACH DISK before checkpoint!

Reason: If inode checkpointed first, it points to D_b that might not exist
        (inode pointer must never point to unwritten-to-disk data)
```

**Write order**:

```
1. Data write: Write D_b to disk (may wait optional)
2. Journal write: TxB, metadata, TxE
3. Checkpoint: Metadata blocks

Result: Inode always points to valid data (already on disk)
```

#### Transaction Batching

Instead of journaling each operation separately, batch multiple operations:

```
User ops:
create("/foo/file1")
create("/foo/file2")
create("/foo/file3")

Without batching: 3 separate transactions to journal = 3 × I/O cost

With batching (commit every 5 seconds):
┌──────────────────────────────────────────┐
│ Single transaction in journal:           │
├──────────────────────────────────────────┤
│ TxB                                      │
│ ├─ file1: I[new], B[bitmap], D[dir]    │
│ ├─ file2: I[new], B[bitmap], D[dir]    │
│ ├─ file3: I[new], B[bitmap], D[dir]    │
│ TxE                                      │
│                                          │
│ Checkpoint file1, file2, file3 all at   │
│ once (merge seeks)                       │
└──────────────────────────────────────────┘

Benefits:
- 1 journal write instead of 3
- Single checkpoint for all 3 operations
- Much faster (similar to write buffering)
```

#### Journal Circular Buffer

Journal is finite size; must reuse space after checkpoint:

```
Journal timeline:

TxA TxB TxC TxD TxE ...
│   │   │   │   │

After checkpointing TxA:
    TxB TxC TxD TxE ... [TxF - new]
    ↑
    Space freed, can reuse for new transactions
```

#### Special Case: Block Reuse

Problem occurs when block is freed, then immediately reused:

```
Initial: directory foo in block 1000, inode 50
Journal: TxA writes directory foo to journal

Delete directory foo:
- Remove inode 50
- Free block 1000

Create new directory bar using inode 51:
- Allocate block 1000 (reused!)
- Directory bar goes to block 1000

Now journal has:
TxA: [...block 1000 contains foo data...]

If crash and recovery replays TxA:
Block 1000 overwritten with old foo data!
Directory bar corrupted!
```

**Solution**: Revoke records

When deleting block, write "revoke" record to journal:

```
TxA: Writes directory foo to block 1000
TxDelete: Revoke record for block 1000 (says "don't replay old writes to this block")
TxB: Writes directory bar to block 1000

On recovery:
1. Scan for revoke records
2. During replay, skip any revoke'd blocks
3. Replay TxA skips block 1000 (revoked)
4. Replay TxB correctly writes bar to block 1000
```

---

## Chapter 43: Log-Structured File Systems

### Motivation

Traditional file systems (FFS) spread writes across disk:

```
Create file:
- Allocate inode: write to inode table
- Allocate block: write to bitmap
- Write data block
- Update parent directory: write to directory block
- Update parent inode

Result: ~6-10 disk I/Os for single file creation
        Each I/O involves seek + rotational delay
        Efficiency: ~10% of peak bandwidth
```

**Key insight**: All writes should be sequential to one location!

### LFS (Log-Structured File System) Approach

Buffer all updates in memory; write as single sequential batch to disk:

```
In-memory buffer (segment):

[D1][D2][Inode_1][Inode_2][Bitmap][Dir_data]...

When segment full (e.g., 1 MB):
→ Write entire segment sequentially to disk

Result:
- All writes in one long I/O
- No seeks
- Efficiency: ~90-95% of peak bandwidth!
```

### Structure: Segments

A segment is large-ish chunk of data written sequentially.

```
Segment (e.g., 1 MB):

┌───────────────────────────────────────────────┐
│ Segment Summary Block                         │
│ ├─ Describes each block in segment            │
│ │  [(inode_5, offset_0), (inode_7, offset_1)] │
│ │   ...                                        │
│ └─ Used to determine block liveness           │
├───────────────────────────────────────────────┤
│ Inode 5's data blocks                         │
│ Inode 7's inode                               │
│ Inode 7's data blocks                         │
│ ...                                            │
├───────────────────────────────────────────────┤
│ Inode map (imap) updates                      │
│ ├─ (inode_5 → new_disk_address)               │
│ └─ (inode_7 → new_disk_address)               │
└───────────────────────────────────────────────┘
```

### Solution: Inode Map (Indirection)

Problem: Inodes are scattered throughout disk; can't find them easily.

Solution: **Inode map** provides LBA lookup:

```
imap[inode_number] → latest_disk_address_of_inode

imap structure:
imap[0] = LBA 100    (inode 0 lives at block 100)
imap[1] = LBA 2050   (inode 1 lives at block 2050)
imap[2] = free       (inode 2 not allocated)
...

Usage:
To read inode 5:
1. Look in imap: imap[5] = LBA 205
2. Read block at LBA 205
3. Have inode 5
```

**Advantages**:

- Inode can move anywhere on disk
- Update inode location by updating imap
- One indirection layer isolates inode movement

### Checkpoint Region

imap itself would require seeking if stored at fixed location.

Solution: **Checkpoint region** (CR) updated periodically:

```
┌────────────────────────────────────┐
│ Checkpoint Region (at LBA 0)       │
│ ├─ Pointers to latest imap chunks  │
│ ├─ Timestamp                       │
│ └─ Other metadata                  │
└────────────────────────────────────┘

On-disk layout:
LBA 0:   Checkpoint Region
LBA 100-199: imap chunk 0
LBA 200-299: imap chunk 1
...
LBA 1000: First segment
LBA 2000: Second segment
...

Finding files:
1. Read CR from fixed LBA 0
2. Get pointers to imap chunks
3. Read imap chunks into memory (usually cached)
4. Use imap to find inode locations
5. Read inodes and data
```

**Atomicity**: CR updated with two copies at disk ends:

- Write CR copy 1
- Write body
- Write CR copy 2

If crash between copies, use most recent CR with valid timestamps.

### Reading File from Disk

**Step 1**: Boot/mount

```
Read CR from fixed location (LBA 0)
Get pointers to all imap chunks
Read imap into memory and cache
```

**Step 2**: Open file "/foo/bar"

```
Look up "foo" in root directory
Get inode_foo
Look in imap: inode_map[inode_foo] → LBA_foo_inode
Read inode at LBA_foo_inode

Look up "bar" in foo's directory
Get inode_bar
Look in imap: inode_map[inode_bar] → LBA_bar_inode
Read inode at LBA_bar_inode (usually cached, 0 I/O)
```

**Step 3**: Read data block

```
Consult bar's inode for block pointers
Read data block from disk
Like normal filesystem (imap cached, so same performance)
```

### The Garbage Collection Problem

LFS never overwrites in place - just writes new versions:

```
File update:
Old: [Data_old][Inode_old]  at LBA 1000-1001
New: [Data_new][Inode_new]  at LBA 2000-2001

On disk:
LBA 1000-1001: Old data + inode (now GARBAGE!)
LBA 2000-2001: New data + inode (live)
```

Over time, disk fills with garbage (old copies of files).

**Solution**: Garbage collection

#### Block Liveness Determination

Which blocks are live vs. garbage?

Segment summary block records:

```c
struct segment_summary {
    struct block_info {
        ino_t inode_number;      // Which file?
        uint32_t block_offset;   // Which block in file?
        uint32_t version;        // Version number
    } blocks[];
};
```

To check if block D at LBA A is live:

```c
(inode_num, offset, version) = segment_summary[A];

// Read inode
inode = read_inode(imap[inode_num]);

// Check if inode points to this location
if (version == inode.version && inode.block_pointers[offset] == A) {
    // Block is LIVE
} else {
    // Block is GARBAGE (can be freed)
}
```

#### Cleaning Policy

**When to clean**:

- Periodically (e.g., every 30 seconds)
- When disk utilization high
- During idle time

**Which segments to clean**:

- **Hot segments**: Frequently overwritten data
  - Best policy: Wait long (more data becomes garbage)
- **Cold segments**: Stable data, little overwriting
  - Best policy: Clean sooner (garbage won't increase)

LFS uses heuristic to segregate hot vs. cold segments and clean accordingly.

#### Cleaning Example

```
Disk before cleaning:
Segment 0: [L L G G L]  (L=live, G=garbage)
           └─ Utilization: 60%
Segment 1: [L L L G G]
           └─ Utilization: 60%

After garbage collection of Segment 0:

1. Read segment 0
2. Identify live blocks: [L L G G L]
3. Compact live blocks together
4. Write to new segment location:

Segment N: [L L L]  (only 3 live blocks, old garbage discarded)

Segment 0: FREE (can be reused!)

Result: Space reclaimed from garbage, enables new writes
```

### Crash Recovery

LFS maintains log of segments:

```
Checkpoint Region → Latest segment head/tail
                 → Segment links:
                    Segment N → points to Segment N+1
                    Segment N+1 → points to Segment N+2
```

**Recovery**:

1. Read CR (identifies last valid snapshot)
2. Read through log from CR endpoint
3. Use **roll-forward** technique:
   - Replay any valid transactions beyond last CR
   - Recover writes since last CR update
   - Faster than fsck, complete recovery including uncommitted data

---

## Chapter 44: Flash-Based SSDs

### Flash Storage Technology

Unlike disks with mechanical heads, **NAND flash** uses transistors to store charge:

```
SLC (Single-Level Cell):
  Charge level → Binary value
  Low  → 0
  High → 1

MLC (Multi-Level Cell):
  4 charge levels → 2 bits
  Low      → 00
  Mid-low  → 01
  Mid-high → 10
  High     → 11

TLC (Triple-Level Cell):
  8 charge levels → 3 bits per cell
```

### Flash Organization

```
Flash chip:
┌────────────────────────────┐
│ Bank/Plane 0               │
├─────────────┬──────────────┤
│   Block 0   │   Block 1    │
├─────┬───────┼──────┬───────┤
│Pg0 │Pg1│... │Pg0 │Pg1│... │
└─────┴───────┴──────┴───────┘

Flash bank organization:
- Blocks (~128-256 KB each)
  └─ Pages (~2-4 KB each, typically)

Example: 4 blocks, 4 pages per block:
┌─────────────┬─────────────┬─────────────┬─────────────┐
│   Block 0   │   Block 1   │   Block 2   │   Block 3   │
├──┬──┬──┬──┬─┼──┬──┬──┬──┬─┼──┬──┬──┬──┬─┼──┬──┬──┬──┬─┤
│00│01│02│03│ │04│05│06│07│ │08│09│10│11│ │12│13│14│15│ │
└──┴──┴──┴──┴─┴──┴──┴──┴──┴─┴──┴──┴──┴──┴─┴──┴──┴──┴──┴─┘
```

### Flash Operations

#### Read Page

```
Read(page_number) → data

Performance: ~25-75 µs (very fast)
Advantage: Same speed regardless of location (no mechanical delay!)
          Random access instead of sequential disk
```

#### Erase Block

```
Erase(block_number) → all bits set to 1

Performance: ~1500-4500 µs (slow, expensive)
Effect: Erases entire block, making all pages programmable
Important: Must copy live data elsewhere before erasing!
```

#### Program Page

```
Program(page_number, data)
Effect: Change some 1's to 0's to write desired value
Performance: ~200-1350 µs
Requirement: Block must be erased first
             Page state must be ERASED (not VALID)
```

### Flash State Transitions

```
Page states:

INVALID → erase_block() → ERASED
                            ↓
ERASED → program_page() → VALID
            ↑                ↓
            └── only by erasing entire block ──┘
```

**Key insight**: To write to a page, entire block must be erased!

### Write Amplification

Erasing block destroys other data; must copy live data first:

```
Scenario: Rewrite block with some old data still live

Block contents: [Data_A (live)][Data_B (live)][Data_C (old)]

To write new data:
1. Read Data_A and Data_B (copy to memory)
2. Erase block (destroys all)
3. Program Data_A, Data_B, Data_New

Physical I/O: Wrote 3 pages to change 1 page
              Write amplification = 3

Definition:
Write amplification = Total bytes written to flash
                      ─────────────────────────────
                      Total bytes user wrote
```

**Impact**: With WA = 3, user write to 1 GB = 3 GB to flash

- Wears out flash faster
- Reduces throughput
- Wastes bandwidth

### Flash Translation Layer (FTL)

Hardware controller that:

1. Translates logical block addresses (LBA) to physical flash blocks
2. Handles wear leveling
3. Manages garbage collection
4. Reduces write amplification

```
Logical block interface (user):
read(LBA) / write(LBA) / erase(LBA)
        ↓
FTL (inside SSD controller)
├─ Address translation (LBA → physical page)
├─ Wear leveling (distribute erases)
├─ Garbage collection (compaction)
├─ Bad block management
└─ Fault tolerance
        ↓
Physical flash operations:
read_flash(physical_page)
erase_flash(block)
program_flash(physical_page)
```

### Wear Leveling

Flash blocks degrade with P/E cycles (Program/Erase):

```
MLC flash lifetime: ~10,000 P/E cycles
SLC flash lifetime: ~100,000 P/E cycles

Without wear leveling:
Block 0: Write frequently → 10,000 P/Es → DEAD after weeks
         User data lost!

With wear leveling:
Distribute writes across blocks:
Block 0: 100 P/Es
Block 1: 100 P/Es
...
Block 100: 100 P/Es

Result: Each block reaches 10,000 P/Es after weeks of operation
        All blocks die simultaneously (acceptable end-of-life)
        Device lifetime: 10,000 / 100 = 100 weeks ≈ 2 years

Dynamic wear leveling:
- Track P/E count per block
- Remap hot blocks to low-use blocks
- Prevent any block from aging faster
```

### Garbage Collection

Flash GC is similar to LFS GC:

```
FTL reads partially-used flash blocks
(some pages invalid due to overwrites)

Example:
Block 100: [L L G L G]  (L=live, G=garbage)
           Utilization: 60%

GC:
1. Read live pages [1, 2, 4]
2. Program into new block
3. Erase old block 100
4. Block 100 now free for new writes

Tradeoff:
- Benefits: Recovers space
- Cost: Erasing (slow) + rewrites (wear amplification)
- Policy: Collect when utilization low, idle time, etc.
```

### SSD Performance Characteristics

```
Read:
├─ Sequential: ~500 MB/s (all chips parallel)
└─ Random: ~100K IOPS @ 4KB (much better than disk!)

Write:
├─ Sequential: ~400 MB/s (all chips, wear-leveled)
└─ Random: ~30K IOPS (limited by GC, wear leveling)

Latency:
├─ Read:  ~25 µs minimum
├─ Write: ~500 µs minimum
└─ GC pause: Can stall writes (when blocks full)

Comparison to disk:
                SSD         Disk
Random read:    ~150 µs     ~7000 µs  (45× faster!)
Sequential read ~3 MB/s     ~100 MB/s (similar)
                (per chip)  (all heads)
```

---

## Chapter 45: Data Integrity and Protection

### Data Corruption Sources

Disk writes are not perfectly reliable:

```
Potential corruption mechanisms:
├─ Bit flips from cosmic rays
├─ Heat-induced bit errors
├─ Manufacturing defects
├─ Vibrations and shocks
├─ Power anomalies (voltage spikes)
├─ Firmware bugs
└─ Media degradation over time

Silent data corruption:
- Data written is different from what read
- OS doesn't detect failure (no clear error signal)
- Dangerous: User doesn't know data corrupted

Examples:
- File block bit flip: User reads garbage, doesn't realize
- Metadata (inode) corruption: File system crash or wrong file returned
- Directory corruption: Wrong file accessed or file deleted unexpectedly
```

### Detection: Checksums

Add redundancy to detect corruption:

#### Checksum Algorithm

Simple XOR-based checksum:

```c
uint32_t checksum(uint8_t *data, int size) {
    uint32_t sum = 0;
    for (int i = 0; i < size; i++)
        sum ^= data[i];  // XOR each byte
    return sum;
}
```

Storage on disk:

```c
struct data_block {
    uint8_t data[4096];         // User data
    uint32_t checksum;          // 4 bytes
};
```

**Use**:

```c
// Write:
block.checksum = checksum(block.data, sizeof(block.data));
write_to_disk(&block);

// Read:
read_from_disk(&block);
uint32_t expected = checksum(block.data, sizeof(block.data));
if (expected != block.checksum) {
    // CORRUPTION DETECTED!
    // Can retry/recover, or return error
}
```

#### CRC (Cyclic Redundancy Check)

More powerful than XOR:

```
CRC detects:
- Single bit errors: YES
- Burst errors (multiple adjacent bits): YES
- Multiple random errors: Usually YES

Common polynomial (CRC-32):
G(x) = x^32 + x^26 + x^23 + x^22 + x^16 + x^12 + x^11 + x^10
       + x^8 + x^7 + x^5 + x^4 + x^2 + x + 1
```

**Trade-off**:

- CRC more reliable than XOR
- CRC more expensive to compute
- For most purposes, CRC-32 used (4 bytes, fast)

### Recovery: Redundancy

If corruption detected, recovery mechanisms:

#### Replication

Store multiple copies:

```
File block: [D0, D1, D2, D3]

Without replication:
Disk 1: [D0, D1, D2, D3]

With replication (RAID-1):
Disk 1: [D0, D1, D2, D3]
Disk 2: [D0, D1, D2, D3]  (mirror)

If Disk 1 block corrupted:
1. Checksum fails
2. Read Disk 2 mirror
3. Use mirror data
```

**Cost**: 2× space (100% overhead)

#### Parity

Store checksum across multiple blocks (like RAID-4/5):

```
File blocks: [D0, D1, D2]

Parity block: P = D0 ⊕ D1 ⊕ D2

If D0 corrupted:
D0 = D1 ⊕ D2 ⊕ P  (recover using XOR)

Advantage: Only (N+1)/N overhead for N blocks
Example: 4 blocks + 1 parity = 20% overhead (vs. 100% for replication)
```

**Trade-off**:

- Replication: Fast recovery, high cost
- Parity: Slower recovery, lower cost

### Example: Whole-Disk Checksums

Modern SSDs incorporate checksums at multiple levels:

```
Level 1: Block-level checksum
┌─────────────┐
│   Metadata  │ ← Includes checksum of block
│ checksum    │
├─────────────┤
│   Data      │
└─────────────┘

Level 2: Request-level checksum
┌─────────────┐
│ Block from  │
│ Disk 0      │ ← Whole request validated
│ Block from  │
│ Disk 1      │
└─────────────┘

Level 3: End-to-end checksum
┌─────────────────────────────────┐
│ Data in user process memory     │ ← Checksum covers:
│                  ↓              │    - Memory data
│ Data in page cache              │    - Page cache
│                  ↓              │    - Disk blocks
│ Data on disk                    │
└─────────────────────────────────┘
```

### Silent Data Corruption Challenges

Even with checksums, challenges remain:

```
Challenge 1: Checksum itself corrupted
Solution: Store checksum far from data
         Use separate redundancy for checksum

Challenge 2: When/where to check?
Every read? Expensive.
Periodically scan? Might miss recent corruption.
Solution: Hybrid approach - check critical data, sample others

Challenge 3: Can't always recover
If redundancy also corrupted, unrecoverable.
Solution: Multiple levels of redundancy for critical data
         (RAID-6 for 2-disk failure, etc.)

Challenge 4: Detection doesn't equal recovery
Can detect corruption but data might be permanently lost.
Solution: Backup strategy + redundancy
         Regular backups to recover lost data
```

---

## Key Takeaways by Topic

### I/O System Design

- **Hierarchy**: CPU → memory bus → (PC/PCIE) → device buses
- **I/O methods**: Polling (simple, wasteful), interrupts (efficient), DMA (best for bulk data)
- **Device drivers**: Handle low-level communication, interrupt handlers, data buffering

### Disk Performance

- **Seek + rotation dominate**: Data transfer negligible (7 ms seek + 4 ms rotation vs. 0.005 ms transfer)
- **Scheduling matters**: SCAN/C-SCAN prevent starvation better than SSTF
- **Large sequential > small random**: 100× throughput difference

### RAID

- **RAID-0**: Speed (max capacity) but no fault tolerance
- **RAID-1**: Redundancy (2× mirrors) but 50% capacity loss
- **RAID-5**: Balance (N-1 capacity, 1 failure tolerance) but write penalty
- **Small-write problem**: Random write → 4 disk I/Os (2 reads for parity + 2 writes)

### File Systems

- **Inodes**: Metadata about files (size, permissions, block pointers)
- **Multi-level index**: Direct pointers (48 KB) + indirect (4 MB) + double-indirect (4 GB)
- **Path resolution**: Multiple I/Os per open (cache critical for performance)

### FFS & Locality

- **Cylinder groups**: Organize disk to keep related files nearby
- **Locality principle**: Files in same directory accessed together
- **Large files exception**: Spread blocks across groups to avoid consuming entire group

### Crash Consistency

- **Problem**: Partial updates leave system inconsistent
- **Journaling solution**: Write-ahead logging prevents inconsistency
- **Trade-off**: Journaling has overhead but guarantees consistency

### LFS (Log-Structured FS)

- **Key insight**: Make all writes sequential (high bandwidth)
- **Inode map**: Indirection layer to avoid seeking for file inodes
- **Garbage collection**: Reclaim space from overwritten blocks

### Flash SSDs

- **Asymmetry**: Read (µs) fast, erase (ms) very slow
- **Write blocks changes**: Erase entire block to write single page
- **FTL**: Manages translation, wear leveling, garbage collection
- **Wear leveling**: Distribute erases to prevent premature failure

### Data Protection

- **Checksums**: Detect corruption
- **Redundancy**: Enable recovery
- **Trade-off**: 100% overhead for replication vs. ~20% for parity

---

## Terminology Reference

| Term                    | Meaning                                                                                        |
| ----------------------- | ---------------------------------------------------------------------------------------------- |
| **Inode**               | Index node: metadata structure containing file information (size, permissions, block pointers) |
| **Inode map (imap)**    | Mapping from inode number to current disk location (used in LFS)                               |
| **Cylinder group**      | Collection of consecutive cylinders (tracks at same radius); FFS organizational unit           |
| **Direct pointer**      | Block pointer stored directly in inode (12 in ext2/ext3)                                       |
| **Indirect block**      | Block containing pointers to data blocks (1 level of indirection)                              |
| **Block group**         | Modern term for FFS cylinder group (consecutive blocks, not cylinders)                         |
| **Checksum**            | Redundancy data (XOR, CRC) used to detect corruption                                           |
| **Parity**              | XOR-based redundancy allowing recovery of lost data (RAID-4/5)                                 |
| **Segment**             | Large chunk of data written sequentially (LFS term)                                            |
| **Garbage collection**  | Process of reclaiming space from dead/overwritten blocks                                       |
| **Wear leveling**       | Distributing P/E cycles across flash blocks to prevent premature failure                       |
| **FTL**                 | Flash Translation Layer: controller managing logical-to-physical address mapping               |
| **Write amplification** | Ratio of physical bytes written to flash vs. logical bytes user wrote                          |
| **TxB/TxE**             | Transaction begin/end markers in journal                                                       |
| **Checkpoint**          | Writing metadata updates from journal to final on-disk location                                |
| **Revoke record**       | Journal entry indicating block should not be replayed during recovery                          |

---

**Study Guide Complete**

This comprehensive guide synthesizes chapters 35-45 of OSTEP, focusing on I/O, Storage, and Persistence mechanisms. Each section builds understanding from device-level operations (I/O, disk scheduling) through system-level abstraction (file systems, RAID) to advanced techniques (journaling, LFS, flash management). The emphasis is on understanding tradeoffs and seeing how each mechanism addresses fundamental challenges: performance (sequential vs. random I/O), reliability (redundancy, checksums), and consistency (journaling, copy-on-write).

# Distributed Systems & Networking: Comprehensive Study Guide

This guide synthesizes the core concepts from operating systems distributed computing, covering fundamental networking principles, reliable message passing, Remote Procedure Calls (RPC), and two pioneering distributed file system architectures: NFS and AFS.

---

## Chapter 48: Distributed Systems Foundations

### Introduction to Distributed Systems

A **distributed system** is a collection of independent computers connected by a network that appear to act as a single coherent system. The fundamental challenge in designing distributed systems is **handling failure**, which becomes inevitable when you scale beyond a single machine. In a data center with thousands of machines, failure is not an exception—it is the norm.

### 48.1 Packet Loss and Network Unreliability

#### Understanding Packet Loss

Networks are fundamentally unreliable. Packets flowing across routers and network links can be lost for multiple reasons:

1. **Router Memory Overflow**: Routers maintain queues of packets waiting to be transmitted. If too many packets arrive simultaneously, the queue fills up and the router discards incoming packets.

2. **End Host Resource Exhaustion**: When a machine receives a large number of messages directed at it, the receiving machine's resource buffer can become overwhelmed, causing packet loss even at the source or destination.

3. **Physical Link Failures**: Transient electromagnetic noise or timing issues can corrupt packets, causing the receiving hardware to discard them.

**Key Principle**: Packet loss is fundamental in networking—you must design systems assuming some percentage of messages will not arrive at their destination.

#### Checksums for Data Integrity

A **checksum** is a small summary of data that enables detection of corruption during transmission.

```
CHECKSUM IMPLEMENTATION (Simple XOR Example):
─────────────────────────────────────────────

Data Block (16 bytes in hex):
  365e c4cd ba14 8a92 ecef 2c3a 40be f666

Binary representation (grouped by 4 bytes per row):
  0011 0110 0101 1110 1100 0100 1100 1101
  1011 1010 0001 0100 1000 1010 1001 0010
  1110 1100 1110 1111 0010 1100 0011 1010
  0100 0000 1011 1110 1111 0110 0110 0110

XOR each column:
  0010 0000 0001 1011 1001 0100 0000 0011
  Result (hex): 0x201b9403
```

**Checksum Evaluation Axes**:

- **Effectiveness**: How well does the checksum detect bit flips? Stronger checksums detect more corruption patterns.
- **Performance**: How costly is computing the checksum? Fast checksums may miss subtle errors.
- **Tradeoff**: You generally cannot get both maximum effectiveness and minimum cost—there is an inherent tradeoff.

**Common Checksum Types**:

- **XOR**: Simple but has limitations (doesn't detect if two bits in same position flip in each checksummed unit)
- **Addition**: Fast but poor if data is shifted
- **Fletcher Checksum**: Detects all single-bit and double-bit errors, plus many burst errors
- **CRC (Cyclic Redundancy Check)**: Treats data as large binary number, dividing by agreed value. Excellent strength-to-speed ratio.

Process: Compute checksum → Send message + checksum → Receiver computes checksum → Compare: if match, likely uncorrupted.

### 48.2 Unreliable Communication: UDP

The User Datagram Protocol (UDP) is an example of an unreliable communication layer built into modern networking stacks (UDP/IP).

#### UDP Socket Basics

```c
// UDP Socket Setup: Create and bind to a port
int UDP_Open(int port) {
    int sd;
    // Create socket: AF_INET (IPv4), SOCK_DGRAM (UDP), 0 (default protocol)
    if ((sd = socket(AF_INET, SOCK_DGRAM, 0)) == -1)
        return -1;

    struct sockaddr_in myaddr;
    bzero(&myaddr, sizeof(myaddr));
    myaddr.sin_family = AF_INET;
    // Convert port number to network byte order
    myaddr.sin_port = htons(port);
    // Accept connections on any available interface
    myaddr.sin_addr.s_addr = INADDR_ANY;

    // Bind socket to port
    if (bind(sd, (struct sockaddr *)&myaddr, sizeof(myaddr)) == -1) {
        close(sd);
        return -1;
    }
    return sd;
}
```

**What This Does**:

- Creates a UDP socket (datagram-oriented, connectionless)
- Binds the socket to a specific port so the OS knows to route incoming packets on that port to this process
- Returns a socket descriptor (integer) for use in subsequent operations

#### Address Resolution

```c
// Resolve hostname to IP address and populate socket address struct
int UDP_FillSockAddr(struct sockaddr_in *addr,
                     char *hostname, int port) {
    bzero(addr, sizeof(struct sockaddr_in));
    addr->sin_family = AF_INET;           // IPv4
    addr->sin_port = htons(port);         // Convert to network byte order

    struct hostent *host_entry;
    if ((host_entry = gethostbyname(hostname)) == NULL)
        return -1;

    struct in_addr *in_addr = (struct in_addr *)host_entry->h_addr;
    addr->sin_addr = *in_addr;
    return 0;
}
```

**What This Does**:

- Takes a hostname (e.g., "cs.wisc.edu") and looks up its IP address via DNS
- Returns the resolved address in a format the OS can use for routing

#### Sending and Receiving Data

```c
// Send UDP datagram to remote host
int UDP_Write(int sd, struct sockaddr_in *addr,
              char *buffer, int n) {
    int addr_len = sizeof(struct sockaddr_in);
    return sendto(sd, buffer, n, 0, (struct sockaddr *)addr, addr_len);
}

// Receive UDP datagram from remote host
int UDP_Read(int sd, struct sockaddr_in *addr,
             char *buffer, int n) {
    int len = sizeof(struct sockaddr_in);
    return recvfrom(sd, buffer, n, 0, (struct sockaddr *)addr,
                    (socklen_t *)&len);
}
```

**What These Do**:

- `sendto()`: Sends data to a specific address (connectionless)
- `recvfrom()`: Receives data and returns sender's address
- Both return number of bytes sent/received, or -1 on error

#### Complete Client-Server Example

```c
// CLIENT CODE
int main(int argc, char *argv[]) {
    int sd = UDP_Open(20000);  // Listen on port 20000
    struct sockaddr_in addrSnd, addrRcv;
    int rc = UDP_FillSockAddr(&addrSnd, "cs.wisc.edu", 10000);

    char message[BUFFER_SIZE];
    sprintf(message, "hello world");
    rc = UDP_Write(sd, &addrSnd, message, BUFFER_SIZE);

    if (rc > 0)  // Message sent successfully
        int rc = UDP_Read(sd, &addrRcv, message, BUFFER_SIZE);

    return 0;
}

// SERVER CODE
int main(int argc, char *argv[]) {
    int sd = UDP_Open(10000);  // Listen on port 10000
    assert(sd > -1);

    while (1) {
        struct sockaddr_in addr;
        char message[BUFFER_SIZE];
        // Block until message arrives
        int rc = UDP_Read(sd, &addr, message, BUFFER_SIZE);

        if (rc > 0) {
            char reply[BUFFER_SIZE];
            sprintf(reply, "goodbye world");
            // Send reply back to sender
            rc = UDP_Write(sd, &addr, reply, BUFFER_SIZE);
        }
    }
    return 0;
}
```

**Kernel Behavior**:

- Client creates socket on port 20000, sends datagram to server on port 10000
- Kernel routes packet through network stack
- Server's OS receives and places packet in socket's receive buffer
- `UDP_Read()` awakens server, returns data and sender address
- Server sends reply back to client's address (20000)
- Client's `UDP_Read()` receives the reply

**Important**: UDP is unordered, unreliable, and may drop packets. Some packets may be lost in transit or at endpoints.

### 48.3 Reliable Communication: Timeout, Retry, and Sequence Numbers

To build reliability on top of unreliable UDP, we need three mechanisms:

#### Mechanism 1: Acknowledgments

```
Timeline: Basic Acknowledgment
────────────────────────────────

Sender                      Receiver
 [send message]  ──────────>
                              [receive message]
                              [process]
                 <────────── [send ack]
[receive ack]
```

**How it works**: After sending a message, the sender waits for an acknowledgment. When the receiver gets a message, it sends a short "ack" message confirming receipt.

#### Mechanism 2: Timeout and Retry

```
Timeline: Timeout/Retry Handling
────────────────────────────────

Sender                          Receiver
 [send message;
  keep copy;
  set timer]  ──────────>
              (packet lost)
  ...
  (waiting for ack)
  ...
  [timer goes off]
  [set timer/retry]
  [re-send message]  ──────────>
                                  [receive message]
                     <────────── [send ack]
  [receive ack;
   delete copy/timer off]
```

**Kernel Timeout Handler**:

- Client sends message, sets kernel timer for N milliseconds
- Timer expires if no ack received within that window
- Timer interrupt handler re-sends message from saved copy
- Successfully received ack cancels timer

#### Mechanism 3: Sequence Numbers to Prevent Duplicates

```
Problem: Ack Loss
─────────────────

Sender                          Receiver
 [send message N;
  keep copy;
  set timer]  ──────────>
                                  [receive message N]
                                  [process]
                     <────────── [send ack N]
                  (ack lost)
  [timer goes off]
  [timer/retry]
  [re-send message N]  ──────────>
                                  [receive message N (duplicate!)]
                                  [send ack N]
              <────────── [ack N]

Problem: Message got processed twice!
Solution: Receiver must know to ignore duplicate
```

**Sequence Counter Implementation**:

```
Sender and Receiver agree on starting counter value (e.g., 1)

Sending side:
─────────────
counter = 1
send(message, counter=1)  // Include counter in message
counter = 2
send(message, counter=2)
counter = 3
...

Receiving side:
───────────────
expected_counter = 1

receive(message, ID=1):
  if ID == expected_counter {
    process(message)
    expected_counter = 2
  } else {
    ack(ID) but do NOT process  // Duplicate detected
  }

receive(message, ID=2):
  if ID == expected_counter {
    process(message)
    expected_counter = 3
  }
```

**Result**: If ack for message N is lost and sender retries, receiver recognizes duplicate by sequence number and acks without re-processing.

#### TCP: The Standard Reliable Layer

TCP/IP combines all these mechanisms plus additional sophistication:

- Congestion control (adapts to network conditions)
- Multiple outstanding requests (pipelined messages)
- Graceful connection setup and teardown
- Flow control (receiver tells sender how fast to send)
- Hundreds of optimizations refined over decades

**Design Decision**: Many RPC systems use UDP + custom reliability rather than TCP to avoid redundant ack messages. TCP reliability + application retry = waste.

### 48.4 Communication Abstractions: Beyond Raw Messaging

Early distributed systems researchers explored two abstractions:

#### Distributed Shared Memory (DSM)

**Concept**: Extend virtual memory to span multiple machines, creating illusion of shared address space for remote processes.

**How it works**:

```
Process on Machine A      Memory System      Process on Machine B
─────────────────                           ──────────────────
read(page 0x5000)         [Local?]  ────────> [Fetch from B]
                          [Not local, page
                           fault to handler]
                          [Send message to B
                           requesting page]
                          <───────  [B sends ]
                                     [page data]
                          [Install in page
                           table]
[Process resumes]
```

**Why it Failed**:

1. **Failure handling**: If machine B crashes, suddenly part of address space disappears. Dereferencing a pointer into dead machine's memory becomes impossible to handle.
2. **Performance**: Programmers had to write code assuming memory access was cheap. But some "memory" accesses cross network (expensive). Had to carefully arrange computation to minimize communication, defeating the abstraction.
3. **Complexity**: Maintaining cache coherency across machines is extremely difficult.

**Verdict**: DSM was researched extensively but never achieved practical success. Modern systems do not use DSM.

### 48.5 Remote Procedure Call (RPC)

**Concept**: Make calling a function on a remote machine look like calling a local function.

#### The RPC Abstraction

Instead of manually crafting messages and handling network details, programmer writes:

```c
// Local code - looks exactly like normal function calls
int result = remote_func(arg1, arg2, arg3);
printf("Result: %d\n", result);
```

RPC system hides:

- Packing arguments into message format
- Routing to remote machine
- Unpacking at destination
- Executing actual function
- Packing results
- Routing results back
- Unpacking results at caller

#### RPC Architecture: Two Main Components

**1. Stub Generator (Protocol Compiler)**

Input specification:

```
interface {
    int func1(int arg1);
    int func2(int arg1, int arg2);
};
```

Generator produces:

- **Client stub**: For each function, code to marshal args, send message, wait for reply, unmarshal results
- **Server stub**: For each function, code to unmarshal request, call actual function, marshal results, send reply

#### Client-Side Stub Operations

For a client RPC call like `result = func1(42)`:

```
1. Create message buffer
   ┌────────────────────────┐
   │ Function ID | Args     │
   │     1       │ 42       │
   └────────────────────────┘

2. Serialize (marshal) arguments into buffer
   - Function identifier indicates which func to call
   - Arguments packed into binary format
   - Complex structures described by RPC compiler

3. Send message to RPC server via network

4. Wait for reply (synchronous by default)
   - Block until response arrives or timeout

5. Deserialize (unmarshal) response
   - Extract return code(s) from buffer
   - Handle complex return types (lists, structs, etc.)

6. Return to caller
   - Caller sees result as if function returned locally
```

#### Server-Side Stub Operations

When server receives RPC request for `func1(42)`:

```
1. Unmarshal the message
   - Extract function ID (tells us which function to call)
   - Extract arguments (42)

2. Call the actual function
   result = func1(42);

3. Marshal the results
   ┌────────────────────┐
   │ Return code        │
   │ (e.g., result=84) │
   └────────────────────┘

4. Send reply to caller
   - Network system delivers response
```

#### Complex Arguments Challenge

**Problem**: What if function takes a pointer to a buffer?

```c
int write(int fd, char *buf, int count);  // RPC problem!
```

Cannot send pointer over network—must send actual data.

**Solutions**:

- **Well-known types**: RPC compiler understands standard types (buffer_t with associated size)
- **Annotations**: Developer annotates complex structures

```
interface {
    // RPC compiler knows "buf with count bytes" should be serialized
    int write(int fd, buffer<char> buf, int count);
};
```

#### Server Concurrency: Thread Pool

Simple server (processes one request at a time):

```
while true {
    request = wait_for_request()
    handle(request)  // Single-threaded
}
```

Problem: While handling one request (especially if it blocks on I/O), server cannot accept new requests.

**Solution: Thread Pool**

```
Main Thread:
  while true {
    request = wait_for_request()
    worker_thread = get_idle_worker()
    assign(worker_thread, request)
  }

Worker Threads (pool of N threads):
  while true {
    request = wait_for_assignment()
    handle(request)
    return_to_pool()
  }
```

**Benefits**: Multiple requests handled concurrently. While one thread blocks on I/O, others process different requests.

**Tradeoff**: Increased programming complexity—workers must use locks if accessing shared state.

#### RPC Runtime Library Issues

**Problem 1: Service Discovery**

How does client know where server is?

**Solutions**:

- Hard-coded hostname + port: `rpc_connect("server.example.com", 8080)`
- Look up in registry: DNS or special naming service
- Broadcast: Send "does anyone provide service X?" message

#### Problem 2: Transport Protocol Choice

**Option A: Build RPC on TCP**

- Pros: TCP handles all reliability
- Cons: Double acknowledgment overhead
  - TCP acks the request
  - Application acks the reply
  - Two extra round-trips

**Option B: Build RPC on UDP**

- Pros: Single acknowledgment layer, faster
- Cons: Must implement reliability in RPC layer (timeout/retry)
- Most RPC systems choose this path

#### Endianness Handling

Different CPU architectures store multi-byte integers differently:

```
Integer value: 0x12345678

Big Endian (network order):    Little Endian (x86):
┌──────────────────┐           ┌──────────────────┐
│ 12 34 56 78      │           │ 78 56 34 12      │
└──────────────────┘           └──────────────────┘
(Most significant byte first)   (Least significant byte first)
```

**Solution**: RPC layer standardizes on one endianness (network order = big endian). Convert if sender/receiver differ.

```c
// XDR (eXternal Data Representation) - Sun's standard endianness layer
send_message {
    byte_order = htonl(value);  // Convert to network order
}
receive_message {
    value = ntohl(byte_order);  // Convert from network order
}
```

#### Asynchronous RPC

Standard RPC is synchronous (caller blocks):

```c
result = remote_func(arg);  // Block until response arrives
```

**Asynchronous RPC** allows pipelined requests:

```c
id1 = async_remote_func(arg1);  // Send, return immediately
id2 = async_remote_func(arg2);  // Send another
...
result1 = wait_for(id1);        // Block on first result
result2 = wait_for(id2);        // Block on second result
```

**Benefit**: Network used more efficiently—multiple requests in flight simultaneously.

#### Long-Running Remote Calls

**Problem**: What if remote function takes a long time?

```
Sender sets timeout = 5 seconds

Receiver processing:
  [do work]
  [do more work]
  [still processing...]
  [10 seconds elapsed]

Sender's timer fires before response!
```

**Solution**: Explicit intermediate acknowledgment

```c
// Sender receives ack that request was received
// Then explicitly polls: "Are you still working on my request?"
// If receiver keeps saying "yes", sender keeps waiting
```

#### Large Message Fragmentation

Network protocols limit packet size. If RPC message exceeds limit:

**Network layer fragmentation** (automatic):

- Send layer splits large message into packets
- Receive layer reassembles into one message
- Transparent to RPC layer

**Application layer fragmentation** (manual):

- If network layer doesn't support it, RPC library must
- Send data in chunks, reconstruct at receiver

### 48.6 The End-to-End Argument

**Core Idea**: Certain guarantees can only be enforced at application level, not lower layers.

#### Example: Reliable File Transfer

**Naive approach**: Build reliable communication (guaranteed delivery with no corruption) in network layer. Assume this means file transfer is reliable.

**Reality**: File transfer can still fail!

```
Machine A (Sender)          Network          Machine B (Receiver)
──────────────────                          ───────────────────
[File in memory]
[Network corrupts] ───────> [Perfectly      [Received data
 while sending               delivered to    is received by
 (memory error)              Machine B]      kernel]
                                             [Kernel writes
                                              to disk]
                                             [Disk corrupts
                                              the write]
```

Even though network layer guaranteed reliable delivery, file arrived corrupt! Could be:

- Sender memory corruption before send
- Sender memory corruption during send
- Receiver memory corruption
- Disk write corruption on receiver

**Lesson**: To ensure reliable file transfer AND get bytes written correctly to disk, you MUST check end-to-end:

```c
// Sender side
compute_checksum_of_file(file);
send_file_and_checksum(file, checksum);

// Receiver side
receive_file();
receive_checksum();
computed = compute_checksum_of_received_file();
if (computed == received_checksum) {
    // File transfer was truly reliable end-to-end
} else {
    // Corruption detected somewhere
}
```

**Corollary**: Lower-layer reliability mechanisms can improve performance/reduce redundant checking, but cannot replace end-to-end verification.

---

## Chapter 49: Sun's Network File System (NFS)

### 49.1 Introduction to Distributed File Systems

A **distributed file system** allows multiple client machines to access files stored on centralized server(s) across a network. Unlike local file systems (files on same machine), the client-server architecture introduces network latency and new failure modes.

#### Architecture

```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Client 0   │  │  Client 1   │  │  Client 2   │  │  Client 3   │
│ (app + fs)  │  │ (app + fs)  │  │ (app + fs)  │  │ (app + fs)  │
└──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘
       │                │                │                │
       └────────────────┼────────────────┼────────────────┘
                        │ NETWORK
                 ┌──────▼──────┐
                 │   SERVER    │
                 │   (NFS)     │
                 ├─────────────┤
                 │   Storage   │
                 │   (RAID)    │
                 │   Disks     │
                 └─────────────┘
```

#### Benefits vs. Local File Systems

| Aspect         | Local FS                    | Distributed FS                |
| :------------- | :-------------------------- | :---------------------------- |
| Data Sharing   | Limited to local machine    | Any client sees same files    |
| Administration | Must manage each machine    | Centralized on server(s)      |
| Backup         | Backup each machine         | Backup one server             |
| Availability   | Files gone if machine fails | Files available if servers up |
| Performance    | Fast (direct disk)          | Slower (network latency)      |

#### Client-Server Components

**Client Side**:

- Regular applications make system calls (`open()`, `read()`, `write()`, `close()`)
- Client-side file system translates to network protocol messages
- Manages client-side caching for performance

**Server Side**:

- File server receives protocol messages
- Accesses disk storage
- Returns data/metadata in response

### 49.2 NFS Protocol Design Philosophy

**Goal**: Simple and fast server crash recovery.

**Rationale**: One server supporting many clients. If server is down, all clients suffer. Must minimize recovery time.

**Key insight**: The stateless protocol design is revolutionary.

### 49.3 Stateless vs. Stateful Protocols

#### Stateful Protocol: Problems

In stateful systems, server tracks per-client information (file descriptors, file pointers, open files, locks, etc.).

**Example Stateful Approach**:

```c
// Client code
int fd = open("foo", O_RDONLY);  // Open returns descriptor
read(fd, buffer, MAX);            // Pass descriptor, server knows file
read(fd, buffer, MAX);            // Same descriptor
read(fd, buffer, MAX);
close(fd);                        // Tell server we're done
```

**Server tracks**: fd → file path mapping, current pointer position, etc.

**Problem 1: Server Crash Recovery**

```
Timeline of Server Crash
─────────────────────────

[Server opens "foo" for Client1, returns fd=10]
[Client1 reads successfully]
[Server crashes] <─── CLIENT HAS NO WAY TO KNOW
[Client1 tries second read: "read from fd=10"]
[Server restarts] <── But fd=10 is gone!
                      Server has no record what fd=10 refers to
                      File pointer info is lost
```

**Recovery requires**: Complex handshake protocol where client tells server everything about pending requests. Defeats the purpose of crash recovery.

**Problem 2: Client Crash**

```
[Client opens file: fd=15, server allocates resources for fd=15]
[Client crashes]
[Server still thinks fd=15 is open]
[Server never receives close() because client is dead]
[Resource leak on server—fd=15 never cleaned up]
```

Server must detect client crash and clean up. How? Timeout? How long?

#### Stateless Protocol: NFS Solution

**Key principle**: Each request contains ALL information needed to complete it. Server does not maintain state.

**Example Stateless Approach**:

Instead of file descriptors, use **file handles**:

```
File Handle = (Volume ID, Inode Number, Generation Number)
```

This uniquely identifies a file without server tracking anything about the client.

```c
// Client-side logic (Venus)
open("/foo", O_RDONLY) {
    get file handle for /foo = (vol=1, ino=1234, gen=5)
    allocate local file descriptor to this handle
    store handle in open file table at index 3
    return file descriptor 3 to app
}

read(fd=3, buf, MAX) {
    lookup fd=3 in table → get file handle = (1, 1234, 5)
    get current file pointer = 0 (stored on client)
    send NFS_READ(handle=(1,1234,5), offset=0, count=MAX)
}

// Server-side (Vice)
receive NFS_READ(handle=(1,1234,5), offset=0, count=MAX) {
    // Extract what to do from handle alone
    vol = 1
    inode = 1234
    get inode from disk/cache
    read offset 0, count MAX bytes
    return data
    // Done! No state kept about this client's read.
}
```

**Recovery advantages**:

1. **Server crashes**: Just restart. Next client request contains all needed info. No recovery protocol needed.
2. **Network loses request**: Client timer fires, simply retry with same handle. Server does exact same operation—idempotent!
3. **Client crashes**: Server doesn't care. If client reboots, it'll eventually timeout and retry. Server unaware.

### 49.4 File Handles in NFS

A file handle uniquely identifies a file so client can reference it without server tracking any state.

```
File Handle Structure:
┌────────────────────────────────────────────┐
│ Volume ID (which filesystem on server)     │ 16 bits
├────────────────────────────────────────────┤
│ Inode Number (which file within volume)    │ 32 bits
├────────────────────────────────────────────┤
│ Generation Number (inode reuse counter)    │ 32 bits
└────────────────────────────────────────────┘
Total: 80 bits worth of information
```

#### Why Each Component?

**Volume ID**: NFS servers can export multiple file systems. Volume ID tells server which one.

**Inode Number**: Identifies file within that volume. Combined with inode in-memory structure, server can quickly locate data.

**Generation Number**: When inode is deleted, its number might get reused for a new file later. If client has stale handle with (vol=1, ino=500, gen=3) but server deleted that file and reused ino=500 for new file with gen=4, the generation counter prevents this confusion.

### 49.5 NFSv2 Protocol Operations

| Protocol Op | Arguments                            | Returns                                      | Purpose                 |
| :---------- | :----------------------------------- | :------------------------------------------- | :---------------------- |
| GETATTR     | File handle                          | File metadata (size, mode, timestamps, etc.) | Get file attributes     |
| SETATTR     | File handle + attributes             | —                                            | Set file attributes     |
| LOOKUP      | Directory FH + filename              | File FH + attributes                         | Find file in directory  |
| READ        | File FH + offset + count             | File data + attributes                       | Read file data          |
| WRITE       | File FH + offset + count + data      | Attributes                                   | Write file data         |
| CREATE      | Directory FH + filename + attributes | File FH + attributes                         | Create file             |
| REMOVE      | Directory FH + filename              | —                                            | Delete file             |
| MKDIR       | Directory FH + dirname + attributes  | Directory FH + attributes                    | Create directory        |
| RMDIR       | Directory FH + dirname               | —                                            | Delete directory        |
| READDIR     | Directory FH + count + cookie        | Directory entries + new cookie               | List directory contents |

#### Example: Reading a File

```
Application Code:
─────────────────
fd = open("/foo.txt", O_RDONLY);
read(fd, buffer, 1024);
close(fd);

Client-Side File System Translation:
─────────────────────────────────────

open("/foo.txt", O_RDONLY):
  1. Send NFS_LOOKUP(root_FH, "foo.txt")
     [Recursive: first LOOKUP for root → get root FH]
  2. Receive foo.txt's FH + attributes
  3. Store in local open file table:
     fd=3 → (file_handle=..., offset=0)
  4. Return fd=3 to app

read(fd=3, buf, 1024):
  1. Lookup fd=3 → get file handle at offset 0
  2. Send NFS_READ(FH, offset=0, count=1024)
  3. Receive data (1024 bytes) + attributes
  4. Update offset: now 1024
  5. Copy data to user buffer
  6. Return 1024 to app

close(fd=3):
  1. Clean up local file table entry
  2. No network message needed!
     (Stateless: server doesn't care)
```

### 49.6 Handling Server Failure: Idempotency

#### The Problem

When client doesn't receive response, what happened?

```
Possibility 1: Request Lost
Client ──────[lost]──────> Server
       (client timeout, retry)

Possibility 2: Server Down
Client ──────[arrives]──────> Server (crashed)
       (client timeout, retry)
       [server comes back]

Possibility 3: Reply Lost
Client ──────[request]──────> Server
       [received & processed]
       <────[response lost]────
       (client timeout, retry)
```

In all three cases, client retries. **The same request might be received multiple times!**

#### Idempotency: The Solution

**Idempotent operation**: Effect of doing it N times = effect of doing it once.

Example idempotent operations:

```
store(address, value);  // Idempotent: x=5, x=5, x=5 → x=5
read(address);          // Idempotent: read is state-preserving
```

Example NOT idempotent:

```
increment(counter);     // Not idempotent: counter++, counter++ produces different result
                        // than just counter++
```

#### NFS Idempotent Operations

**LOOKUP**: Read-only → idempotent ✓

**READ**: Read-only → idempotent ✓

**WRITE**: Seemingly problematic, but actually idempotent!

```c
// WRITE message contains absolute information:
WRITE(file_handle, offset=512, count=512, data=[...])

// First attempt:
[server receives, writes bytes at offset 512 to end at 1024]
[server sends reply]
[reply lost]

// Retry (from client perspective, same operation):
[server receives SAME message]
[writes bytes at offset 512 to end at 1024]  [same bytes, same location!]
[server sends reply]
[client receives]

// Result: file has correct data written at correct location
```

**Why READ/WRITE idempotent**: WRITE specifies exact bytes at exact offset. Even replayed, same outcome. READ just fetches data—no side effects.

#### Non-Idempotent Operations: mkdir

```c
MKDIR(dir_handle, "newfolder")

// First attempt:
[server creates "newfolder"]
[sends success]
[reply lost]

// Retry:
[server receives same MKDIR]
[tries to create "newfolder" again]
[ERROR: folder exists!]
[sends error]

// Problem: First attempt succeeded, retry "failed"
// But from client perspective, it looks like operation failed
```

**NFS's pragmatic approach**: Accept that mkdir and similar operations aren't perfectly idempotent. In practice, usually works out because:

- Replies rarely lost
- Retries usually succeed with "already exists" error
- Applications typically handle this

**Voltaire's Law**: Perfect is the enemy of good. Don't demand 100% correctness if practical solution handles 99.99%.

### 49.7 Client-Side Caching

Goal: Reduce network traffic and improve performance.

#### Basic Idea

```
First read from /foo:
   Network request → Server reads disk → Expensive

Subsequent reads from /foo:
   Local cache has copy → No network needed → Fast
```

#### Write Buffering: Decoupling Latency from Performance

```c
// Application writes data
write(fd, buffer, 1024);   // Returns immediately
// Data goes to client buffer (not yet to server)
write(fd, buffer, 1024);   // Returns immediately
write(fd, buffer, 1024);   // Returns immediately
close(fd);                 // Forces buffered data to server
```

**Benefit**: Application's `write()` call returns immediately. Server writes happen asynchronously. Decouples app latency from actual disk performance.

### 49.8 The Cache Consistency Problem

#### Problem Scenario

```
Three clients (C1, C2, C3), one file F

Timeline:
─────────
C1 reads F → caches F[version 1] locally
C2 overwrites F with new version → F[version 2] on server
C3 reads F → ???
```

**Question**: What does C3 see?

**Problem 1: Update Visibility**

C2 buffers write in client memory before pushing to server. Meanwhile:

- C3 reads → gets stale F[v1] from server
- Then C2 finally flushes to server
- C3 never sees F[v2]!

```
C2 local: F[v2]
Server:   F[v1]  <── C3 reads this!
```

**Problem 2: Stale Cache**

```
C1 local: F[v1]  (old)
C2 local: F[v2]  (new)
Server:   F[v2]  (new)

If C1 reads F, it uses stale cached copy F[v1]
C2 sees latest, C1 sees old
```

#### Solution 1: Flush-on-Close (Close-to-Open Consistency)

**Rule**: When client closes a file it wrote to, force all buffered writes to server.

```c
fd = open("F", O_WRONLY);
write(fd, "new data", 8);  // Buffered
write(fd, "more data", 9); // Buffered
close(fd);                 // FORCE TO SERVER
// At this point, server has latest version

// Another client (or process):
fd2 = open("F", O_RDONLY);
read(fd2, buf, ...) // Gets latest version because writes were flushed
```

**Guarantee**: When file is opened, you see latest version that was closed.

#### Solution 2: Attribute Cache to Reduce GETATTR Traffic

**Problem**: With basic consistency checking, client sends GETATTR before every read to check if file changed.

```
read(fd) {
    GETATTR(server, "is F changed since I cached it?")
    [server: "I don't see any changes"]
    Use cached copy
}
read(fd) {
    GETATTR(server, "is F changed?")
    [millions of clients hammering server with GETATTRs!]
    "No changes"
}
```

**Solution**: Cache attributes for N seconds (e.g., 3 seconds).

```
Client attribute cache:

F: {
    attributes: { size: 1024, mtime: 3:45:22 }
    cached_at: 3:45:20
    timeout: 3 seconds
}

read(fd) at 3:45:21:
    Check if (current_time - cached_at) < timeout
    Yes: still valid, no GETATTR needed
    Use cached attributes/data

read(fd) at 3:45:24:
    Check timeout: (3:45:24 - 3:45:20) > 3 seconds
    No: expired, send GETATTR
    Get fresh attributes
    Update cached_at time
```

**Tradeoff**: For 3 seconds, changes from other clients might not be visible. But massively reduces GETATTR load on server.

### 49.9 Server-Side Write Buffering

**Critical constraint**: NFS servers MUST NOT return success on WRITE until data is on stable storage (disk).

**Why?**

```
Bad scenario: Server returns success before flushing to disk
───────────────────────────────────────────────────────

Client writes 3 blocks A, B, C:
    write(fd, A_buffer, 512);  // [Server: success]
    write(fd, B_buffer, 512);  // [Server: buffers in memory, success]
    write(fd, C_buffer, 512);  // [Server: success]

File on server:  [A][B][C]
Client happy: all succeeded

Server crashes before B forced to disk!

Recovery:
    Server restarts
    File on server: [A][?old_data?][C]  <── Corrupted!
    Client thinks it succeeded, but got corrupted mix

Solution: Server MUST fsync before returning success.
    Every WRITE waits for server to force data to disk
    Slower, but correct
```

**Performance impact**: Every WRITE stalls waiting for disk. This is a massive bottleneck for file servers that must prioritize correctness.

**Industry response**: Companies like NetApp built entire businesses around fast NFS servers. Their tricks:

1. **Battery-backed write cache**: Writes go to RAM backed by battery. Data protected even if power lost.
2. **Log-structured layout**: Optimize disk writes for sequential access, much faster than random.

### 49.10 Summary: Key NFS Design Insights

| Design Choice       | Rationale                                     | Tradeoff                          |
| :------------------ | :-------------------------------------------- | :-------------------------------- |
| Stateless Protocol  | Fast recovery—no complex handshake on restart | Longer protocol messages          |
| File Handles        | Server doesn't track client state             | Hard to revoke access             |
| Idempotent Ops      | Can safely retry without duplication concerns | Some ops not naturally idempotent |
| Client Caching      | Improves performance, reduces network traffic | Consistency complexity            |
| Flush-on-Close      | Ensures visible updates across clients        | Temporary inconsistency possible  |
| Sync Writes to Disk | Prevents corruption if server crashes         | Slower performance                |

#### Key Takeaways

- **Statelessness** is the foundation of crash recovery
- **Idempotency** enables replay without fear of duplication
- **Caching** required for performance but adds inconsistency challenges
- **Simplicity and pragmatism** beat perfection; handle 99% of cases well

#### Important Terms

| Term                     | Meaning & Context                                                                                        |
| :----------------------- | :------------------------------------------------------------------------------------------------------- |
| **File Handle**          | Unique identifier (volume ID + inode + generation) that references file without server maintaining state |
| **Stateless Protocol**   | Server doesn't remember anything about client; each request is self-contained                            |
| **Idempotent Operation** | Operation can be executed multiple times with same result as executing once                              |
| **Flush-on-Close**       | Force buffered writes to stable storage when file is closed                                              |
| **Attribute Cache**      | Client-side cache of file metadata (size, permissions) with timeout to reduce server queries             |
| **Stable Storage**       | Persistent media (disk) where data survives crashes, as opposed to volatile memory                       |

---

## Chapter 50: The Andrew File System (AFS)

### 50.1 Design Goal: Scalability

**Research Question**: How many clients can one server support?

**NFS baseline**: ~20-30 clients per server (limited by CPU and network load).

**AFS Goal**: Design file system that scales much further.

**Key insight**: Protocol design fundamentally limits scalability. NFS forces too many client-server interactions.

### 50.2 AFSv1: Whole-File Caching

#### Design Principle: Whole-File Caching on Local Disk

Instead of caching individual blocks in memory (like NFS), AFS caches entire files on client's local disk.

```
Open-Read-Write-Close Cycle:
───────────────────────────

open(file):
  [Fetch entire file from server]
  [Store on local disk in cache]
  [Set up callback with server]

read():
  [Access local cached copy] ← NO NETWORK

write():
  [Access local cached copy] ← NO NETWORK

close():
  [If modified, send entire file back to server]
```

#### File Operations

```c
// Client code (Venus)
open("file.txt", O_RDONLY) {
    send Fetch(pathname="/home/remzi/file.txt") to server
}

// Server code (Vice)
receive Fetch("/home/remzi/file.txt") {
    traverse_path("/home/remzi/file.txt")
    read file from disk
    send entire file_content to client
}

// Client receives
receive file_content {
    write to local disk at cache location
    return file descriptor to app
}
```

#### Protocol Operations

| Operation   | Arguments               | Action                           |
| :---------- | :---------------------- | :------------------------------- |
| TestAuth    | File, stat info         | Check if cached file still valid |
| GetFileStat | File                    | Get metadata                     |
| Fetch       | Pathname                | Fetch entire file                |
| Store       | Pathname, file contents | Store entire file                |
| SetFileStat | File, attributes        | Set metadata                     |
| ListDir     | Directory               | List contents                    |

#### Problems Identified

AFS designers measured the prototype and found two critical bottlenecks:

**Problem 1: Pathname Traversal CPU Overhead**

```
Client requests: Fetch("/home/remzi/notes.txt")

Server traversal:
  1. Find "/" in root
  2. Find "home" in /
  3. Find "remzi" in /home
  4. Find "notes.txt" in /home/remzi

With many clients, CPU spent traversing path hierarchy.
Server becomes CPU-bound.
```

**Problem 2: TestAuth Flood**

```
Every read hit:
  1. TestAuth("/home/remzi/notes.txt")
     "Has this file changed on server?"
  2. Server: "No, use your copy"

But each TestAuth is a network round-trip!
Millions of TestAuth messages → server CPU and network bandwidth saturated.

Worse: most of the time, NOBODY else accessed the file.
```

**Scalability Result**: Only ~20 clients per server. Not an improvement over NFS.

### 50.3 AFSv2: Callbacks and File Identifiers

To scale better, redesign protocol to minimize server interactions.

#### Key Innovation 1: Callbacks

**Concept**: Server promises to notify client when a cached file changes.

**Polling (Old Way - AFSv1)**:

```
Client: "Has file X changed?"
Server: "No, same as before"
[5 seconds later]
Client: "Has file X changed?"
Server: "No, still same"
[Pattern repeats constantly]
```

**Interrupts (New Way - AFSv2)**:

```
Client: "I'm caching file X, please tell me if anyone changes it"
Server: [Establishes callback on file X]

[Other client modifies file X]
Server: [Sends message to original client]
        "Your cache for file X is invalid. Throw it away."
Client: [Invalidates file X from cache]
```

When file changes, server proactively notifies client (like interrupt vs. polling).

#### Key Innovation 2: File Identifiers (FID)

Instead of pathnames, use file IDs to reduce pathname traversal work.

```
File Identifier (FID):
┌────────────────────────┐
│ Volume ID: 1           │  Which filesystem
├────────────────────────┤
│ File ID: 4567          │  Which file in volume
├────────────────────────┤
│ Uniquifier: 3          │  For inode reuse
└────────────────────────┘
```

**Benefit**: Client can traverse pathname locally in its cache!

```
Path: /home/remzi/notes.txt

AFSv1:
  Client: Fetch("/home/remzi/notes.txt")
  Server: [traverses entire path in kernel]

AFSv2:
  Client: [locally cached: "/" → FID(1,1,1)]
          Fetch(FID(1,1,1), "home")  ← just look up "home" in root
  Server: [looks up "home" in /, returns FID(1,2,1)]

  Client: [locally cached: "home" → FID(1,2,1)]
          Fetch(FID(1,2,1), "remzi")  ← just look up "remzi" in home
  Server: [looks up "remzi", returns FID(1,3,1)]

  Client: [locally cached: "remzi" → FID(1,3,1)]
          Fetch(FID(1,3,1), "notes.txt")
  Server: [looks up file, returns FID and content]

  Client: [locally cached directory contents for all three]
```

**Result**: With directory caching + callbacks, subsequent accesses are entirely local!

#### Complete File Access Timeline

```
First access to /home/remzi/notes.txt:

Client (Venus)                      Server (Vice)           Cache State
──────────────────                  ──────────────          ──────────────

open("/home/remzi/notes.txt"):

Send Fetch(FID(root), "home")
                                    look "home" in root
                                    establish callback(C1) on "home"
                            ←────── return FID(home), content
Receive, cache "home" dir
record callback valid

Send Fetch(FID(home), "remzi")
                                    look "remzi" in home
                                    establish callback(C1) on "remzi"
                            ←────── return FID(remzi), content
Receive, cache "remzi" dir
record callback valid

Send Fetch(FID(remzi), "notes.txt")
                                    look for "notes.txt"
                                    establish callback(C1) on "notes.txt"
                            ←────── return FID(file), content
Receive, cache entire file          Cache: home, remzi
record callback valid               directories, notes.txt
                                    file

local open(notes.txt)
return fd

read(fd):
  access local cached copy
  return data

close(fd):
  check if modified
  if modified: send Store() to server

---

Second access to /home/remzi/notes.txt:

Client (Venus)                      Server (Vice)           Cache State
──────────────────                  ──────────────          ──────────────

open("/home/remzi/notes.txt"):

Check callback(home) valid? YES
  → use cached "home" dir locally

Check callback(remzi) valid? YES
  → use cached "remzi" dir locally

Check callback(notes.txt) valid? YES
  → use cached file locally

local open(notes.txt)
return fd

No network communication needed!
```

### 50.4 Cache Consistency in AFSv2

AFS consistency model is simpler and more intuitive than NFS.

#### Between-Machine Consistency

**Guarantees**: When you close a file after writing to it, the server updates all callbacks for that file across all clients.

```
Timeline:
─────────

Client1    Client2    Server          Cache
──────     ──────     ──────          ─────

open(F)
write(X)
          open(F)  [gets old version]
close(F)  ────────────→ [Server invalidates callbacks]
          ─── Receives callback break ──→
          [Cache for F marked invalid]
                       write(X) → disk

          open(F)
          ─── Fetch(F) ────→ [Gets new version X]
                    ←─── return X ───
          read() → X ✓
```

**Pattern**: Last closer wins. Whichever client closes last, its version is what persists.

#### Within-Machine Consistency

Exception: Processes on same machine see writes immediately (like local filesystem).

```
Process1 (on Client1)        Process2 (on Client1)
──────────────────          ──────────────────────

open(F, write):
  Fetch F, cache locally

write("hello")
  ↓ to local cache

                            open(F, read):
                              [Same local cache]
                            read() → "hello" ✓
                              Sees write immediately

close(F)
  [Flush to server, break callbacks]
```

### 50.5 Crash Recovery in AFSv2

More complex than NFS because of server state (callbacks).

#### Client Crash Recovery

```
Timeline:
─────────
Client1 opens file F, establishes callback
║
[Client1 crashes and reboots]
║
[Meanwhile: Server sends callback break for F, but Client1 is offline]
║
Client1 (after reboot):
  All local cache suspect (might have missed callback breaks during crash)
  Before using any cached file, send TestAuth(F)
  "Is my cached copy valid?"
  Server: "No, get fresh version"
  Client: Fetch new version, reestablish callback
```

#### Server Crash Recovery

```
Timeline:
─────────
[Server crashes]
  All callbacks lost (in-memory only, not persistent)
[Server comes back]
[Server has no idea which clients cache what]

Client accesses file:
  [Expects callback to be valid]
  [But server lost all callback state]
  [Corruption possible: client uses stale cache]
  [Server modified file, client only sees old version]

Solution: Server must tell all clients "I crashed, don't trust your cache"
  - Broadcast "server-crash" message to all clients
  - Or clients detect via heartbeat timeout and invalidate cache
  - Or client re-learns callback state on next access (TestAuth)
```

### 50.6 Performance Comparison: AFS vs. NFS

```
Workload Analysis
═════════════════

Assumptions:
- Ns = blocks in small file
- Nm = blocks in medium file
- NL = blocks in large file
- Lnet = network access time
- Lmem = memory access time
- Ldisk = disk access time
- Hierarchy: Lnet >> Ldisk >> Lmem

Workload 1: Small file, sequential read
─────────────────────────────────────
NFS:  Ns · Lnet  (fetch Ns blocks from network)
AFS:  Ns · Lnet  (fetch whole file once)
Ratio: 1 (same)

Workload 2: Small file, sequential re-read
──────────────────────────────────────────
NFS:  Ns · Lmem  (hit memory cache)
AFS:  Ns · Lmem  (hit memory cache)
Ratio: 1 (same)

Workload 6: Large file, sequential re-read
──────────────────────────────────────────
NFS:  NL · Lnet  (re-fetch all blocks from network, doesn't fit in memory)
AFS:  (NL · Ldisk) / Lnet  (read from local cached copy on disk)
Ratio: Lnet/Ldisk  (AFS much faster, maybe 100x if network is slow)

Workload 10: Large file, sequential overwrite
──────────────────────────────────────────
NFS:  NL · Lnet  (write blocks)
AFS:  2·NL · Lnet  (fetch whole file, then write back whole file)
Ratio: 2 (AFS slower)

Workload 11: Large file, single small write
──────────────────────────────────────────
NFS:  Lnet * 2  (read small block, write small block)
AFS:  2·NL · Lnet  (fetch entire large file, write entire large file)
Ratio: 2·NL  (AFS MUCH slower)
```

#### Key Observations

1. **AFS better for**: Files accessed sequentially, re-read case, repeated accesses to same file
2. **AFS worse for**: Random access within large files, small writes to large files
3. **Depends on workload**: Which type of file access dominates in your environment?

### 50.7 AFS Additional Features

Beyond protocol design, AFS added features that NFS lacked:

| Feature          | NFS                                 | AFS                             | Benefit                     |
| :--------------- | :---------------------------------- | :------------------------------ | :-------------------------- |
| Global Namespace | No (each client mounts differently) | Yes                             | Same path on all clients    |
| Security         | Basic (easily spoofed)              | Advanced (Kerberos integration) | Prevent unauthorized access |
| Access Control   | Standard UNIX permissions           | Access Control Lists            | Fine-grained sharing        |
| Administration   | Per-machine                         | Centralized                     | Easier management           |

### 50.8 Summary: NFS vs. AFS

| Aspect             | NFS                                | AFS                                    |
| :----------------- | :--------------------------------- | :------------------------------------- |
| **Caching**        | Block-based in memory              | Whole-file on disk                     |
| **Server State**   | Stateless                          | Callbacks (maintains who caching what) |
| **Consistency**    | Complex (attribute cache timeouts) | Simple (close-to-open)                 |
| **Protocol**       | Simple                             | More complex                           |
| **Scalability**    | 20-30 clients/server               | 50+ clients/server                     |
| **Crash Recovery** | Simple (stateless)                 | Complex (callbacks lost)               |
| **Performance**    | Good for random access             | Good for sequential access             |

#### Key Takeaways

- **Callbacks** transform polling into interrupt-driven model
- **Whole-file caching** amortizes fetch cost across multiple accesses
- **File identifiers** reduce pathname traversal overhead
- **Workload matters**: Protocol should match common access patterns
- **Measured approach** (Patterson's Law): Measure prototype, identify bottlenecks, fix them

#### Important Terms

| Term                          | Meaning & Context                                                                             |
| :---------------------------- | :-------------------------------------------------------------------------------------------- |
| **Callback**                  | Server promise to notify client when cached data changes. Enables interrupt model vs. polling |
| **Whole-File Caching**        | Entire file cached on local disk when opened, rather than individual blocks in memory         |
| **File Identifier (FID)**     | Tuple (volume ID, file ID, uniquifier) that uniquely references file without server state     |
| **Close-to-Open Consistency** | Updates visible when file is closed and then opened by another process                        |
| **Last Writer Wins**          | In concurrent writes from different clients, version from last close() persists on server     |
| **Workload**                  | Typical access patterns (sequential? random? mixed?) that a system must handle efficiently    |

---

## Chapter 51: Dialogue Summary on Distributed Systems

This chapter is primarily narrative dialogue between professor and student. Key lessons recap:

- **Everything can fail** in distributed systems
- **Redundancy** helps hide failures
- **Protocols** (exact bit formats) fundamentally affect scalability and failure recovery
- **Retry** is a powerful and commonly used technique
- **Careful protocol design** enables handling failure elegantly

---

## Synthesis: Key Principles Across All Chapters

### Principle 1: The End-to-End Argument

Lower layers provide mechanisms, but applications must verify correctness end-to-end.

Example network checksum vs. application-level file transfer checksum—need both.

### Principle 2: Statelessness Enables Simplicity

NFS's stateless design recovers from server crash without complex handshake. Trade-off: longer messages containing all needed info.

### Principle 3: Idempotency Enables Replay

NFS WRITE operations are idempotent (same operation N times = same result). Enables simple retry mechanism for handling failure.

### Principle 4: Protocol Design Affects Scalability

NFS: 20-30 clients limited by TestAuth flood.
AFS: 50+ clients by adding callbacks and file IDs and reducing TestAuth.

The protocol determines how many clients a server scales to.

### Principle 5: Caching Improves Performance but Adds Complexity

Client-side caching reduces network traffic but introduces cache consistency challenges (stale cache, update visibility). Must design protocol carefully (flush-on-close, callbacks, etc.).

### Principle 6: Measurement Drives Design

AFS designers measured bottlenecks, identified pathname traversal and TestAuth flood as problems, and fixed them proactively.

---

## Reference: Protocol Comparison Table

| Protocol         | Transport                | Failures Handled                                  | Consistency Model         | Scalability     | Crash Recovery           |
| :--------------- | :----------------------- | :------------------------------------------------ | :------------------------ | :-------------- | :----------------------- |
| **UDP (raw)**    | Unreliable datagrams     | None                                              | N/A                       | N/A             | N/A                      |
| **TCP**          | Reliable stream          | Packet loss, order                                | In-order delivery         | Medium          | Automatic                |
| **RPC over UDP** | UDP + reliability layer  | Packet loss, timeout/retry                        | At-most-once semantics    | Depends on app  | Custom                   |
| **RPC over TCP** | TCP (redundant)          | All                                               | Exactly-once (with dedup) | Limited by acks | Automatic                |
| **NFSv2**        | UDP + stateless protocol | Server crash, packet loss, client crash           | Weak (eventual)           | 20-30 clients   | Trivial (stateless)      |
| **AFSv2**        | UDP + callback-based     | Server crash (complex), packet loss, client crash | Strong (close-to-open)    | 50+ clients     | Complex (callbacks gone) |

---

## Glossary of Key Terms

| Term                            | Definition                                                                |
| :------------------------------ | :------------------------------------------------------------------------ |
| **Acknowledgment (ACK)**        | Short message from receiver confirming message receipt                    |
| **Callback**                    | Server promise to notify client when cached data becomes invalid          |
| **Checksum**                    | Mathematical summary of data enabling corruption detection                |
| **Close-to-Open Consistency**   | File consistency model: updates visible after close-open sequence         |
| **Distributed System**          | Collection of independent machines appearing as single system             |
| **Endianness**                  | Byte order: big-endian (network norm) vs little-endian (x86)              |
| **File Handle**                 | Server-independent identifier (volume+inode+generation) for files         |
| **Flush-on-Close**              | Write buffering strategy: force all writes to disk when closing           |
| **Idempotent**                  | Operation with same effect whether executed once or N times               |
| **Inode**                       | File system structure storing metadata and location info                  |
| **Latency**                     | Time delay for operation completion (network travel time, disk I/O, etc.) |
| **Marshaling**                  | Packing data into network message format                                  |
| **Packet Loss**                 | Network packets dropped during transmission (normal, expected)            |
| **Protocol**                    | Formal specification of message formats and exchange rules                |
| **Remote Procedure Call (RPC)** | Calling function on remote machine as if local                            |
| **Reliable Communication**      | Guaranteed delivery despite packet loss (requires timeout/retry)          |
| **Retry**                       | Re-sending request after timeout, assuming original was lost              |
| **Sequence Number**             | Counter in message preventing duplicate detection and replay              |
| **Stateless Protocol**          | Server maintains no per-client state; each request self-contained         |
| **Stub**                        | Generated code marshaling/unmarshaling RPC arguments                      |
| **Timeout**                     | Wait period before assuming message loss occurred                         |
| **Transparent Access**          | User doesn't see difference between local and remote file access          |
| **Whole-File Caching**          | Caching entire file on client disk (vs. block-based in memory)            |
