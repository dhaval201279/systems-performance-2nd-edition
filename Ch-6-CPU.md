# Terminology
For reference, CPU-related terminology used in this chapter includes the following:
1. **Processor**: The physical chip that plugs into a socket on the system or processor board and contains one or more CPUs implemented as cores or hardware threads.
2. **Core**: An independent CPU instance on a multicore processor. The use of cores is a way to scale processors, called chip-level multiprocessing (CMP).
3. **Hardware thread**: A CPU architecture that supports executing multiple threads in parallel on a single core (including Intel’s Hyper-Threading Technology), where each thread is an independent CPU instance. This scaling approach is called simultaneous multithreading (SMT).
4. **CPU instruction**: A single CPU operation, from its instruction set. There are instructions for arithmetic operations, memory I/O, and control logic.
5. **Logical CPU**: Also called a virtual processor (also known as virtual CPU), an operating system CPU instance (a schedulable CPU entity). This may be implemented by the processor as a hardware thread (in which case it may also be called a virtual core), a core, or a single-core processor.
6. **Scheduler**: The kernel subsystem that assigns threads to run on CPUs. 
7. **Run queue**: A queue of runnable threads that are waiting to be serviced by CPUs. Modern kernels may use some other data structure (e.g., a red-black tree) to store runnable threads, but we still often use the term run queue

## CPU Utilization
High CPU utilization may not necessarily be a problem, but rather a sign that the system is doing work. Some people also consider this a return of investment (ROI) indicator: a highly utilized system is considered to have good ROI, whereas an idle system is considered wasted. Unlike with other resource types (disks), performance does not degrade steeply under high utilization, as the kernel supports priorities, preemption, and time sharing. These together allow the kernel to understand what has higher priority, and to ensure that it runs first.

CPU may be highly utilized because it is often stalled waiting for memory I/O, not just executing instructions, as described in the previous section. This is the case for the Netflix cloud, where the CPU utilization is mostly memory stall cycles

CPU utilization is often split into separate kernel- and user-time metrics.


### User time and kernel time
The CPU time spent executing user-level software is called user time, and kernel-level software is kernel time. Kernel time includes time during system calls, kernel threads, and interrupts. When measured across the entire system, the user time/kernel time ratio indicates the type of workload performed.

Applications that are computation-intensive may spend almost all their time executing user-level code and have a user/kernel ratio approaching 99/1. Examples include image processing, machine learning, genomics, and data analysis.

Applications that are I/O-intensive have a high rate of system calls, which execute kernel code to perform the I/O. For example, a web server performing network I/O may have a user/kernel ratio of around 70/30.

These numbers are dependent on many factors and are included to express the kinds of ratios expected.

### Saturation
A CPU at 100% utilization is saturated, and threads will encounter scheduler latency as they wait to run on-CPU, decreasing overall performance. This latency is the time spent waiting on the CPU run queue or other structure used to manage threads.

Another form of CPU saturation involves CPU resource controls, as may be imposed in a multi-tenant cloud computing environment. While the CPU may not be 100% utilized, the imposed limit has been reached, and threads that are runnable must wait their turn.

## Tools Method
For CPUs, the tools method can involve checking the following (Linux):
1. **uptime/top**: Check the load averages to see if load is increasing or decreasing over time. Bear this in mind when using the following tools, as load may be changing during your analysis.
2. **vmstat**: Run vmstat(1) with a one-second interval and check the system-wide CPU utilization (“us” + “sy”). Utilization approaching 100% increases the likelihood of scheduler latency.
3. mpstat: Examine statistics per-CPU and check for individual hot (busy) CPUs, identifying a possible thread scalability problem.
4. **top**: See which processes and users are the top CPU consumers.
5. **pidstat**: Break down the top CPU consumers into user- and system-time.
6. **perf/profile**: Profile CPU usage stack traces for both user- or kernel-time, to identify why the CPUs are in use.
7. **perf**: Measure IPC as an indicator of cycle-based inefficiencies.
8. **showboost/turboboost**: Check the current CPU clock rates, in case they are unusually low.
9. dmesg: Check for CPU temperature stall messages (“cpu clock throttled”).

//// to be continued from 6.5.2 i.e. USE Method

# Observability 
| Tool | Description |
| :---    | :---     |
| *perf* | CPU profiling, CPU flame graphs, syscall tracing |

# References from book reading google group
1. [Unlocking Kafka's Potential: Tackling Tail Latency with eBPF](https://blog.allegro.tech/2024/03/kafka-performance-analysis.html)