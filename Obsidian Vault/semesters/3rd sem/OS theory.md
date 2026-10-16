# Unit 1
# Operating System – Unit 1: Introduction to Operating System

Based on your uploaded OS Question.pdf, Unit 1 contains 4 short-answer questions, 4 medium-answer questions, and 2 long-answer questions. Below are exam-ready answers in simple English, following that question paper's structure.

OS Question.pdf

## Part A: Short-answer questions

### 1. Define an Operating System. What are its main objectives?

An Operating System (OS) is system software that manages computer hardware and software resources and provides an interface between users and the computer.

Main objectives:

- Convenience: Makes computers easy to use.
    
- Efficiency: Uses hardware resources efficiently.
    
- Resource management: Manages CPU, memory, files, and devices.
    
- Security: Protects data and system resources.
    

### 2. What are the major elements of a computer system?

The four major elements of a computer system are:

1. Hardware: Physical components such as the CPU, RAM, keyboard, and disk.
    
2. Operating System: Manages hardware and provides services to programs.
    
3. Application Programs: Software such as browsers, editors, and games.
    
4. Users: People or other systems that interact with the computer.
    

### 3. What is a system call? Give any two examples.

A system call is a mechanism through which a user program requests a service from the operating system kernel.

Examples:

- `open()` – Opens a file.
    
- `read()` – Reads data from a file or input device.
    

### 4. What are Operating System services? Mention any four services.

Operating System services are functions provided by the OS to help users and application programs operate efficiently.

Four major services are:

1. Program execution
    
2. I/O operations
    
3. File-system management
    
4. Communication between processes
    

## Part B: Medium-answer questions

### 1. Explain the elements and organization of a computer system with a suitable diagram.

A computer system consists of four main elements: hardware, the operating system, application programs, and users.

- Hardware: Provides computing resources such as the CPU, memory, and I/O devices.
    
- Operating System: Controls hardware and allocates resources.
    
- Application Programs: Use OS services to perform tasks.
    
- Users: Interact with applications to complete their work.
    

The operating system acts as an intermediary between applications and hardware.

Users

Application Programs

Operating System

Hardware (CPU, Memory, I/O Devices)

This organization allows applications to use hardware without directly controlling every hardware component.

### 2. Explain the objectives and functions of an Operating System.

The main objectives of an operating system are to make the computer easy to use, improve system efficiency, and manage resources securely.

Functions of an Operating System:

1. Process management: Creates, schedules, and terminates processes.
    
2. Memory management: Allocates and deallocates main memory.
    
3. File management: Creates, stores, reads, and deletes files.
    
4. Device management: Controls I/O devices through device drivers.
    
5. Security and protection: Controls access to system resources.
    
6. User interface: Provides a command-line interface (CLI) or graphical user interface (GUI).
    
7. Resource allocation: Distributes CPU time, memory, and devices among programs.
    
8. Error handling: Detects and handles system errors.
    

These functions ensure that the computer operates efficiently and reliably.

### 3. Explain the evolution of Operating Systems from early computer systems to modern Operating Systems.

Operating systems evolved to improve computer performance, usability, and resource utilization.

1. Serial Processing: Early computers had no operating system. Users executed programs manually, one at a time.
    
2. Batch Operating Systems: Similar jobs were collected into batches and executed automatically.
    
3. Multiprogramming Systems: Multiple programs were kept in memory to improve CPU utilization.
    
4. Time-Sharing Systems: CPU time was shared among multiple users, allowing interactive computing.
    
5. Personal Computer Systems: Operating systems introduced user-friendly interfaces for individual users.
    
6. Network and Distributed Systems: Computers communicated and shared resources over networks.
    
7. Modern Operating Systems: Systems such as Linux, Windows, Android, and macOS support multitasking, networking, security, virtualization, and modern graphical interfaces.
    

This evolution has made computers faster, more accessible, and more capable of handling complex tasks.

### 4. Explain system calls and system programs. Discuss different categories of system calls with examples.

System calls provide an interface through which application programs request services from the OS kernel.

System programs are utilities that provide a convenient environment for developing, running, and managing programs. Examples include file-management utilities, compilers, command interpreters, and text editors.

Categories of system calls:

1. Process control: Creates or terminates processes. Example: `fork()`, `exec()`.
    
2. File management: Creates, opens, reads, and writes files. Example: `open()`, `read()`, `write()`.
    
3. Device management: Requests or releases devices and performs device I/O.
    
4. Information maintenance: Obtains system information, such as process IDs and time.
    
5. Communication: Allows processes to exchange data. Example: pipes and socket operations.
    
6. Protection: Controls permissions and access to resources. Example: `chmod()` on Unix-like systems.
    

System calls provide controlled access to kernel services, while system programs make common computing tasks easier.

## Part C: Long-answer questions

### 1. Explain the structure of an Operating System in detail. Discuss OS services, user–Operating System interface, system calls, and system programs.

#### Introduction

An Operating System is system software that acts as an interface between users, application programs, and computer hardware. It manages resources and provides services required for program execution.

#### 1. Structure of an Operating System

An operating system is organized into components that work together to manage the computer.

- Kernel: The core part of the OS that manages CPU, memory, processes, and devices.
    
- Process management: Controls process creation, execution, scheduling, and termination.
    
- Memory management: Allocates memory and tracks its usage.
    
- File-system management: Organizes files and directories and controls file access.
    
- I/O management: Handles input/output operations through device drivers.
    
- Protection and security: Controls access to system resources.
    

#### 2. Operating System services

The OS provides several services:

1. Program execution: Loads and runs programs.
    
2. I/O operations: Performs input/output operations.
    
3. File-system manipulation: Creates, reads, writes, and deletes files.
    
4. Communication: Supports data exchange between processes.
    
5. Error detection: Detects hardware and software errors.
    
6. Resource allocation: Allocates CPU time, memory, and devices.
    
7. Accounting: Tracks resource usage.
    
8. Protection and security: Prevents unauthorized access.
    

#### 3. User–Operating System interface

The interface enables users to interact with the operating system.

- Command-Line Interface (CLI): Users enter commands through a terminal, such as Bash in Linux.
    
- Graphical User Interface (GUI): Users interact through windows, icons, menus, and a pointer.
    
- System-call interface: Programs request kernel services through system calls.
    

#### 4. System calls

System calls allow programs to request services from the kernel. They provide controlled access to OS functionality.

Examples include:

- `open()` – Opens a file.
    
- `read()` – Reads data.
    
- `write()` – Writes data.
    
- `fork()` – Creates a process on Unix-like systems.
    
- `exec()` – Executes a program.
    

System calls are classified into process control, file management, device management, information maintenance, communication, and protection.

#### 5. System programs

System programs are utilities that make the computer convenient to operate and develop software on.

Examples include:

- File-management utilities
    
- Command interpreters or shells
    
- Compilers and assemblers
    
- Text editors
    
- System monitoring tools
    
- Communication utilities
    

#### Conclusion

The OS structure, services, interfaces, system calls, and system programs work together to provide a reliable and convenient computing environment while managing hardware resources efficiently.

### 2. Explain different Operating System structuring methods, such as simple structure, layered approach, microkernel, and modular structure. Discuss their advantages and disadvantages.

#### Introduction

Operating System structure describes how its components are organized and communicate with one another. Different structures offer different trade-offs in performance, reliability, security, and maintainability.

#### 1. Simple structure

In a simple structure, OS components are not divided into clearly separated layers. Many components can directly interact with one another.

Example: Early MS-DOS is commonly used as an example of a simple OS structure.

Advantages:

- Simple to develop for small systems.
    
- Low communication overhead.
    
- Can provide good performance.
    

Disadvantages:

- Difficult to maintain and extend.
    
- Weak separation between components.
    
- A fault in one component can affect other components.
    

#### 2. Layered structure

The OS is divided into multiple layers. Each layer uses services from the layer below it and provides services to the layer above it.

Advantages:

- Easier debugging and testing.
    
- Clear separation of responsibilities.
    
- Easier maintenance and design.
    

Disadvantages:

- Defining suitable layers can be difficult.
    
- Requests may pass through several layers.
    
- Additional overhead can reduce performance.
    

#### 3. Microkernel structure

A microkernel keeps only essential functions, such as basic scheduling, low-level memory management, and inter-process communication, in the kernel. Other services may run in user space.

Examples: MINIX 3 and QNX.

Advantages:

- Better isolation between services.
    
- Improved fault containment.
    
- Easier to extend or replace user-space services.
    
- A smaller kernel can reduce the attack surface.
    

Disadvantages:

- Communication between components may add overhead.
    
- Design and implementation can be complex.
    
- Performance depends on the architecture and communication mechanisms.
    

#### 4. Modular structure

A modular OS uses a core kernel with separate modules that can be loaded or unloaded when needed.

Example: Linux supports loadable kernel modules, including device drivers and file-system modules.

Advantages:

- Flexible and extensible.
    
- Easier to maintain individual components.
    
- Modules can be loaded when required.
    
- Avoids placing every optional component permanently in the kernel.
    

Disadvantages:

- Faulty kernel modules can crash the system.
    
- Modules executing in kernel space have high privileges.
    
- Dependencies between modules can complicate troubleshooting.
    

#### Comparison

|Structure|Main advantage|Main disadvantage|
|---|---|---|
|Simple|Low overhead|Poor separation|
|Layered|Organized design|Layer-crossing overhead|
|Microkernel|Isolation and fault containment|Communication overhead|
|Modular|Flexibility and extensibility|Kernel-module risks|


---


# Unit 2
# Operating System – Unit 2: Process Management

These answers follow Unit II: Process Management in your uploaded OS Question.pdf, including all 4 short-answer questions, 4 medium-answer questions, and both long-answer questions.

OS Question.pdf

## Part A: Short-answer questions

### 1. Define a process. How is a process different from a program?

A process is a program currently being executed. It includes program code, data, CPU registers, memory, and execution state.

|Program|Process|
|---|---|
|Passive set of instructions|Active execution of instructions|
|Stored as a file|Has an execution state and allocated resources|
|Does not itself need an execution context|Requires an execution context|

### 2. What is CPU scheduling? State any four scheduling criteria.

CPU scheduling is the process by which the operating system selects a process from the ready queue to execute on the CPU.

Four scheduling criteria are:

1. CPU utilization: Keep the CPU busy.
    
2. Throughput: Maximize completed processes per unit time.
    
3. Turnaround time: Minimize the total time from arrival to completion.
    
4. Waiting time: Minimize time spent waiting in the ready queue.
    

### 3. What is a semaphore? How is it used for process synchronization?

A semaphore is a synchronization mechanism used to control access to shared resources by multiple processes.

It uses two atomic operations:

- `wait()` – Acquires a resource or waits until it becomes available.
    
- `signal()` – Releases a resource or notifies a waiting process.
    

A binary semaphore can help protect a critical section so that processes do not access shared data unsafely.

### 4. Define deadlock. List the four necessary conditions for deadlock.

A deadlock occurs when a group of processes waits indefinitely for resources held by one another.

The four necessary conditions are:

1. Mutual exclusion: A resource can be used by only one process at a time.
    
2. Hold and wait: A process holds one resource while waiting for another.
    
3. No preemption: Resources cannot be forcibly taken away.
    
4. Circular wait: Processes form a circular chain of resource dependencies.
    

All four conditions must exist simultaneously for a deadlock to occur.

## Part B: Medium-answer questions

### 1. Explain the process concept and process states with a suitable state-transition diagram.

A process is a program in execution. The operating system tracks each process using a Process Control Block (PCB), which stores information such as the process ID, state, CPU registers, scheduling information, and memory details.

Main process states:

- New: The process is being created.
    
- Ready: The process is waiting for CPU time.
    
- Running: The CPU is executing the process.
    
- Waiting/Blocked: The process is waiting for an event, such as I/O completion.
    
- Terminated: The process has finished execution.
    

New

Ready

Running

Terminated

Waiting

Returns to Ready after event completion

Conceptual process-state diagram. A running process can also return to Ready when preempted by the scheduler.

These states help the OS manage execution, CPU allocation, and process transitions efficiently.

### 2. Explain and compare FCFS, SJF, Priority, and Round Robin CPU scheduling algorithms.

CPU scheduling algorithms determine which ready process gets CPU time next.

1. First-Come, First-Served (FCFS)

Processes execute in order of arrival.

- Advantage: Simple and easy to implement.
    
- Disadvantage: Long processes can delay short processes.
    

2. Shortest Job First (SJF)

The process with the shortest CPU burst is selected first.

- Advantage: Gives minimum average waiting time under standard non-preemptive assumptions when burst times are known.
    
- Disadvantage: Burst-time prediction is difficult; long processes may starve.
    

3. Priority Scheduling

The process with the highest priority is selected first.

- Advantage: Important processes can receive CPU time sooner.
    
- Disadvantage: Low-priority processes may starve. Aging can reduce starvation.
    

4. Round Robin (RR)

Each process receives a fixed time slice called a time quantum. An unfinished process returns to the end of the ready queue.

- Advantage: Fair CPU sharing and good responsiveness for interactive systems.
    
- Disadvantage: Very small time quanta cause excessive context-switching overhead.
    

|Algorithm|Selection rule|Preemptive?|
|---|---|---|
|FCFS|Earliest arrival|Usually no|
|SJF|Shortest CPU burst|Usually no|
|Priority|Highest priority|Can be either|
|Round Robin|Cyclic queue with time quantum|Yes|

### 3. Explain the critical-section problem and discuss how semaphores can be used to solve it.

A critical section is the part of a program where shared data or resources are accessed or modified. If multiple processes execute this section simultaneously without proper synchronization, race conditions may occur.

The critical-section problem requires three properties:

1. Mutual exclusion: Only one process can enter the protected section at a time.
    
2. Progress: If the section is free, the choice of the next process should not be postponed indefinitely.
    
3. Bounded waiting: A process should not wait forever to enter the section after requesting access.
    

Using a binary semaphore:

```
semaphore mutex = 1;

wait(mutex);

// Critical section
// Access shared resource safely

signal(mutex);
```

The `wait()` operation acquires the semaphore before entering the critical section. The `signal()` operation releases it afterward.

Semaphore operations must be atomic. Every execution path should release the semaphore correctly; otherwise, other processes may remain blocked.

### 4. Explain deadlock prevention, avoidance, detection, and recovery methods.

A deadlock occurs when processes wait indefinitely for resources held by one another. The OS can manage deadlocks using four approaches.

1. Deadlock prevention

Prevents deadlock by ensuring at least one of the four necessary conditions cannot hold.

Examples include requesting all resources at once or imposing a fixed resource-ordering rule.

2. Deadlock avoidance

Allocates resources only when doing so keeps the system in a safe state.

Example: The Banker's Algorithm checks whether a safe sequence of process execution exists before granting a request.

3. Deadlock detection

Allows deadlocks to occur and periodically checks the system for circular resource dependencies or deadlocked processes.

4. Deadlock recovery

Once deadlock is detected, the OS can:

- Terminate one or more processes.
    
- Preempt resources where possible.
    
- Roll back processes to a safe checkpoint.
    

Prevention avoids a necessary condition, avoidance rejects unsafe allocations, detection identifies deadlocks, and recovery resolves them after detection.

## Part C: Long-answer questions

### 1. Calculate waiting time and turnaround time using FCFS and SJF scheduling. Draw the Gantt charts.

The question provides the following processes:

|Process|Arrival time|Burst time|
|---|---|---|
|P1|0|8|
|P2|1|4|
|P3|2|2|
|P4|3|5|

We will use non-preemptive SJF for the SJF calculation.

Formulas

- Turnaround Time (TAT) = Completion Time − Arrival Time
    
- Waiting Time (WT) = Turnaround Time − Burst Time
    
- Average WT = Total WT ÷ Number of Processes
    
- Average TAT = Total TAT ÷ Number of Processes
    

#### A. FCFS (First-Come, First-Served)

Processes execute in arrival order: P1 → P2 → P3 → P4.

Gantt chart:

![[Pasted image 20261010105907.png]]


|Process|CT|TAT = CT − AT|WT = TAT − BT|
|---|---|---|---|
|P1|8|8 − 0 = 8|8 − 8 = 0|
|P2|12|12 − 1 = 11|11 − 4 = 7|
|P3|14|14 − 2 = 12|12 − 2 = 10|
|P4|19|19 − 3 = 16|16 − 5 = 11|
|Total||47|28|



![[Pasted image 20261010150243.png]]

#### B. SJF (Shortest Job First)

At time 0, only P1 has arrived, so it executes first and finishes at time 8. The remaining processes are then selected by shortest burst time: P3 → P2 → P4.

Gantt chart:

![[Pasted image 20261010105302.png]]

|Process|CT|TAT = CT − AT|WT = TAT − BT|
|---|---|---|---|
|P1|8|8 − 0 = 8|8 − 8 = 0|
|P2|14|14 − 1 = 13|13 − 4 = 9|
|P3|10|10 − 2 = 8|8 − 2 = 6|
|P4|19|19 − 3 = 16|16 − 5 = 11|
|Total||45|26|


![[Pasted image 20261010105105.png]]


#### Final comparison

|Metric|FCFS|Non-preemptive SJF|
|---|---|---|
|Average waiting time|7|6.5|
|Average turnaround time|11.75|11.25|

Conclusion: For these processes, non-preemptive SJF gives lower average waiting and turnaround times than FCFS.

### 2. Explain deadlock in detail. Discuss its four necessary conditions, prevention, avoidance using the Banker's Algorithm, detection, and recovery techniques.

#### Introduction

A deadlock is a situation in which two or more processes wait indefinitely because each is waiting for a resource held by another process. As a result, none of the affected processes can proceed.

Example: Process P1 holds resource R1 and waits for R2, while process P2 holds R2 and waits for R1. Neither can continue.

#### 1. Four necessary conditions

1. Mutual exclusion: At least one resource is non-shareable.
    
2. Hold and wait: A process holds resources while requesting additional resources.
    
3. No preemption: Resources cannot be forcibly removed under the system's resource-allocation rules.
    
4. Circular wait: A circular chain of processes exists, with each process waiting for a resource held by the next.
    

All four conditions must hold simultaneously for a deadlock to occur.

#### 2. Deadlock prevention

Prevention ensures that at least one necessary condition cannot occur.

- Make resources shareable where practical.
    
- Require processes to request all needed resources before execution.
    
- Allow resource preemption where safe and appropriate.
    
- Impose a fixed ordering for resource acquisition to prevent circular wait.
    

These policies can reduce resource utilization or introduce other practical limitations.

#### 3. Deadlock avoidance: Banker's Algorithm

The Banker's Algorithm checks resource requests before granting them. It grants a request only if the resulting allocation leaves the system in a safe state.

Important data structures:

- Available: Resources currently available.
    
- Max: Maximum resources each process may need.
    
- Allocation: Resources currently assigned.
    
- Need: Remaining resources each process may require.
    

\[ \text{Need}=\text{Max}-\text{Allocation} \]

The algorithm checks whether there is a safe sequence in which every process can obtain its remaining resources, finish, and release its allocation. If such a sequence exists, the state is safe; otherwise, the request is not granted immediately.

#### 4. Deadlock detection

The operating system can allow resource requests and periodically run a detection algorithm.

For single-instance resource types, a wait-for graph can help identify cycles. For multiple instances, detection algorithms examine available resources, current allocations, and outstanding requests.

#### 5. Deadlock recovery

After detecting a deadlock, the OS may:

- Terminate all deadlocked processes.
    
- Terminate processes one at a time until the deadlock is removed.
    
- Preempt resources from selected processes, where possible.
    
- Roll back processes to an earlier safe state.
    

Recovery decisions should consider process priority, resources held, and the cost of restarting work.

#### Conclusion

Deadlock prevention stops a necessary condition, avoidance checks whether allocations remain safe, detection identifies existing deadlocks, and recovery restores progress after a deadlock has occurred.
