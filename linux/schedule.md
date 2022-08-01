# linux调度

Linux中就绪态和运行态对应的都是TASK_RUNNING标志位，就绪态表示进程正处在队列中，尚未被调度；运行态则表示进程正在cpu上运行。

## Linux调度器
内核默认提供了5个调度器，使用struct sched_class来对调度器进行抽象:

### Stop调度器
stop_sched_class 优先级最高的调度类，可以抢占其他所有进程，不能被其他进程抢占。

### Deadline调度器
dl_sched_class 使用红黑树，把进程按照绝对截止期限进程排序，选择最小进程进行调度运行

### RT调度器
rt_sched_class 实时调度器，为每个优先级维护一个队列

### CFS调度器
cfs_sched_class 完全公平调度器，采用完全公平调度算法，引入虚拟运行时间概念

### IDLE-Task调度器
idle_sched_class 空闲调度器，每个CPU都会有一个idle线程，当没有其他进程可以调度时，调度运行idel线程

## 调度策略
### SCHED_NORMAL
普通进程调度策略，使task选择CFS调度器来调度运行

### SCHED_BATCH
普通进程调度策略，批量处理，提供吞吐量，使task选择CFS调度器来调度运行

### SCHED_IDLE
普通进程调度策略，使task以最低优先级选择CFS调度器来调度运行

### SCHED_DEADLINE
限期进程调度策略，使task选择Deadline调度器来调度运行

### SCHED_FIFO
一个先入先出的实时过程。当调度器将cpu分配给进程时，它会将进程描述符保留在运行队列列表中的当前位置。如果没有其他更高优先级的实时进程可以运行，那么该进程将继续使用CPU，即使其他具有相同优先级的实时进程也可以运行。

SCHED_FIO是一个简单的没有时间片的先进先出的调度算法。一个可运行的SCHED_FIFO任务总是调度在任何SCHED_NORMAL任务之上。当SCHED_FIFO任务变为可运行时，它会继续运行，直到它阻塞或显式让出处理器；SCHED_FIFO没有时间片，可以无限期运行。只有更高优先级的SCHED_FIFO或SCHED_RR任务可以抢占SCHED_FIFO任务。具有相同优先级的两个或多个SCHED_FIFO任务只有在它们明确选择让出处理器时才可以循环运行。如果SCHED_FIFO任务是可运行的，则所以优先级较低的任务在完成之前都无法运行。

### SCHED_RR
一个实时的时间片轮转过程。当调度器将cpu分配给进程时，它会将进程描述符放在运行队列表的末尾。此策略可确保将cpu时间公平地分配给所有具有相同优先级的SCHED_RR实时进程。

SCHED_RR与SCHED_FIFO相同，只是每个进程只能运行到它用完预定的时间片。SCHED_RR是一种实时循环调度算法。当SCHED_RR任务耗尽其时间片时，任何其他处于相同优先级的实时进程都将被调度轮询。时间片仅用于允许重新安排相同优先级的进程。与SCHED_FIFO一样，高优先级的进程总是立即抢占低优先级的进程，而低优先级的进程永远不能抢占SCHED_RR任务，即使它的时间片已用尽。

只有当发生一下事件之一时，实时进程才会被另一个进程取代:
* 该进程被具有更高实时优先级的另一个进程抢占
* 该进程执行阻塞操作，并将其置于休眠状态
* 进程停止(状态为TASK_STOPPED或TASK_TRACED)，或被终止(状态为EXIT_ZOMBIE或EXIT_DEAD)
* 该进程通过调用sched_yield()系统调用来自愿放弃cpu
* SCHED_RR已经耗尽了它的时间片

