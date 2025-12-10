# operating-system-project
Operating System Simulator
Group Members

- Rodolfo Mendoza

- Angel Olivares

## Project Overview
This project is a comprehensive Operating System Simulator built in C++. It simulates the core lifecycle of an OS, starting from User Authentication, moving to Process Creation (with realistic CPU/IO bursts), managing execution via CPU Scheduling Algorithms, and finally simulating hardware-level Virtual Memory Translation.

Unlike simple simulators that calculate results mathematically, this project simulates the OS tick-by-tick, allowing for realistic concurrency between CPU execution and I/O waiting, as well as real-time memory address translation.

## Features
Part 1: Security & Authentication
- Login System: Secure entry point requiring valid credentials.

- Credentials: Username: admin, Password: password123

## Part 2 & 3: Process Management & Scheduling
- Multi-Burst Processes: Processes are not static; they follow a realistic lifecycle: CPU Burst → I/O Wait → CPU Burst.

- Concurrent I/O: Implements a Waiting Queue. While the CPU executes one process, other processes can simultaneously decrement their I/O timers in the background.

- #### Algorithms Implemented:

  1. FCFS (First-Come, First-Served) - Non-preemptive.

  2. SJF (Shortest Job First) - Non-preemptive, optimizes for throughput.

  3. SRTF (Shortest Remaining Time First) - Preemptive version of SJF, offering the lowest average waiting time.

### Part 4: Virtual Memory (MMU)
#### Hardware Simulation:
Simulates a Memory Management Unit (MMU) that translates 32-bit Virtual Addresses to Physical Addresses.

#### Paging Architecture:
- Page Size: 4 KB (12-bit offset).

- TLB (Translation Lookaside Buffer): A fast-cache simulation for recent translations.

- Page Table: A map-based structure simulating RAM frames.

### Design Decisions & Simulation Architecture
This section explains the technical choices behind the simulation logic ("The Why and How").

### 1. The Tick-by-Tick Architecture
#### Why: 
Many simulators just calculate Start Time + Burst = End Time. We chose a tick-based loop (incrementing time by 1 unit).

#### How:
- Inside Scheduler.cpp, the main loop represents the system clock.

- Preemption: Every tick, the scheduler checks if a new process arrived or if a shorter job exists (for SRTF), allowing immediate context switching.

- Concurrency: By simulating time units, we can execute the CPU process and update all processes in the WaitingQueue simultaneously. This accurately simulates an OS where I/O devices work in parallel with the CPU.

### 2. Integrated MMU Simulation
#### Why:
We wanted to simulate memory accesses as they happen, rather than as a separate post-calculation. 

#### How:
- The MMU object is embedded inside the Scheduler.

- During a CPU burst, the simulator randomly generates "Load/Store" instructions (random 32-bit integers).

- These virtual addresses are passed to mmu.translate(), which performs bitwise operations (>> and &) to extract the Page Number and Offset, simulating real hardware logic.

### 3. Multi-Burst Process Structure
#### Why:
Real processes don't just use the CPU once and die; they interact with I/O (disk, network).

#### How:
- The Process class uses a std::vector<int> (e.g., {5, 2, 3}) to store a sequence of bursts.

- The class acts as a state machine, tracking the current_burst_index. Even indices are treated as CPU (sent to Ready Queue), and odd indices are I/O (sent to Waiting Queue).

### 4. Data Integrity Strategy
#### Why:
We need to compare multiple algorithms (FCFS vs. SJF) on the exact same dataset to be fair.

#### How:
- The Scheduler methods accept processes by value (copies), not reference.

- This allows main.cpp to generate a random master list once, and then pass fresh copies to each algorithm. The simulation can modify states (running, terminated) without "ruining" the data for the next algorithm test.

## How to Compile & Run

### Compilation
To compile all components (Scheduler, Process logic, MMU, Auth) into a single executable:

     g++ main.cpp auth.cpp Process.cpp Scheduler.cpp MMU.cpp -o os_sim
     

### Execution
1. Run the program:

     ./os_sim


3. Login: Enter admin and password123.

4. Observation:

  - The system generates random processes.

  - It runs FCFS, SJF, and SRTF sequentially.

  - You will see [MMU] logs appear during CPU bursts, showing address translations.

  - Final statistical tables will compare the performance of each algorithm.

## Algorithms Summary

#### Algorithm	Type	Description
FCFS:	Non-Preemptive;	Processes execute in arrival order. Simple but suffers from the "Convoy Effect."

SJF:	Non-Preemptive;	Selects the process with the shortest current CPU burst. Reduces waiting time but can starve long jobs.

SRTF:	Preemptive;	Checks the queue at every time tick. If a new process arrives with a shorter remaining time, the current process is paused.


## File Structure
- main.cpp: Entry point. Generates random process datasets and triggers the scheduler.

- Process.h/cpp: Defines the PCB (Process Control Block), managing states, multi-burst vectors, and statistics.
  
- Scheduler.h/cpp: The core engine. Manages Ready/Waiting queues, executes algorithms, and integrates the MMU.
  
- MMU.h/cpp: Simulates hardware paging, including TLB lookup and Page Table mapping.
  
- auth.h/cpp: Handles user login security.

## Sample Output

##### [MMU] Process 1: Virt: 45092 -> Phys: 8196

##### [MMU] Process 2: Virt: 12288 -> Phys: 4096

##### ...

##### | PID | Arrival | Total CPU | Complete | Turnaround | Waiting |

##### |  1    |    0      |     5       |    5       |     5        |    0      |

##### ...

##### Average Waiting Time: 1.57
