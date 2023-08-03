# Concurrency

**Thread**
- like a separate process
- but share the same address space and so can access the same data
- each thread has its own private set of registers
- each thread has its own program counter that tracks where the program is fetching instructions from
- when switching from one thread to another, a context switch must take place
- one stack per thread
- Two reasons to use threads
    1. parallelism
    2. avoid blocking program process due to slow I/O

**Critical section** - piece of code that accesses a shared resource

**Race condition** - multiple threads of exeuction enter the critical section at roughly the same time

**Indeterminate** - program consists of one or more race conditions

**Mutual exclusion** - guarantees that only a single thread ever enters a critical sections, thus avoiding races


### Building a lock
- mutual exclusion
- fairness - does each thread contending for the lock get a fair shot at acquiring it once it is free?
- performance - time overheads added by using the lock


#### Condition Variables
- there are many cases where a thread wishes to check whether a condition is true before continuing execution
- simple approache of just spinning until condition becomes true is grossly inefficient
- condition variable is an explicit queue that threads can put themselves on when some state of execution is not as desired
- when it changes state, it can then wake one of those waiting threads and thus allow them to continue
- wait() - call executed when a thread wishes to put itself to sleep
- signal() - call executed when a thread has changed something in the program and thus wants to wake a sleeping thread


**Bounded buffer problem**
- producers generate data items and place in buffer
- consumers grab items from buffer and process
- bounded buffer is a shared resource
- don't fully understand this. need to spend some more time


