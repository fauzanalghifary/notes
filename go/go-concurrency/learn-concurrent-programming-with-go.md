# Book: Learn Concurrent Programming with Go by James Cutajar

- https://learning.oreilly.com/library/view/learn-concurrent-programming/9781633438385
- https://github.com/fauzanalghifary/ConcurrentProgrammingWithGo


## 00

- concurrent programming skills are still in short supply.
- The tech industry lacks programmers skilled in concurrency because concurrent programming is perceived as extremely challenging. Many developers even dread using concurrent programming to solve problems. The perception in the tech industry is that this is an advanced topic, reserved only for hard-core computer nerds. There are many reasons for this. Developers are not familiar with the concepts and tools available for managing concurrency, and sometimes they fail to recognize how concurrency can be modeled programmatically.
- Concurrent programming adds another dimension to your programming: programs stop being a set of instructions executing one after the other. This makes it a challenging topic, and it requires you to think about programs in a different way.

## 01 - Stepping into concurrent programming

- “If the code is not written in a manner that takes advantage of the additional cores, it won’t matter if you allocate multiple processors. The code needs to use concurrent programming for it to run faster when you add more processing resources.”
- Even with a single CPU core, concurrency offers benefits because it enables time-sharing and lets us perform tasks while we’re waiting for I/O operations to complete
- Even with a single CPU core, concurrency offers benefits because it enables time-sharing and lets us perform tasks while we’re waiting for I/O operations to complete
- Concurrent programming is about writing code so that multiple tasks and processes can execute and interact at the same time.
- Concurrent programming makes our software more responsive because we don’t need to wait for one task to finish before responding to a user’s input
- Go uses a lightweight construct, called a goroutine, to model the basic unit of concurrent execution.
- goroutines give us a system of user-level threads running on a set of kernel-level threads and managed by Go’s runtime.
- we should focus mainly on writing correct concurrent programs, letting Go’s runtime and hardware mechanics deal with parallelism
- The principle is that if you need something to be done concurrently, create a goroutine to do it. If you need many things done concurrently, create as many goroutines as you need, without worrying about resource allocation. Then, depending on the hardware and environment that your program is running on, your solution will scale.
- This concurrency model of having isolated goroutines communicating and synchronizing using channels (see figure 1.5) reduces the risk of race conditions—types of programming errors that occur in bad concurrent programming and that are typically very hard to debug and lead to data corruption and unexpected behavior. This type of modeling concurrency is more akin to how concurrency happens in our everyday lives, such as when we have isolated executions (people, processes, or machines) working concurrently, communicating with each other by sending messages back and forth.
- Depending on the problem, the classic concurrency primitives used with memory sharing (such as mutexes and condition variables, found in many other languages) will sometimes do a better job and result in better performance than using CSP-style programming
- In geek speak, we say that we have hit our scalability limit. Why does this happen? Why can’t we continue to double our resources (people, money, or processors) and always reduce the time spent by half?
- parallel-to-sequential ratio. This became known as Amdahl’s law.
- Amdahl’s law states that the overall performance improvement gained by optimizing a single part of a system is limited by the fraction of time that the improved part is actually used.
- Amdahl’s law tells us that the non-parallel parts of an execution act as a bottleneck and limit the advantage of parallelizing the execution
- The article gives an alternative perspective on the limits of parallelism. Their main argument is that, in practice, the size of the problem changes when we have access to more resources.
- The second argument against Amdahl’s law is that when you increase the problem’s size, the non-parallel part of the problem typically does not grow in proportion with the problem size. In fact, Gustafson argues that for many problems, this remains constant. Thus, when you take these two points into account, the speedup can scale linearly with the available parallel resources
- Gustafson’s law tells us that as long as we find ways to keep our extra resources busy, the speedup should continue to increase and not be limited by the serial part of the problem. However, this is only true if the serial part stays constant as we increase the problem size, which, according to Gustafson, is the case in many types of programs.

## 02 - Dealing with threads

- Threads
  - In the context of concurrent programming, the operating system gives us various tools to help manage this concurrency. Two of these tools, processes and threads, represent the concurrent actors in our code.
  - A process represents a program that is currently running on the system
  - A thread is an extra construct that executes within the process context to give us a more lightweight and more efficient approach to concurrency. As we shall see, each process is started with a single thread of execution, sometimes referred to as the primary or main thread
  - Each operating system process has its own memory space, isolated from other processes. Typically, a process would work independently, having minimal interaction with other processes
  - Since processes do not share memory with each other, they tend to minimize communication with other processes.
  - When processes do need to communicate and synchronize with each other, we program them to use operating system tools and other applications, such as files, databases, pipes, sockets, etc.
  - creating too many processes takes a heavy toll on the system. For this reason, it’s quite unusual for one program to use a large number of processes concurrently, all working on the same problem.
  - Threads are the answer to some of the problems that come with using processes for concurrency. We can think of threads as the lightweight alternative to multiple processes. Creating a thread is much faster (sometimes 100 times faster), and a thread consumes fewer system resources than a process. Conceptually, threads are another execution context (kind of a microprocess) within a process.
  - multiple threads will execute concurrently sharing the same memory space. This is more efficient because we’re not consuming large amounts of memory for each execution
  - When we have more than one thread in a single process, we say that the process is multithreaded.
  - The stack space stores the local variables that live within a function. These are typically short-lived variables—when the function finishes, they are not used anymore. This space does not include variables that are shared between functions (using pointers), which are allocated on the main memory space, called the heap.
  - Since multiple threads are sharing the same memory space, we need to take care so that the threads are not stepping over each other and causing problems. We do this by using thread communication and synchronization.
  - Since threads share memory space, any change made in main memory by one thread (such as changing a global variable’s value) is visible to every other thread in the same process. This is the main advantage of using threads—multiple threads can use this shared memory to work on the same problem together. This enables us to write concurrent code that is very efficient and responsive.
  - Threads do not share stack space. Although threads share the same memory space, it’s important to realize that each thread has its own private stack space
  - Whenever we create a local non-shared variable in a function, we are placing this variable on the stack space. These local variables are thus visible only to the thread that creates them. It’s important that each thread has its own private stack space because it might call completely different functions than other threads and will need its own private space to store the variables and return values used in these functions.
  - When we have multiple threads and only one core processor, each thread in a process gets a time slice of the processor. This improves responsiveness, and it’s useful in applications where you need to respond to multiple requests concurrently (such as in a web server). If multiple processors (or processor cores) are present in a system, threads will get to execute in parallel with each other. This gives our application a speedup.
  - In Go, when the main thread of execution terminates, the entire process also terminates, even if other threads are still running. This is different than in some other languages. In Java, for instance, a process will terminate only when all the threads in the process have finished.
- Goroutines
  - Go’s answer to concurrency is the goroutine. As we shall see, it doesn’t tie in directly with an operating system thread. Instead, goroutines are managed by Go’s runtime at a higher level to give us an even more lightweight construct, consuming far fewer resources than an operating system thread.
  - We can also refer to this manner of calling functions as an asynchronous call, meaning that we don’t have to wait for the function to complete to continue executing. We can refer to a normal function call as synchronous because we need to wait for the function to return before proceeding with other instructions.
  - This is because when we run jobs concurrently, we can never guarantee the execution order of those jobs. When our main() function creates the five goroutines and submits them, the operating system might pick up the executions in a different order than we created them in.
  - It turns out that goroutines are neither OS threads nor processes. 
  - kernel-level threads: the operating system manages them.
  - user-level threads: the runtime manages them.
  - Using user-level threads is like having different threads of execution running inside the main kernel-level thread
  - From an operating system point of view, a process containing user-level threads will appear to have just one thread of execution. The OS doesn’t know anything about user-level threads. The process itself is responsible for managing, scheduling, and context switching its own user-level threads. To execute this internal context switch, there needs to be a separate runtime that maintains a table containing all the data (such as the state) of each user-level thread.
  - The main advantage of user-level threads is performance. Context-switching a user-level thread is faster than context-switching a kernel-level one
  - The downside of using user-level threads comes when they execute code that invokes blocking I/O calls. To work around this limitation, applications using user-level threads tend to use non-blocking calls to perform their I/O operations. However, using non-blocking I/O is not ideal, since not every device supports non-blocking calls.
  - Another disadvantage of user-level threads is that if we have a multiprocessor or a multicore system, we will be able to utilize only one of the processors at any point in time.
  - It is perhaps inaccurate to refer to Go’s goroutines as green threads since, as we shall see, Go’s runtime allows its goroutines to take full advantage of multiple CPUs.
  - Go provides a hybrid system that gives us the great performance of user-level threads without most of the downsides. It achieves this by using a set of kernel-level threads, each managing a queue of goroutines. Since we have more than one kernel-level thread, we can utilize more than one processor if multiple ones are available.
  - The system that Go uses for its goroutines is sometimes called the M:N threading model. This is when you have M user-level threads (goroutines) mapped to N kernel-level threads. This contrasts with normal user-level threads, which are referred to as an N:1 threading model, meaning N user-level threads to 1 kernel-level thread.
  - Go’s runtime determines how many kernel-level threads to use based on the number of logical processors. This is set in the environment variable called GOMAXPROCS. If this variable is not set, Go populates this variable by querying the operating system to determine how many CPUs your system has
  - To work around the problem of blocking calls, Go wraps any blocking operations so that it knows when a kernel-level thread is about to be descheduled. When this happens, Go creates a new kernel-level thread (or reuses an idle one from a pool) and moves the queue of goroutines to this new thread, which picks a goroutine from the queue and starts executing it. The old thread with its goroutine waiting for I/O is then descheduled by the OS. This system ensures that a goroutine making a blocking call will not block the entire local run queue of goroutines
  - This system of moving goroutines from one queue to another is known in Go as work stealing.
  - Work stealing does not just happen when a goroutine makes a blocking call. Go can also use this mechanism when there is an imbalance in the number of goroutines in the queues.
  - After a kernel-level thread has had its fair share of time on the CPU, the OS scheduler context switches the next thread from the run queue. This is known as preemptive scheduling.
  - The Go scheduler needs to execute to perform its context switching. Thus, the Go scheduler needs user-level events to trigger its execution 
  - We have no control over which goroutine the scheduler will select to execute. When we call the Go scheduler, it might pick up the other goroutine and start executing it, or it might continue the execution of the goroutine that called the scheduler.
  - As with the OS scheduler, we cannot predictably determine what the Go scheduler will execute next. As programmers writing concurrent programs, we must never write code that relies on an apparent scheduling order, because the next time we run the program, the ordering might be different
  - If we need to control the order of execution of our threads, we’ll need to add synchronization mechanisms to our code instead of relying on the scheduler
- Concurrency versus parallelism
  - We can think of concurrency as an attribute of the program code and parallelism as a property of the executing program
  - These tasks then may or may not execute in parallel. Whether they execute in parallel will depend on the hardware and environment where we execute the program.
  - For parallel execution to happen, we require multiple processing units. Otherwise, the system can interleave between the tasks, giving the impression that it is doing more than one task at the same time
  - Concurrency is about planning how to do many tasks at the same time. Parallelism is about performing many tasks at the same time.
  - In fact, we can say that parallelism is a subset of concurrency. Only concurrent programs can execute in parallel, but not all concurrent programs will execute in parallel.

## 03 - Thread communication using memory sharing

- two ways to communicate between threads: memory sharing and message passing
- Memory sharing is similar to having all our executions share a large, empty canvas (the process’s memory) on which each execution gets to write the results of its own computation
- In contrast, message passing is exactly what it sounds like. Just like people, threads can communicate by sending messages to each other.
- The type of thread communication we use in our applications will depend on the type of problem we’re trying to solve.
- Sharing memory
  - Communication via memory sharing is like trying to talk to a friend, but instead of exchanging messages, we’re using a whiteboard (or a large piece of paper), and we’re exchanging ideas, symbols, and abstractions
  - In concurrent programming using memory sharing, we allocate a part of the process’s memory—for example, a shared data structure or a variable—and we have different goroutines work concurrently on this memory
  - The mechanism for dealing with reads and writes on memory and caches in a multiprocessor system is known as the cache-coherency protocols
  - With many more processors, implementing cache coherence will become a lot more complex and costly and might eventually limit performance. This limit is known as the coherency wall.
- Memory sharing in practice
  - What happens here is that we have a very simple memory-sharing concurrent program. One goroutine updates the contents of a particular memory location, and another thread reads its contents.
  - Where should we allocate memory space for the variable count? This is a decision that the Go compiler must make for every new variable we create. It has two choices: allocate space on the function’s stack or in the main process’s memory, which we call the heap space.
  -  Go’s compiler is smart enough to realize when we are sharing memory between goroutines. When it notices this, it allocates memory on the heap instead of the stack, even though our variables might look like they are local ones belonging on the stack.
  - In technical speak, when we declare a variable that looks like it belongs to the local function’s stack but instead is allocated in the heap memory, we say that the variable has escaped to the heap. Escape analysis consists of the compiler algorithms that decide whether a variable should be allocated on the heap instead of the stack.
  - There are many instances where a variable escapes to the heap. Anytime a variable is shared outside the scope of a function’s stack frame, the variable is allocated on the heap. Sharing a variable’s reference between goroutines is one such example
  - In Go, there is an additional small cost to using memory on the heap as opposed to the stack. This is because when we are done using the memory, the heap needs to be cleaned up by Go’s garbage collection. The garbage collector goes through the objects in the heap that are no longer referenced by any goroutine and marks the space as free so that it can be reused. When we use space on the stack, this memory is reclaimed when the function finishes.
  - `go tool compile -m countdown.go`
  -  Inlining is an optimization where, under certain conditions, the compiler replaces the function call with the contents of the function. The compiler does this to improve performance since calling a function has a slight overhead, which comes from preparing the new function stack, passing the input params onto the new stack, and making the program jump to the new instructions on the function.
  - When we execute the function in parallel, it doesn’t make sense to inline the function, since the function is executed using a separate stack, potentially on another kernel-level thread.
  - By using goroutines, we are forfeiting some compiler optimizations, such as inlining, and we are increasing overhead by putting our shared variable on the heap. The tradeoff is that by executing our code concurrently, we potentially achieve a speedup.
  - race condition—when we have multiple threads (or processes) sharing a resource and they step over each other, giving us unexpected results
- Race conditions
  - Race conditions are what happens when your program is trying to do many things at the same time, and its behavior is dependent on the exact timing of independent unpredictable events.
  - This can happen because the concurrent executions are lacking proper synchronization and are stepping over each other.
  - A race condition is a good example of a Heisenbug.
  - Heisenbug is a bug that disappears or changes behavior when we attempt to debug and isolate it.
  - Since they’re very hard to debug, the best way to deal with Heisenbugs is to not have them at all. Thus, it’s vital to understand what causes race conditions and learn techniques for preventing them in our code.
  - The word atomic has its origins in the ancient Greek language, meaning “indivisible.” In computer science, when we mention an atomic operation, we mean an operation that cannot be interrupted.
  - A critical section in our code is a set of instructions that should be executed without interference from other executions affecting the state used in that section. When this interference is allowed to happen, race conditions may arise.
  - Even if the instructions were atomic, we might still run into issues. 
  - This means that there is a possibility that the two goroutines operating on separate CPUs are not seeing each other’s changes until they complete the periodic flush to memory.
  - When we are executing a badly written concurrent program in a parallel environment, it is even more likely that these types of errors will arise. Goroutines running in parallel increase the chance of these types of race conditions happening since now we will be performing some steps at the same time.
  - Never rely on telling the runtime when to yield the processor to solve race conditions. There is no guarantee that another parallel thread will not interfere.
  - How can we write concurrent programs that avoid race conditions? There is no magic bullet here. There is no single technique best used to solve every case. The first step is to make sure that we’re using the right tool for the job
  - The second step to good concurrent programming is recognizing when a race condition can occur. We must be mindful when we are sharing resources with other goroutines. Once we know where these critical code parts are, we can think about the best practices to employ so that the resources are shared safely.
  - To avoid race conditions in our programming, we need good synchronization and communication with the rest of the goroutines to make sure they don’t step over each other. Good concurrent programming involves effectively synchronizing your concurrent executions to eliminate race conditions while improving performance and throughput
  - `go run -race stingyspendy.go`
  - Go’s race detector finds race conditions only when a particular race condition is triggered. For this reason, the detector is not infallible. When using the race detector, you should test your code with production-like scenarios, but enabling it in a production environment is usually not desirable, since it slows performance and uses a lot more memory.

## 04 - Synchronization with mutexes

- mutexes
  - We can protect critical sections of our code with mutexes so that only one goroutine at a time accesses a shared resource. In this way, we eliminate race conditions
  - ensure that only one thread of execution runs critical sections
  - Think of them as being physical locks that block certain parts of our code from more than one goroutine at any time. If only one goroutine is accessing a critical section at a time, we are safe from race conditions. After all, race conditions happen only when there is a conflict between two or more goroutines.
  - If another goroutine tries to lock a mutex that is already locked, the goroutine will be suspended until the mutex is released. If more than one goroutine is suspended, waiting for a lock to become available, only one goroutine is resumed, and it is the next to acquire the mutex lock.
  - When we create a new mutex, its initial state is always unlocked.
  - We should protect all critical sections, including parts where the goroutine is only reading the shared resources. The compiler’s optimizations might re-order instructions, causing them to execute in a different manner. Using proper synchronization mechanisms, such as mutexes, ensures that we’re reading the latest copy of the shared resources.
  - Using mutexes has the effect of limiting concurrency. The code in between locking and unlocking a mutex is executed by one goroutine at any time, effectively turning that part of the code into sequential execution. As we saw in chapter 1, and according to Amdahl’s law, the sequential-to-parallel ratio will limit the performance scalability of our code, so it’s essential that we reduce the time spent holding the mutex lock.
  - When deciding how and when to use mutexes, it’s best to focus on which resources we should protect and discover where critical sections start and end. Then we need to think about how to minimize the number of Lock() and Unlock() calls.
  - We are basically maximizing the scalablity of our program by using the locks only on the code sections that run very quickly in proportion to the rest
  - he lesson here is to minimize the amount of time spent holding the mutex lock, while also trying to lower the number of mutex calls. This makes sense if you think back to Amdahl’s law, which tells us that if our code spends more time on the parallel parts, we can finish faster and scale better.
  - A goroutine will block when it calls the Lock() operation if the mutex is already in use by another execution. This is what’s known as a blocking function: the execution of the goroutine stops until Unlock() is called by another goroutine. In some applications, we might not want to block the goroutine, but instead perform some other work before attempting again to lock the mutex and access the critical section.
  - Note that while correct uses of TryLock do exist, they are rare, and use of TryLock is often a sign of a deeper problem in a particular use of mutexes
- readers–writer mutexes
  - We can think of mutexes as blunt tools that solve concurrency problems by blocking concurrency. Only one goroutine at a time can execute our mutex-protected critical section. This is great for guaranteeing that we don’t suffer from race conditions, but this might needlessly restrict performance and scalability for some applications
  - Using readers–writer mutexes, we can improve the performance of read-heavy applications where we are doing a large number of read operations on shared data in comparison with updates.
  - In the main() function, we then start a match-recorder goroutine and 5,000 client handler goroutines. Basically, we are simulating a game that is ongoing and that has a large number of users making simultaneous requests to get game updates. 
  - In comparison to the number of read queries, the data changes very slowly. When we use normal mutex locks, every time a goroutine reads the shared basketball data, it blocks all the other serving goroutines until it’s finished. Even though the client handlers are just reading the shared slice without any modifications, we are still giving each one of them exclusive access to the slice. Note that if multiple goroutines are just reading shared data without updating it, there is no need for this exclusive access; concurrent reading of shared data does not cause any interference.
  - Race conditions only happen if we change the shared state without proper synchronization. If we don’t modify the shared data, there is no risk of race conditions.
  - We would only block access to the shared data if there was a need to update it
  - Thus, we would benefit from a system that allows multiple concurrent reads but exclusive writes.
  - This is what the readers–writer lock gives us. When we just need to read a shared resource without updating it, the readers–writer lock allows multiple concurrent goroutines to execute the read-only critical section part. When we need to update the shared resource, the goroutine executing the write critical section requests the write lock to acquire exclusive access.
  - obtaining a write lock blocks all other access, both read and write, just like a normal mutex.
  - When the write lock is acquired, it will block any other goroutine from accessing the critical section in our clientHandler() function until we release the write lock by calling UnLock().
  - If we have a system running multiple cores, this example should give us a speedup over a system with a single core. That’s because we would be running a number of client handler goroutines in parallel since they can access the shared data at the same time. In a test run, I achieved a threefold increase in throughput performance using the readers–writer mutex:
  - This implementation of the readers–writer lock is read-preferring. This means that if we have a consistent number of readers’ goroutines hogging the read part of the mutex, a writer goroutine would be unable to acquire the mutex. In technical terms, we say that the reader goroutines are starving the writer ones, not allowing them access to the shared resource. In the next chapter, we will improve this when we discuss condition variables.

## 05 - Condition variables and semaphores

- Mutexes are not the only synchronization tool that we have available: condition variables give us extra controls that complement exclusive locking. They give us the ability to wait for a certain condition to occur before unblocking the execution.
- Semaphores go one step further than mutexes in that they allow us to control how many concurrent goroutines can execute a certain section at the same time. In addition, semaphores can be used to store a signal (of an occurring event) for later access by an execution.
- Conditional variables
  - We can use them in situations where a goroutine needs to block and wait for a particular condition to occur
  - Condition variables work together with mutexes and give us the ability to suspend the current execution until we have a signal that a particular condition has changed**
  - The key to understanding condition variables is to grasp that the Wait() function releases the mutex and suspends the execution in an atomic manner. This means that another execution cannot come in between these two operations, acquire the lock, and call the Signal() function before the execution calling Wait() has been suspended.
  - Every time we add money to our shared money variable, we send a signal by calling the Signal() function on the condition variable.
  - Whenever a waiting goroutine receives a signal or broadcast, it will try to reacquire the mutex. If another execution is holding on to the mutex, the goroutine will remain suspended until the mutex becomes available.
  - A monitor is a synchronization pattern that has a mutex with an associated condition variable.
  - In Go, we use the monitor pattern every time we use a mutex with a condition variable.
  - We need to ensure that when we call the signal or broadcast function, there is another goroutine waiting for it; otherwise, the signal or broadcast is not received by any goroutine, and it’s missed.
  - To ensure that we don’t miss any signals and broadcasts, we need to use them in conjunction with mutexes. That is, we should call these functions only when we’re holding the associated mutex. In this way, we know for sure that the main() goroutine is in a waiting state because the mutex is only released when the goroutine calls Wait().
  - We can modify the doWork() function from listing 5.6 so that it locks the mutex before calling signal(), as shown on the right side of figure 5.5. This ensures that the main() goroutine is in a waiting state, as shown in the next listing.
  - TIP Always use Signal(), Broadcast(), and Wait() when holding the mutex lock to avoid synchronization problems.
  - When we have multiple goroutines suspended on a condition variable’s Wait(), Signal() will arbitrarily wake up one of these goroutines. The Broadcast() call, on the other hand, will wake up all goroutines that are suspended on a Wait().
  - When a group of goroutines is suspended on Wait() and we call Signal(), we only wake up one of the goroutines. We have no control over which goroutine the system will resume, and we should assume that it can be any goroutine blocked on the condition variable’s Wait(). Using Broadcast(), we ensure that all suspended goroutines on the condition variable are resumed.
  - When all the other goroutines unblock from the Wait(), as a result of the Broadcast(), they exit the condition-checking loop and release the mutex.
  - In technical-speak, we call this scenario write-starvation—we can’t update our shared data structures because the reader parts of the execution are continuously accessing them, blocking access to the writer
  - Starvation is a situation where an execution is blocked from gaining access to a shared resource because the resource is made unavailable for a long time (or indefinitely) by other greedy executions.
  - We need a different design for a readers–writer lock that is not read-preferred—one that doesn’t starve our writer goroutines
  - To achieve this, instead of having the goroutines block on a mutex, we could have them suspended using a condition variable. With a condition variable, we can have different conditions on when to block readers and writers
  - The RWMutex bundled with Go is write-preferring: If a goroutine holds a RWMutex for reading and another goroutine might call Lock, no goroutine should expect to be able to acquire a read lock until the initial read lock is released. In particular, this prohibits recursive read locking. This is to ensure that the lock eventually becomes available; a blocked Lock call excludes new readers from acquiring the lock.
  - The writers’ waiting counter ensures that any newcomer reader will know there are waiting writers. The reader will then give priority to the writer by blocking until the writers’ waiting counter is back to 0. This is what makes our readers–writer mutex write-preferring.
- Counting semaphores
  - we saw how mutexes allow only one goroutine to have access to a shared resource, while a readers–writer mutex allows us to specify multiple concurrent reads but exclusive writes.
  - Semaphores give us a different type of concurrency control, in that we can specify the number of concurrent executions that are permitted
  - Mutexes give us a way to allow only one execution to happen at a time. What if we need to allow a variable number of executions to happen concurrently?
  - Once the limit is reached, we can either make the goroutines wait or return an error message to the client saying the system is at capacity.
  - This is where semaphores come in handy. They allow a fixed number of permits that enable concurrent executions to access shared resources.
  - Once all the permits are used, further requests for access will have to wait until a permit is freed again 
  - A mutex ensures that only a single goroutine has exclusive access, whereas a semaphore ensures that at most N goroutines have access. In fact, a mutex gives the same functionality as a semaphore where N has a value of 1. A counting semaphore allows us the flexibility to choose any value of N.
  - Although a mutex is a special case of a semaphore with one permit, there is a slight difference in how they are expected to be used. When using mutexes, the execution that is holding a mutex should also be the one to release it. When using semaphores, this is not always the case.
  - Looking at semaphores from another perspective, they provide similar functionality to the wait and signal of a condition variable, with the added benefit of recording a signal even if no goroutine is waiting.
  - In listing 5.6, we saw an example of using condition variables to wait for a goroutine to finish its task. The problem we had was that we could end up calling the Signal() function before the main() goroutine had called Wait(), resulting in a missed signal. We can solve this problem by using a semaphore initialized with 0 permits.

## 06 - Synchronizing with waitgroups and barriers