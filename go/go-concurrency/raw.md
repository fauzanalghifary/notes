# Book: Grokking Concurrency by Kirill Bobrov

(https://learning.oreilly.com/library/view/grokking-concurrency/9781633439771)

## 01 - Introducing Concurrency
- A concurrent system is a system that can deal with many things at once.
- Concurrency is decoupling strategy!

## 02 - Serial and Parallel Execution
- The opposite of sequential programming is concurrent programming. 
- Concurrency is based on the idea that there are independent computations that can be executed in an arbitrary order with the same outcome.
- The requirement of Parallel execution: task independence and hardware support.
-  Just because you can write programs to be parallel doesn’t mean you always should, because the costs and overhead associated with parallelization sometimes outweigh the benefits.
  - LET'S GET ONE INTERESTING EXAMPLE FOR THIS ONE ABOVE.
- Concurrency is about multiple tasks that start, run, and complete in overlapping time periods in no specific order. Parallelism is about multiple tasks running at the same time on hardware with multiple computing resources
- parallelism does not imply concurrency (????)
- Rob Pike: “Concurrency is about DEALING with lots of things at once. Parallelism is about DOING lots of things at once.”


### 05 - Interprocess Communication
- The most popular types of IPC (interprocess communication) are via shared memory and message passing.
- Slogan from the Go language documentation: “Do not communicate by sharing memory; instead, share memory by communicating.”
- Channels can be thought of as pipes that are used by goroutines to communicate.
- Message queues provide a powerful means of decoupling tasks in a system, allowing producers and consumers to interact with the queue instead of directly with each other.
- Sockets are a two-way communication channel that can use networking capabilities. Here, data communication takes place through the socket interface instead of the file interface. In most cases, sockets provide the best combination of performance, scalability, and ease of use.
- As the name implies, a thread pool is made by creating a small collection of long-running worker threads at program startup and then putting them into a pool (a container). When a task needs to be executed, the pool takes one of the pre-created threads and executes it; the developer does not need to create them. Sending tasks to the thread pool is similar to adding them to the to-do list for worker threads.
- Message queues are a means of communication between threads within a pool. 
- We can implement many things with parallel hardware, but sometimes we have to use only one core, so parallel hardware is a luxury. That is not a reason to give up concurrency, because this is where concurrency beats parallelism

### 06 - Multitasking
- An application is considered bound by something when a required resource for its work is a bottleneck for achieving increased performance. 
- There are two main types of operations: CPU-bound and I/O-bound.
- Applications naturally become more and more I/O-bound
- Multitasking is the concept of performing multiple tasks over a period of time by executing them concurrently.
- By switching quickly and passing control to the tasks in the queue, the OS creates the illusion of multitasking, although only one task is executing at any given time.
- The OS abstracts from how the hardware works, and even if the system has only one core, the OS gives the developer the illusion that it doesn’t. So even if the system cannot use parallelism, developers can still use concurrent programming and take advantage of the OS’s multitasking.
- It is generally more efficient to implement multitasking by creating a single multithreaded process rather than multiple processes.
- This means the scheduler is unpredictable regarding what task will be selected for execution at any given time. Thus, the developer should never write a program based on previously seen behavior, because it is not guaranteed to happen every time. We must control the synchronization and coordination of tasks to achieve determinism in our application. 

### 07 - Decomposition
- Popular concurrency patterns for creating concurrent applications: the pipeline, map, fork/join, and map/reduce patterns
- Therefore, the first step to decompose the problem is to find the dependencies of all its constituent tasks and identify those that are independent
- One of the most popular patterns in the big data world is ETL (extract, transform, load)—a popular paradigm for collecting and processing data from various sources that implements a pipeline pattern
- Data decomposition answers the question, “How do you decompose task data into chunks that can be processed independently of each other?”. Each task performs the same set of instructions but with its own chunk of data.
- the map pattern: It is used in cases where a single operation is applied to all elements of a collection, but all individual tasks are processed autonomously and have no side effects 
- the fork/join pattern: It is used when a task is divided into subtasks that are executed concurrently and then joined back together to produce a result
- The map and reduce steps are more independent than the standard fork/join as they can scale beyond a single computer, utilizing a fleet of machines to perform a single operation on a large volume of data.
- Data decomposition and task decomposition are not mutually exclusive and can be implemented simultaneously by combining them for the same application.
- When fine granularity is used, the program is broken into many small tasks. Using fine granularity leads to more parallelism and therefore increases the system’s performance
- But creating a large number of small tasks has a downside: by increasing the number of tasks that need to communicate, we significantly increase the cost of communication
- Thus, optimal performance is achieved between the two extremes, fine-grained and coarse-grained.
- Ideally, the number of tasks into which a problem is decomposed should be at least the number of available processing resources, preferably more, to provide flexibility for the runtime system.


### 08 - Solving Concurrency Problems: Race Conditions and Synchronization
- A function or operation is thread safe if it behaves correctly when accessed from multiple tasks, regardless of how those tasks are scheduled or interleaved by the execution environment
- When we have a race condition, tasks access shared resources or common variables that can be used concurrently by other tasks, and as a result, the correctness of the program depends on the relative timing of concurrent operations.
- Errors caused by race conditions are hard to reproduce and isolate. They are a kind of heisenbug: a program error that disappears or changes its behavior when we try to investigate it.
- Synchronization is a mechanism that controls access to shared resources between multiple tasks
- The right synchronization mechanism ensures exclusivity and orderly access to a resource across tasks.
- mutual exclusion (mutex) is a synchronization technique that ensures that only one task can access a shared resource at a time. 
- The first task that acquires the mutex lock is the only one that can access the shared resource until it releases the lock. The other task must wait until the lock is released.
- Synchronization is only effective when it is used consistently by all threads in the application. If we create a mutex to restrict access to a shared resource, all threads must receive the same mutex before attempting to manipulate the resource. Failing to do so would compromise the protection provided by the mutex, leading to potential errors.
- unlike a mutex, a semaphore can allow several tasks to access the resource at the same time;
- a semaphore holds a counter that keeps track of how many times it has been acquired or released. As long as the value of the semaphore counter is positive, any task can acquire the semaphore
- In essence, a mutex can be viewed as a specialized type of semaphore called a binary semaphore. In the case of a mutex, the internal counter can have only two possible values: 0 or 1.
- Another way to solve synchronization problems is to create more powerful operations that are executed in one step and thus eliminate the possibility of undesired interrupts. Such operations exist and are called atomic.
- Atomic means no other thread can see the operation in a partially completed state.
- In atomic operation, When an interruption occurs, the operation either does not execute at all or executes to the end; there is no intermediate state.
- Synchronization is expensive. Therefore, if possible, try to design without synchronization of any kind.


### 09 - Solving Concurrency Problems: Deadlocks and Starvation
- Common concurrency problems: deadlocks, livelocks, and starvation
- popular concurrency design patterns: the producer-consumer and readers-writer patterns
- Locks (mutexes and semaphores) are very tricky to use. Incorrect use of locks can break an application when the locks acquired are not released or the locks that need to be acquired never become available
- During a deadlock, multiple tasks are waiting for resources occupied by the others, and none of them can continue execution. The program is stuck in this state forever, so it is necessary to manually terminate its execution.
- Every time a task tries to get more than one lock at a time, there is a possibility of a deadlock.
- Never assume a specific order of execution. When there are multiple threads, as we have seen, the execution order is nondeterministic. If you care about the execution order of one thread relative to another, you must apply synchronization. But for the best performance, you should avoid synchronization as much as possible. In particular, you want highly detailed tasks that do not require synchronization; this will allow your cores to work as fast as possible on each task assigned to them.
- A livelock is similar to a deadlock and occurs when two tasks are competing for the same set of resources—but in a livelock, a task gives up its first lock in an attempt to get a second lock. After getting the second lock, it comes back and tries to get the first lock again. The task is now in the same blocked state because it spends all its time releasing one lock and trying to get another instead of doing actual work.
-  It is similar to a deadlock, but the difference is that the tasks are “polite” and let others do their work first.
- Livelocks are a subset of a broader set of problems called starvation.
- Starvation is exactly what it sounds like: a thread is quite literally starved, never gaining access to required resources, and no progress is made.
- Starvation is one of the basic ideas behind the most famous type of attacks against online services: denial of service (DOS) attacks. In these attacks, the attacker tries to deplete all of the server’s resources. The service starts to run out of available resources (storage, memory, or computing resources), crashes, and cannot provide its services.
- A possible solution to starvation is to use a scheduling algorithm with priority queuing, which also uses the aging technique. Aging is a technique of gradually increasing the priority of threads waiting in the system for a long time.
- The producer-consumer pattern is a classic concurrency pattern that describes a situation where one or more tasks produce data and put it into a shared buffer, while one or more tasks consume data from the buffer.
- It is possible to allow multiple simultaneous reads of data as long as anyone writing the data does so exclusively—that is, as long as there are no simultaneous writers.
- Any number of readers can read shared data at the same time. Only one writer can write to the shared data at a time. While a writer is writing to shared data, no reader can read it.
- In this way, we achieve an efficient solution to the problem instead of simply mutually excluding a shared resource for any operation. This is the readers-writer pattern.
- Libraries and programming languages often include a readers-writer lock (RWLock) that solves such problems. This type of lock is usually used in large operations and can greatly improve performance if the protected data structure is read often and changed only occasionally. 
- When it comes to thread safety, good design is the best protection a developer can have. Avoiding shared resources and minimizing communication between tasks makes it less likely that these tasks will mess with each other. However, it is not always possible to create an application that does not use shared resources. In that case, proper synchronization is required.
- Synchronization helps ensure that the code is correct, but it comes at the expense of performance.
- Therefore, if possible, try to design without synchronization of any type. In the case of communication, instead of shared memory, consider using message-passing IPC—in that case, you can avoid sharing memory between different tasks so each task has its own copy of the data to work with safely. You can do this with algorithmic improvements, good design models, proper data structures, or synchronization-independent classes.


### 10 - Nonblocking I/O

- The server listens for incoming connections; when a client connects, the server communicates with it until the connection is closed (close the client to close the connection). It then continues to listen for new connections
- In this implementation, the main thread contains a listening server socket that accepts incoming connections from clients. Each client connecting to the server is handled in a separate thread. The server creates another thread that communicates with the client.
- Concurrency is achieved by using multiple threads. The OS overlaps multiple threads with preemptive scheduling.

## Book:

## Book:

## Talk: Concurrency is not Parallelism by Rob Pike

(https://www.youtube.com/watch?v=oV9rvDllKEg&t=2s)
- 