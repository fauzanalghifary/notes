# Book: Grokking Concurrency by Kirill Bobrov

(https://learning.oreilly.com/library/view/grokking-concurrency/9781633439771)

- A concurrent system is a system that can deal with many things at once.
- Concurrency is decoupling strategy!

- The opposite of sequential programming is concurrent programming. 
- Concurrency is based on the idea that there are independent computations that can be executed in an arbitrary order with the same outcome.
- The requirement of Parallel execution: task independence and hardware support.
-  Just because you can write programs to be parallel doesn’t mean you always should, because the costs and overhead associated with parallelization sometimes outweigh the benefits.
  - LET'S GET ONE INTERESTING EXAMPLE FOR THIS ONE ABOVE.
- Concurrency is about multiple tasks that start, run, and complete in overlapping time periods in no specific order. Parallelism is about multiple tasks running at the same time on hardware with multiple computing resources
- parallelism does not imply concurrency (????)
- Rob Pike: “Concurrency is about DEALING with lots of things at once. Parallelism is about DOING lots of things at once.”

- The most popular types of IPC (interprocess communication) are via shared memory and message passing.
- Slogan from the Go language documentation: “Do not communicate by sharing memory; instead, share memory by communicating.”
- Channels can be thought of as pipes that are used by goroutines to communicate.
- Message queues provide a powerful means of decoupling tasks in a system, allowing producers and consumers to interact with the queue instead of directly with each other.
- Sockets are a two-way communication channel that can use networking capabilities. Here, data communication takes place through the socket interface instead of the file interface. In most cases, sockets provide the best combination of performance, scalability, and ease of use.
- As the name implies, a thread pool is made by creating a small collection of long-running worker threads at program startup and then putting them into a pool (a container). When a task needs to be executed, the pool takes one of the pre-created threads and executes it; the developer does not need to create them. Sending tasks to the thread pool is similar to adding them to the to-do list for worker threads.
- Message queues are a means of communication between threads within a pool. 
- We can implement many things with parallel hardware, but sometimes we have to use only one core, so parallel hardware is a luxury. That is not a reason to give up concurrency, because this is where concurrency beats parallelism

- An application is considered bound by something when a required resource for its work is a bottleneck for achieving increased performance. 
- There are two main types of operations: CPU-bound and I/O-bound.
- Applications naturally become more and more I/O-bound
- Multitasking is the concept of performing multiple tasks over a period of time by executing them concurrently.
- By switching quickly and passing control to the tasks in the queue, the OS creates the illusion of multitasking, although only one task is executing at any given time.
- The OS abstracts from how the hardware works, and even if the system has only one core, the OS gives the developer the illusion that it doesn’t. So even if the system cannot use parallelism, developers can still use concurrent programming and take advantage of the OS’s multitasking.
- It is generally more efficient to implement multitasking by creating a single multithreaded process rather than multiple processes.
- This means the scheduler is unpredictable regarding what task will be selected for execution at any given time. Thus, the developer should never write a program based on previously seen behavior, because it is not guaranteed to happen every time. We must control the synchronization and coordination of tasks to achieve determinism in our application. 

## Book:

## Book:

## Talk: Concurrency is not Parallelism by Rob Pike

(https://www.youtube.com/watch?v=oV9rvDllKEg&t=2s)
- 