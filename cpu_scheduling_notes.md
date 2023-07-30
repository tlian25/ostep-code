# CPU Scheduling

## Basic Types
* FIFO - first in first out
* SJF - shortest job first
* STCF - shortest time-to-completion first
    - each time a new job enters the system, the STCF scheduler determins which of the remaining jobs has the least time left and schedules that one
    - optimizes for turnaround time but bad for response time
* RR - round robin
    - runs a job for a time slice and switches to the next job in the run queue
    - incurs a lot of overhead from context switching
    - optimizes for response time but bad for turnaround time


Metrics
* T_turnaround = T_completion - T_arrival
* T_repsonse = T_firstrun - T_arrival


## MLFQ - multi-level feedback queue

- would like to optimize turnaround time by running shorter jobs first
- OS doesn't generally know how long a job will run for
- would like to also make system feel responsive to interactive users and thus minimize response time
- number of distinct queues, each assigned a priority level
- at any given time, a job that is ready to run is on a single queue
- varies the priority of a job based on observed behavior
- use the history of the job to predict its future behavior




#### Basic Rules
1. If Priority(A) > Priority(B), run A
2. If Priority(A) == Priority(B), run A & B in RR
3. When job enters system, it is placed at the highest priority

#### Attempt #1 - Change Priority
4a. If a job uses up an entire time slice while running, its priority is reduced
4b. If a job gives up the CPU before the time slice is up, it stays at the same priority level 


- approximates SJF 
- if a job is doing a lot of I/O, it will relinquish CPU before its time slice is complete. In this case we don't penalize job and keep it at the same level
- problem of starvation - if too many interactive jobs in the system, they will combine to consume all CPU time
- long-running jobs will never receive any CPU time (starve)
- smart user could rewrite their program to game the scheduler by adding no-op I/O


#### Attempt #2 - Priority Boost

5. After some time period S, move all the jobs in the system to the topmost queue

- periodically boost priority of all the jobs in the system
- processes guaranteed not to starve
- if a CPU-bound job has become interactive, the scheduler treats it properly once it has received the priority boost
- if S set too high, long-running jobs could starve
- if S set too low, interactive jobs may not get a proper share of the CPU

#### Attempt #3 - Better accounting

4. Once a job uses up its time allotment at a given level (reqgardless of how many times it has given up the CPU), its priority is reduced

- scheduler should keep track of how much of a time slice a process used at a given level
- whether it uses the time slice in one long burst or many small onces does not matter
- allow for varying time-slice length across different queues



## Proportional Share

- try to gaurantee that each job obtain a certain percentage of CPU time instead of optimizing for turnaround or response time

#### Lottery Scheduling
- every so often, hold a lottery to determine which process should get to run next
- processes that should run more often should be given more chances to win the lottery
- tickets present share
- ticket transfer - a process can temporarily hand off its tickets to another process
- ticket inflation - a process can temporarily raise or lower the number of tickets it owns


**Randomness**
- can help avoid strange corner-case behaviors
- lightweight and requires little state to track alternatives
- can be quite fast if generation is fast

#### Linux Completely Fair Scheduler (CFS)

- aims to spend very little time making scheduling decisions
- goal is to fairly divide a CPU evenly among all competing processes
- using counting-based technique known as virtual runtime (vruntime)
- as each process runs, it accumulated vruntime
- pick the process with the lowest vruntime to run next
- key question is when and how often CFS context switches (fairness vs overhead)
- keeps running and runnable processes in a red-black tree (sleep processes removed)



**Metrics**
- sched_latency - how long one process should run before considering a switch
- min_granularity - never set the time slice of a process to less than this value, ensuring that not too much time is spent in scheduling overhead
- niceness - range of -20 to +19, with default 0, positive nice imply lower priority, used for weights on vruntime



## Multiprocessor Scheduling

- temporal cache locality - when a piece of data is accessed, it is likely to be accessed again
- spatial cache locality - if a program accesses a data item at an address, it is likely to access data around that address
- cache coherence - multiprocessor complications with shared memory


#### Bus Snooping
- each cache pays attention to memory updates by observing the bus that connects them to main memory
- when a CPU sees an update for a data item it holds in its cache, it will notice the change and either invalidate its copy or update it


#### Cache Affinity
- a process running on a particular CPU builds up a fair bit of state in the caches of the CPU
- the next time it runs, it is often advantageous to run it on the same CPU

#### Single-Queue Scheduling (SQMS)
- most basic approach is to simply resuse single processing framework
- put all jobs that need to be scheduled into a single queue
- lack of scalability
- will have to insert some form of locking to ensure SQMS code accesses the single queue properly
- locks greatly reduce performance as number of CPUs in the systems grows
- contention for a single lock increases time in lock overhead
- lack of cache affinity


#### Multi-Queue Scheduling (MQMS)
- each queue follows a particular scheduling discipline
- when a job enters the system it is placed on exactly one scheduling queue according to some heuristic
- scheduled essentially independently, thus avoiding the problems of information sharing and synchronization
- problem of load imbalance - one queue could be much larger than the other
- solution is migration of jobs from one queue to the other

**Work stealing**
- a queue that is low on jobs will occasionally peek at another queue to see how full it is
- if target queue is more full than source queue, the source will steal one or more jobs from the target to help balance load
- if look at other queues too often, will have high overhead and trouble scaling


#### Linux Multiprocessor Schedulers
- O(1) schedulers
    - multi-queue
    - priority based schedulers
    - changes a process's priority over time
    - scheduling those with the highest priority in order to meet various scheduling objectives

- Completely Fair scheduler (CFS)
    - multi-queue
    - deterministic proportional-share approach

- BF Scheduler (BFS)
    - single queue
    - earliest eligible virtual deadline first (EEVDF)

