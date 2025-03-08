# Book: Grokking Concurrency by Kirill Bobrov

- https://learning.oreilly.com/library/view/grokking-concurrency/9781633439771
- https://github.com/fauzanalghifary/grokking_concurrency

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

## 05 - Interprocess Communication
- The most popular types of IPC (interprocess communication) are via shared memory and message passing.
- Slogan from the Go language documentation: “Do not communicate by sharing memory; instead, share memory by communicating.”
- Channels can be thought of as pipes that are used by goroutines to communicate.
- Message queues provide a powerful means of decoupling tasks in a system, allowing producers and consumers to interact with the queue instead of directly with each other.
- Sockets are a two-way communication channel that can use networking capabilities. Here, data communication takes place through the socket interface instead of the file interface. In most cases, sockets provide the best combination of performance, scalability, and ease of use.
- As the name implies, a thread pool is made by creating a small collection of long-running worker threads at program startup and then putting them into a pool (a container). When a task needs to be executed, the pool takes one of the pre-created threads and executes it; the developer does not need to create them. Sending tasks to the thread pool is similar to adding them to the to-do list for worker threads.
- Message queues are a means of communication between threads within a pool. 
- We can implement many things with parallel hardware, but sometimes we have to use only one core, so parallel hardware is a luxury. That is not a reason to give up concurrency, because this is where concurrency beats parallelism

## 06 - Multitasking
- An application is considered bound by something when a required resource for its work is a bottleneck for achieving increased performance. 
- There are two main types of operations: CPU-bound and I/O-bound.
- Applications naturally become more and more I/O-bound
- Multitasking is the concept of performing multiple tasks over a period of time by executing them concurrently.
- By switching quickly and passing control to the tasks in the queue, the OS creates the illusion of multitasking, although only one task is executing at any given time.
- The OS abstracts from how the hardware works, and even if the system has only one core, the OS gives the developer the illusion that it doesn’t. So even if the system cannot use parallelism, developers can still use concurrent programming and take advantage of the OS’s multitasking.
- It is generally more efficient to implement multitasking by creating a single multithreaded process rather than multiple processes.
- This means the scheduler is unpredictable regarding what task will be selected for execution at any given time. Thus, the developer should never write a program based on previously seen behavior, because it is not guaranteed to happen every time. We must control the synchronization and coordination of tasks to achieve determinism in our application. 

## 07 - Decomposition
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


## 08 - Solving Concurrency Problems: Race Conditions and Synchronization
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


## 09 - Solving Concurrency Problems: Deadlocks and Starvation
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
- Starvation is one of the basic ideas behind the most famous type of attacks against online services: denial of service (DOS) attacks. In these attacks, the attacker tries to deplete all the server’s resources. The service starts to run out of available resources (storage, memory, or computing resources), crashes, and cannot provide its services.
- A possible solution to starvation is to use a scheduling algorithm with priority queuing, which also uses the aging technique. Aging is a technique of gradually increasing the priority of threads waiting in the system for a long time.
- The producer-consumer pattern is a classic concurrency pattern that describes a situation where one or more tasks produce data and put it into a shared buffer, while one or more tasks consume data from the buffer.
- It is possible to allow multiple simultaneous reads of data as long as anyone writing the data does so exclusively—that is, as long as there are no simultaneous writers.
- Any number of readers can read shared data at the same time. Only one writer can write to the shared data at a time. While a writer is writing to shared data, no reader can read it.
- In this way, we achieve an efficient solution to the problem instead of simply mutually excluding a shared resource for any operation. This is the readers-writer pattern.
- Libraries and programming languages often include a readers-writer lock (RWLock) that solves such problems. This type of lock is usually used in large operations and can greatly improve performance if the protected data structure is read often and changed only occasionally. 
- When it comes to thread safety, good design is the best protection a developer can have. Avoiding shared resources and minimizing communication between tasks makes it less likely that these tasks will mess with each other. However, it is not always possible to create an application that does not use shared resources. In that case, proper synchronization is required.
- Synchronization helps ensure that the code is correct, but it comes at the expense of performance.
- Therefore, if possible, try to design without synchronization of any type. In the case of communication, instead of shared memory, consider using message-passing IPC—in that case, you can avoid sharing memory between different tasks so each task has its own copy of the data to work with safely. You can do this with algorithmic improvements, good design models, proper data structures, or synchronization-independent classes.


## 10 - Nonblocking I/O

- The server listens for incoming connections; when a client connects, the server communicates with it until the connection is closed (close the client to close the connection). It then continues to listen for new connections
- In this implementation, the main thread contains a listening server socket that accepts incoming connections from clients. Each client connecting to the server is handled in a separate thread. The server creates another thread that communicates with the client.
- Concurrency is achieved by using multiple threads. The OS overlaps multiple threads with preemptive scheduling.
- This approach is used in many technologies, such as the popular Apache web server MPM Prefork module, servlets in Jakarta EE (< version 3), the Spring Framework (< version 5), Ruby on Rails’ Phusion Passenger, Python Flask, and many others.
- Although threads are relatively cheap to create and manage, the OS spends significant time, precious RAM space, and other resources managing them. For small tasks such as processing single requests, the overhead associated with thread management may outweigh the benefits of concurrent execution.
- Responding to thousands of connection requests simultaneously using multiple threads or processes takes up a significant amount of system resources, reducing responsiveness.
- With a high level of concurrency (say, 10,000 threads, if you can configure the OS to create that many threads), having many threads can affect throughput due to the frequent context-switching overhead. This is a scalability problem
- Let’s understand why we need threads in the first place: we need them to handle blocking operations.
- It is a synchronized task; you are “in sync” with the oven. You have to wait and be there until the moment the oven finishes with the pizza.
- For web servers, I/O is the primary task, and it turns out the server doesn’t use the CPU while it’s waiting for a response from the client. This communication is very inefficient because it is blocking.
- If a program is CPU-bound, context switching will become a performance nightmare
- If a program has a lot of I/O-bound operations, context switching is an advantage. As soon as a task goes into a Blocked state, another task in a Ready state takes its place. This allows the processor to stay busy if work (tasks in the Ready state) needs to be done
- So, if a function is blocked (for whatever reason), it can delay other tasks, and the overall progress of the entire system may suffer. If a function is blocked because it’s performing a CPU task, there’s not much we can do. But if it is blocked because of I/O, we know that the CPU is idle and can be used to perform another task that needs the CPU.
- Recalling Chapter 6, it is possible to achieve concurrency without any parallelism. This can be handy when dealing with a large number of I/O-bound tasks. We can abandon thread-based concurrency to achieve more scalability,
- The idea of nonblocking I/O is to request an I/O operation and not wait for a response so we can move on to other tasks.
- Going back to the pizza cooking analogy, this time you don’t constantly monitor the pizza. Instead, you periodically go over to the oven and “ask” it whether the pizza is ready—turn on the light in the oven, and check whether the pizza is done.
- It is a common misconception that nonblocking I/O results in faster I/O operations. Although nonblocking I/O does not block the task, it does not necessarily execute faster. Instead, it enables the application to perform other tasks while waiting for I/O operations to complete. This allows for better utilization of processing time and efficient handling of multiple connections, ultimately enhancing overall performance. Nonetheless, the speed of the I/O operation is primarily determined by hardware and network performance characteristics, and nonblocking I/O does not affect these factors.
- Using nonblocking I/O in the right situation hides latency and improves our application’s throughput and/or responsiveness. It also allows us to work with a single thread, potentially saving us from synchronization problems between threads and the costs of thread management and associated system resources.
- One simple approach to overcome the costly creation of threads or processes is to use the busy-waiting approach, where, with a single thread, we can concurrently process multiple client requests, using nonblocking operations.
- in cooperative multitasking, it’s important not to run long operations—or, if we do, to periodically return control.

## 11 - Event-based Concurrency

- Event-based concurrency involves organizing an application around events or messages rather than threads or processes. When an event occurs, the application responds by invoking a handler function, which performs the necessary processing. 
- In an event-driven program, we need to specify the code that will be run when each event occurs. This code is called a callback.
- The event loop is the heart and soul of JavaScript. In JavaScript, we are not allowed to create new threads. Instead, concurrency in JavaScript is achieved through the mechanism of event loops.
- The reactor pattern allows for an event-driven concurrency model, avoiding the overhead of creating and managing system threads, context switching, and complexities associated with shared memory and locks in traditional thread-based models. By utilizing events for concurrency, resource consumption is significantly reduced, as only one execution thread is employed
- Synchronous communication requires both parties to be ready to exchange data at the same time, creating an explicit synchronization point for both tasks
- Asynchronous communication does not require synchronization when sending and receiving, and the sender is not blocked until the receiver is ready. The calling application accesses the results asynchronously and can check for events at any convenient time. This approach allows the processor to spend time processing other tasks instead of waiting.
- In programming, asynchronous communication occurs when the caller initiates a task and does not wait for it to complete but moves on (like an inattentive partner). It does not require synchronization when sending and receiving: the sender is not blocked until the receiver is ready. If it cares about the results (or partner) provided by such a task, it must have a way to get them (by providing a callback or in any other way). Regardless of which method is used, we say that the calling application accesses the results asynchronously.
- With multiple recipients, the advantage of asynchronous messaging becomes even more apparent
- Event-based concurrency is more suitable for high-load I/O applications because it provides better scalability with higher concurrency. Such applications require less memory, even if they handle thousands of simultaneous connections. 

## 12 - Asynchronous Communication

- A sequential programming model can be extended to support concurrency by overloading synchronous calls with asynchronous semantics
- A synchronous call with asynchronous semantics added is called an asynchronous call or asynchronous procedure call (APC)
- Coroutines (user-level threads)
- we could write concurrent single-threaded code. Guess what? We can do it with coroutines!
- Coroutines are a programming construct that allows for cooperative multitasking, where a single thread of execution can be paused and resumed at specific points in the code. This approach offers several advantages, including more efficient and flexible code capable of handling asynchronous tasks without explicit threading.
- The key difference between coroutines and OS threads is that coroutine switching is cooperative rather than preemptive. This means the developer, along with the programming language and its execution environment, has control over when the switch between coroutines occurs. At the right moment, a coroutine can be paused, allowing another task to start executing instead.
- But if there is no callback method, how do you know when the burger is ready? In other words, how do you get the result of an asynchronous call?
- As the return value of an asynchronous call, we can make an object that guarantees a future result (expected result or error). This object is returned as a “promise” of a future result
- All other things being equal, we can have orders of magnitude more tasks than OS threads because the cooperative-multitasking approach uses one expensive thread to handle a large number of cheap tasks.
- JavaScript is single-threaded, so the only way to achieve multithreading is to run multiple instances of the JavaScript engine. But then how do we communicate between these instances? This is where web workers come in. They allow tasks to run in a separate thread in the background, isolated from the main thread of the web application. This multithreading capability is provided by the browser container, so not all browsers support web workers yet. Node.js is another container for the JavaScript engine, which provides multithreading with the OS.
- the asynchronous model works best in the following situations: You have a large number of tasks, the application spends most of its time doing I/O rather than processing, and tasks are largely independent, so there’s no need for intertask communication
- Asynchronous communication is a software development method that enables a single process to continue running without being blocked by time-consuming tasks such as I/O operations or network requests. Instead of waiting for a task to finish before moving on to the next one, an asynchronous program can execute other code while the task is being performed in the background
- A coroutine is a function that is partially executed and paused and, under appropriate conditions, resumed at some point in the future until its execution is complete

## 13 - Writing Concurrent Applications

- Concurrent programming means splitting a program into tasks and running them in any order with the same result
- Now that we have a way to safely run tasks concurrently, we also need a way to coordinate them with shared resources
- We have a way to safely coordinate tasks, but they often need to communicate with one another. Communication between tasks can be synchronous or asynchronous
- Foster’s design methodology.
  1. Partitioning: we decompose the problem into many tasks
  2. Communication: to obtain the data needed to execute a task and coordinate task execution.
  3. Agglomeration: grouping tasks into larger tasks to reduce communication or simplify implementation while maintaining flexibility if possible.
  4. Mapping: assign tasks in order to minimize overall execution time
- A common mistake in designing concurrent systems is choosing the specific mechanisms for concurrency too early in the design process. Each mechanism has advantages and disadvantages, and the best mechanism for a particular use case is often determined by subtle compromises and concessions. The earlier a mechanism is chosen, the less information we have on which to base a choice.
- The goal of partitioning is to discover tasks that are as granular as possible. our attention is focused on recognizing opportunities for parallel execution.
-  Two types of decomposition exist: data decomposition and task decomposition 
- If the algorithm is used to process large amounts of data, we can try to divide the data into parts, each of which allows independent processing by a separate processing unit. This is data decomposition. Another approach involves dividing calculations based on their functionality. This is task decomposition.
- Task decomposition may reveal problems or opportunities for better optimization that an inexperienced programmer may miss by looking at just the data.
- the number of subtasks may greatly exceed the number of available processor cores. But that is fine at this stage; we have a follow-up stage (agglomeration) where we aggregate the computations for our specific needs.
- these tasks need to exchange data with each other. Organizing this communication efficiently can be a challenge. Even simple decomposition can have complex communication structures. We want to minimize this overhead in our program, so it is important to define it.
- The simple solution is duplicating matrix B in all the tasks. Another option is to use shared memory all the time
- In the first two stages of the design process, computation is broken down to maximize concurrency, and communication between tasks is introduced so that tasks have the data they need.
- This third step, agglomeration, revisits the decisions made in the partitioning and communication steps.
- The goal of this step is to improve performance and simplify development efforts, often by combining groups of tasks into larger tasks
- Reducing communication overhead is one way to improve performance. When two tasks exchanging data with each other are combined into one task, the data communication becomes part of one task, and that communication and overhead are removed. This is called increasing locality.
- Another way to reduce communication overhead is to group tasks that send data, and group tasks that receive data, where possible.
-  You should design your system to take advantage of more cores as they appear. Make the number of cores an input variable, and design based on it.
- The last step in Foster’s methodology is assigning each task to a processing unit.
- Scheduling becomes a factor if we’re using a distributed system or specialized hardware with many processors for large-scale tasks
- The goal of mapping the algorithm is twofold: to minimize the overall program execution time and optimize resource utilization.
- There are two basic strategies for achieving that: place tasks that can run in parallel on different processors to increase overall concurrency, or focus on placing tasks that often interact with each other on the same processor to increase locality by keeping them close to each other.
- The distributed word count problem is a classic example of a big data problem that can be solved using distributed computing
- we must tackle two main challenges: breaking down the text files into individual words and counting the number of occurrences of each word. The second task depends on completing the first, as we cannot begin counting word occurrences until the text has been divided into individual words. This situation is a prime example of task decomposition, where we can break down the problem into smaller tasks based on their functionality. In this approach, the focus is primarily on the type of task to be performed rather than on the data needed for the computation.
- The most important (and complex) aspect of the task-scheduling algorithm is the strategy used to distribute tasks among workers. Typically, the strategy chosen is a compromise between the conflicting demands of independent work (to reduce communication costs) and global knowledge of the state of computation (to improve the load balance).
- We implement the simplest idea: a central scheduler. The central scheduler sends tasks to workers, tracks progress, and returns results. It selects idle workers and assigns them either a map task or a reduce task. When all workers have finished their map task, the scheduler notifies them to start the reduce task (in our case, it is only one worker).

## Epilog

After reading these 13 chapters, you should have a solid foundation to go deeper into the field of concurrency. There is still a lot to discover there!