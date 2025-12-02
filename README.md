```markdown
# xv6 Scheduling Policies Extension

This patch for the MIT xv6 operating system introduces **5 different CPU scheduling policies**, allowing users to experiment with and compare various algorithms in a teaching OS environment. Currently, **2 policies are fully implemented**: FCFS and LOTTERY. The framework is prepared for the remaining policies: PRIORITY and SML.

[![xv6-riscv](https://img.shields.io/badge/xv6-riscv64-blue.svg?style=flat-square&logo=c)](https://pdos.csail.mit.edu/6.828/2024/xv6.html)
[![Language: C](https://img.shields.io/badge/Language-C-orange.svg?style=flat-square)](https://en.cppreference.com/w/c)
[![License: MIT](https://img.shields.io/badge/License-MIT-brightgreen.svg?style=flat-square)](LICENSE)
[![MIT 6.S081](https://img.shields.io/badge/MIT-6.S081%20Lab-success.svg?style=flat-square)](https://pdos.csail.mit.edu/6.828/2024/schedule.html)

## Overview

This extension enhances xv6's scheduler to support multiple policies via a compile-time flag. If no flag is specified, the **DEFAULT (Round-Robin)** policy is used. To enable a specific policy:

```bash
$ make qemu SCHEDFLAG=FCFS
```

Alternatively, to patch an existing xv6 system:

```bash
$ ./generate.sh --lab lab_scheduling --flags FCFS
```

This patch also includes new system calls for performance analysis and dynamic control, such as `wait2` for detailed process timing metrics and `chtickets` for lottery ticket adjustments.

## Policies

Navigate to detailed descriptions:
- [DEFAULT – Round-Robin](#default---round-robin)
- [FCFS – First Come First Served](#fcfs---first-come-first-served)
- [PRIORITY – Priority Scheduling Algorithm](#priority---priority-scheduling-algorithm)
- [SML – Static Multilevel Queue Scheduling](#sml---static-multilevel-queue-scheduling)
- [LOTTERY – Lottery Probabilistic Scheduling Algorithm](#lottery---lottery-probabilistic-scheduling-algorithm)

## DEFAULT - Round-Robin

The default algorithm implemented in xv6 is one of the simplest (along with FCFS) and relies on the Round-Robin policy. It loops through all processes available to run (marked with the `RUNNABLE` state) and gives access to the CPU to each one of them one at a time.

To schedule processes fairly, a round-robin scheduler generally employs time-sharing, giving each job a time slot or quantum (its allowance of CPU time), and interrupting the job if it is not completed by then. The job is resumed next time a time slot is assigned to that process. If the process terminates or changes its state to waiting during its attributed time quantum, the scheduler selects the first process in the ready queue to execute.

In the absence of time-sharing, or if the quanta were large relative to the sizes of the jobs, a process that produced large jobs would be favored over other processes. Round-robin scheduling is simple, easy to implement, and starvation-free.

To enable it and see how DEFAULT works, use this command when compiling xv6:

```bash
$ make qemu SCHEDFLAG=DEFAULT
```

```bash
$ ./generate.sh --lab lab_scheduling --flags DEFAULT
```

## FCFS - First Come First Served

First come first served (FCFS) is the simplest scheduling algorithm. FCFS simply queues processes in the order that they arrive in the ready queue.

The scheduling overhead due to using this policy is minimal since context switches only occur upon process termination, and no reorganization of the process queue is required. Throughput can be low, because long processes can be holding the CPU, making short processes wait for a long time, so short processes in the queue are penalized over the longer ones (known as the convoy effect).

By using this policy, we have no starvation, because each process gets a chance to be executed after a definite time. Turnaround time, waiting time, and response time depend on the order of their arrival and can be high for the same reasons above.

There isn't prioritization, so using this policy we cannot force certain processes to be completed first, which means that this system has trouble meeting process deadlines. The lack of prioritization means that as long as every process eventually completes, there is no starvation. In an environment where some processes might not complete, there can be starvation since the processes that come next to the one which might not complete are never executed.

To enable it and see how FCFS works, use this command when compiling xv6:

```bash
$ make qemu SCHEDFLAG=FCFS
```

```bash
$ ./generate.sh --lab lab_scheduling --flags FCFS
```

## PRIORITY - Priority Scheduling Algorithm

Priority algorithm based on priorities values.

*(Framework prepared; full implementation pending.)*

## SML - Static Multilevel Queue Scheduling

**S**tatic **M**ulti­**L**evel queue scheduling.

*(Framework prepared; full implementation pending.)*

## LOTTERY - Lottery Probabilistic Scheduling Algorithm

The lottery is a probabilistic scheduling algorithm where each process is assigned some number of lottery tickets, and the scheduler draws a random ticket to select the next process to run.

The distribution of tickets need not be uniform; granting a process more tickets provides it a relative higher chance of selection. This technique can be used to approximate other scheduling algorithms, such as Shortest Job Next and Fair-share scheduling.

Lottery scheduling solves the problem of starvation. Giving each process at least one lottery ticket guarantees that it has a non-zero probability of being selected at each scheduling operation.

A straightforward way to implement a centralized lottery scheduler is to randomly select a winning ticket, and then search a list of clients to locate the client holding that ticket. This requires a random number generation and O(n) operations to traverse a client list of length n, accumulating a running ticket sum until it reaches the winning value.

The following system call will change the tickets of the process with a specific pid:

```c
int chtickets(int pid, int tickets)
```

In this case, `tickets` is a number between 1 and 100 which represents the new process' tickets.

Example:  
![Lottery Scheduling Algorithm Example](https://image.prntscr.com/image/aAn5ZSaWTbuTZIRISLnaFw.png)  
*Five clients compete in a list-based lottery with a total of 20 tickets. The fifteenth ticket is randomly selected, and the client list is searched for the winner. A running ticket sum is accumulated until the winning ticket value is reached. In this example, the third client is the winner.*

Additional information on the Lottery algorithm can be found here: [Lottery Paper](https://www.usenix.net/legacy/publications/library/proceedings/osdi/full_papers/waldspurger.pdf).

To enable it and see how LOTTERY works, use this command when compiling xv6:

```bash
$ make qemu SCHEDFLAG=LOTTERY
```

```bash
$ ./generate.sh --lab lab_scheduling --flags LOTTERY
```

This patch also includes a new system call, similar to `wait`, but with more functionalities, in order to check the performances of our scheduling algorithms. The new system call is called `wait2` and returns the creation time, the running time (`rutime`), the sleeping time (`stime`), and the ready time (runnable) (`retime`) for each process. This helps a lot in understanding how a scheduling policy affects the times of every process.

For example, using the PRIORITY algorithm, the sleeping time of processes with a low priority (the one which we need to run first) will be smaller than the one with high priority (the one which we want to run later).

Two other important functions which can be useful to play with are...

## Authors

* **Ali Bagheri** - *Added FCFS and lottery scheduling algorithms, edited nice and reorganization of all scheduling policies (with descriptions and optimizations)* - [aibgr](https://github.com/aibgr)

## License

This project is licensed under the MIT License.

## Acknowledgments

xv6 is inspired by John Lions's Commentary on UNIX 6th Edition (Peer to Peer Communications; ISBN: 1-57398-013-7; 1st edition (June 14, 2000)). See also http://pdos.csail.mit.edu/6.828/2014/xv6.html, which provides pointers to on-line resources for v6.

xv6 borrows code from the following sources:  
    JOS (asm.h, elf.h, mmu.h, bootasm.S, ide.c, console.c, and others)  
    Plan 9 (entryother.S, mp.h, mp.c, lapic.c)  
    FreeBSD (ioapic.c)  
    NetBSD (console.c)

The following people have made contributions:  
    Russ Cox (context switching, locking)  
    Cliff Frey (MP)  
    Xiao Yu (MP)  
    Nickolai Zeldovich  
    Austin Clements

In addition, we are grateful for the bug reports and patches contributed by Silas Boyd-Wickizer, Peter Froehlich, Shivam Handa, Anders Kaseorg, Eddie Kohler, Yandong Mao, Hitoshi Mitake, Carmi Merimovich, Joel Nider, Greg Price, Eldar Sehayek, Yongming Shen, Stephen Tu, and Zouchangwei.

The code in the files that constitute xv6 is Copyright 2006-2017 Frans Kaashoek, Robert Morris, and Russ Cox.

---

**Last Updated: December 02, 2025**  
Ready for OS courses, research, and MIT labs. Contributions welcome!
```
