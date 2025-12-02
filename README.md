# xv6 Scheduling Policies Extension

Advanced implementation of **5 CPU scheduling algorithms** in the xv6 operating system (MIT 6.S081 / 6.828).

Currently implemented & fully tested:  
- **DEFAULT** – Round-Robin (original xv6)  
- **FCFS** – First Come First Served (non-preemptive)  
- **LOTTERY** – Lottery-based Probabilistic Scheduling

Ready for implementation (structure & hooks prepared):  
- PRIORITY – Static Priority Scheduling  
- SML – Static Multi-Level Queue Scheduling

[![xv6](https://img.shields.io/badge/xv6-riscv-blue.svg)](https://pdos.csail.mit.edu/6.828/2023/xv6.html)
[![MIT License](https://img.shields.io/badge/License-MIT-brightgreen.svg)](LICENSE)
[![C](https://img.shields.io/badge/language-C-orange.svg)]()

## Features

- Clean, modular scheduler selection via compile-time flag
- New system call: `wait2(pid, &rtime, &wtime, &stime)` – returns runtime, ready time, and sleeping time for performance analysis
- New system call: `chtickets(pid, tickets)` – dynamically change lottery tickets (1–100)
- Zero overhead when using DEFAULT (Round-Robin)
- Extensive comments and debugging macros
- Convoy effect demonstration with FCFS
- Starvation-proof Lottery scheduler

## How to Run

```bash
# Choose your scheduler at build time
make qemu SCHEDFLAG=FCFS      # First Come First Served
make qemu SCHEDFLAG=LOTTERY   # Lottery Scheduling
make qemu SCHEDFLAG=DEFAULT # Original Round-Robin (default if no flag)

# Or use the helper script (recommended for labs)
./generate.sh --lab lab_scheduling --flags LOTTERY
