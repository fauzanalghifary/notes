# Book: Concurrency in GO by Katherine Cox-Buday

- https://learning.oreilly.com/library/view/concurrency-in-go/9781491941294/
- https://github.com/fauzanalghifary/concurrency-in-go-src

## Preface

- we will have discussed the entire stack of Go concurrency concerns: common concurrency pitfalls, motivation behind the design of Go’s concurrency, the basic syntax of Go’s concurrency primitives, common concurrency patterns, patterns of patterns, and various tooling that will help you along the way.

## 01 - An Introduction to Concurrency

- When most people use the word “concurrent,” they’re usually referring to a process that occurs simultaneously with one or more processes. It is also usually implied that all of these processes are making progress at about the same time.
- Why Is Concurrency Hard?
  - Race Conditions
    - A race condition occurs when two or more operations must execute in the correct order, but the program has not been written so that this order is guaranteed to be maintained.
    - Most of the time, this shows up in what’s called a data race, where one concurrent operation attempts to read a variable while at some undetermined time another concurrent operation is attempting to write to the same variable.
    - Most of the time, data races are introduced because the developers are thinking about the problem sequentially.
    - When writing concurrent code, you have to meticulously iterate through the possible scenarios.
  - Atomicity
    - When something is considered atomic, or to have the property of atomicity, this means that within the context that it is operating, it is indivisible, or uninterruptible.
    - When thinking about atomicity, very often the first thing you need to do is to define the context, or scope, the operation will be considered to be atomic in. Everything follows from this.
    - Making the operation atomic is dependent on which context you’d like it to be atomic within
  - Memory Access Synchronization
    - In fact, there’s a name for a section of your program that needs exclusive access to a shared resource. This is called a critical section.
    - one way to solve this problem is to synchronize access to the memory between your critical sections. 
    - It is true that you can solve some problems by synchronizing access to the memory, but as we just saw, it doesn’t automatically solve data races or logical correctness.
    - the calls to Lock you see can make our program slow. Every time we perform one of these operations, our program pauses for a period of time.
  - Deadlocks, Livelocks, and Starvation
    - A deadlocked program is one in which all concurrent processes are waiting on one another. In this state, the program will never recover without outside intervention.
    - It turns out there are a few conditions that must be present for deadlocks to arise (Coffman Conditions)
      - Mutual Exclusion: concurrent process holds exclusive rights to a resource at any one time.
      - Wait For Condition: concurrent process must simultaneously hold a resource and be waiting for an additional resource.
      - No Preemption: resource held by a concurrent process can only be released by that process, so it fulfills this condition.
      - Circular Wait: concurrent process (P1) must be waiting on a chain of other concurrent processes (P2), which are in turn waiting on it (P1), so it fulfills this final condition too.
    - If we ensure that at least one of these conditions is not true, we can prevent deadlocks from occurring.
    - Unfortunately, in practice these conditions can be hard to reason about, and therefore difficult to prevent
    - Livelocks are programs that are actively performing concurrent operations, but these operations do nothing to move the state of the program forward.
    - This example demonstrates a very common reason livelocks are written: two or more concurrent processes attempting to prevent a deadlock without coordination.
    - livelocks are more difficult to spot than deadlocks simply because it can appear as if the program is doing work
    - Livelocks are a subset of a larger set of problems called starvation.
    - Starvation is any situation where a concurrent process cannot get all the resources it needs to perform work.
    - Livelocks warrant discussion separate from starvation because in a livelock, all the concurrent processes are starved equally, and no work is accomplished. More broadly, starvation usually implies that there are one or more greedy concurrent process that are unfairly preventing one or more concurrent processes from accomplishing work as efficiently as possible, or maybe at all.
    - One of the ways you can detect and solve starvation is by logging when work is accomplished, and then determining if your rate of work is as high as you expect it.
    - If you utilize memory access synchronization, you’ll have to find a balance between preferring coarse-grained synchronization for performance, and fine-grained synchronization for fairness
    - When it comes time to performance tune your application, to start with, I highly recommend you constrain memory access synchronization only to critical sections; if the synchronization becomes a performance problem, you can always broaden the scope. It’s much harder to go the other way.
- How do you expose the concurrency to callers? What techniques do you use to create a solution that is both easy to use and modify? What is the right level of concurrency for this problem? Although there are ways to think about these problems in structured ways, it remains an art.
- Concurrency is certainly a difficult area in computer science, but I want to leave you with hope: these problems aren’t intractable, and with Go’s concurrency primitives, you can more safely and clearly express your concurrent algorithms
- the excellent work that has been done on Go’s garbage collector has dramatically reduced the audience that needs to concern themselves with the minutia of how Go’s garbage collection works
- Go has made it much easier to use concurrency in your program by not forcing you to manage memory, let alone across concurrent processes.
- Go’s runtime also automatically handles multiplexing concurrent operations onto operating system threads.
- all you need to know is that it allows you to directly map concurrent problems into concurrent constructs instead of dealing with the minutia of starting and managing threads, and mapping logic evenly across available threads.
- For example, say you write a web server, and you’d like every connection accepted to be handled concurrently with every other connection. In some languages, before your web server begins accepting connections, you’d likely have to create a collection of threads, commonly called a thread pool, and then map incoming connections onto threads. Then, within each thread you’ve created, you’d need to loop over all the connections on that thread to ensure they were all receiving some CPU time. In addition, you’d have to write your connection-handling logic to be pausable so that it shares fairly with the other connections.
- Whew! In contrast, in Go you would write a function and then prepend its invocation with the go keyword. The runtime handles everything else we discussed automatically!

## 02 - Modeling Your Code: Communicating Sequential Processes

- The Difference Between Concurrency and Parallelism
  - The difference between concurrency and parallelism turns out to be a very powerful abstraction when modeling your code, and Go takes full advantage of this
  - Concurrency is a property of the code; parallelism is a property of the running program.
  - The first is that we do not write parallel code, only concurrent code that we hope will be run in parallel. Once again, parallelism is a property of the runtime of our program, not the code.
  - The second interesting thing is that we see it is possible—maybe even desirable—to be ignorant of whether our concurrent code is actually running in parallel
  - The third and final interesting thing is that parallelism is a function of time, or context. For example, if our context was a space of five seconds, and we ran two operations that each took a second to run, we would consider the operations to have run in parallel. If our context was one second, we would consider the operations to have run sequentially.
  - What’s happening is that as we begin moving down the stack of abstraction, the problem of modeling things concurrently is becoming both more difficult to reason about, and more important
  - In other words, the more difficult it is to get concurrency right, the more important it is to have access to concurrency primitives that are easy to compose. Unfortunately, most concurrent logic in our industry is written at one of the highest levels of abstraction: OS threads.
  - Before Go was first revealed to the public, this was where the chain of abstraction ended for most of the popular programming languages. If you wanted to write concurrent code, you would model your program in terms of threads and synchronize the access to the memory between them. 
  - Threads are still there, of course, but we find that we rarely have to think about our problem space in terms of OS threads. Instead, we model things in goroutines and channels, and occasionally shared memory
- CSP
  - the shared memory model can be difficult to utilize correctly—especially in large or complicated programs. It’s for this reason that concurrency is considered one of Go’s strengths: it has been built from the start with principles from CSP in mind and therefore it is easy to read, write, and reason about.
- How This Helps You
  - What does Go do so differently that has set it apart from other popular languages when it comes to concurrency?
  - Goroutines free us from having to think about our problem space in terms of parallelism and instead allow us to model problems closer to their natural level of concurrency. 
  - In Go, we can almost directly represent the natural state of this problem in code: we would create a goroutine for each incoming connection, field the request there (potentially communicating with other goroutines for data/services), and then return from the goroutine’s function
  - This is achieved by a promise Go makes to us: that goroutines are lightweight, and we normally won’t have to worry about creating one
  - In the case of Go, the language was designed around concurrency.
  - This decoupling of concurrency and parallelism has another benefit: because Go’s runtime is managing the scheduling of goroutines for you, it can introspect on things like goroutines blocked waiting for I/O and intelligently reallocate OS threads to goroutines that are not blocked. This also increases the performance of your code.
  - we’ll naturally be writing concurrent code at a finer level of granularity than we perhaps would in other languages
- Go’s Philosophy on Concurrency
  - CSP was and is a large part of what Go was designed around; however, Go also supports more traditional means of writing concurrent code through memory access synchronization and the primitives that follow that technique. Structs and methods in the sync and other packages allow you to perform locks, create pools of resources, preempt goroutines, and more.
  - Package sync provides basic synchronization primitives such as mutual exclusion locks. Other than the Once and WaitGroup types, most are intended for use by low-level library routines. Higher-level synchronization is better done via channels and communication.
  - "Regarding mutexes, the sync package implements them, but we hope Go programming style will encourage people to try higher-level techniques. In particular, consider structuring your program so that only one goroutine at a time is ever responsible for a particular piece of data."
  - "Do not communicate by sharing memory. Instead, share memory by communicating."
  - One of Go’s mottos is “Share memory by communicating, don’t communicate by sharing memory.” 
  - That said, Go does provide traditional locking mechanisms in the sync package. Most locking issues can be solved using either channels or traditional locks. So which should you use? Use whichever is most expressive and/or most simple.
  - the way we can mostly differentiate comes from where we’re trying to manage our concurrency: internally to a tight scope, or externally throughout our system
  - Are you trying to transfer ownership of data? USE CHANNELS. If you have a bit of code that produces a result and wants to share that result with another bit of code, what you’re really doing is transferring ownership of that data.
  - Are you trying to guard internal state of a struct? USE MUTEX. By using memory access synchronization primitives, you can hide the implementation detail of locking your critical section from your callers.
  - Are you trying to coordinate multiple pieces of logic? USE CHANNELS. Having locks scattered throughout your object-graph sounds like a nightmare, but having channels everywhere is expected and encouraged! I can compose channels, but I can’t easily compose locks or methods that return values.
  - You will find it much easier to control the emergent complexity that arises in your software if you use channels because of Go’s select statement, and their ability to serve as queues and be safely passed around
  - If you find yourself struggling to understand how your concurrent code works, why a deadlock or race is occurring, and you’re using primitives, this is probably a good indicator that you should switch to channels.
  - Is it a performance-critical section? USE MUTEX. If you have a section of your program that you have profiled, and it turns out to be a major bottleneck that is orders of magnitude slower than the rest of the program, using memory access synchronization primitives may help this critical section perform under load. This is because channels use memory access synchronization to operate, therefore they can only be slower. however, a performance-critical section might be hinting that we need to restructure our program.
  - Go’s philosophy on concurrency can be summed up like this: aim for simplicity, use channels when possible, and treat goroutines like a free resource.

## 03 - Go’s Concurrency Building Blocks

- Goroutines
  - Every Go program has at least one goroutine: the main goroutine, which is automatically created and started when the process begins.
  - a goroutine is a function that is running concurrently (remember: not necessarily in parallel!) alongside other code. You can start one simply by placing the go keyword before a function.
  - They’re not OS threads, and they’re not exactly green threads—threads that are managed by a language’s runtime—they’re a higher level of abstraction known as coroutines. Coroutines are simply concurrent subroutines (functions, closures, or methods in Go) that are nonpreemptive—that is, they cannot be interrupted. Instead, coroutines have multiple points throughout which allow for suspension or reentry.
  - What makes goroutines unique to Go are their deep integration with Go’s runtime. 
  - Go’s runtime observes the runtime behavior of goroutines and automatically suspends them when they block and then resumes them when they become unblocked
  - goroutines can be considered a special class of coroutine.
  - Go follows a model of concurrency called the fork-join model.
  - The go statement is how Go performs a fork, and the forked threads of execution are goroutines
  - In order to a create a join point, you have to synchronize the main goroutine and the sayHello goroutine. This can be done in a number of ways, but I’ll use one we’ll talk about in “The sync Package”: sync.WaitGroup.
  - Closures close around the lexical scope they are created in, thereby capturing variables
  - Interesting! It turns out that goroutines execute within the same address space they were created in
  - Because goroutines operate within the same address space as each other, and simply host functions, utilizing goroutines is a natural extension to writing nonconcurrent code. Go’s compiler nicely takes care of pinning variables in memory so that goroutines don’t accidentally access freed memory, which allows developers to focus on their problem space instead of memory management;
  - Since multiple goroutines can operate against the same address space, we still have to worry about synchronization
  - It is practical to create hundreds of thousands of goroutines in the same address space. If goroutines were just threads, system resources would run out at a much smaller number.
  - the garbage collector does nothing to collect goroutines that have been abandoned somehow.
  - Those numbers are quite large! On my laptop I have 8 GB of RAM, which means that in theory I can spin up millions of goroutines without requiring swapping.
  - Creating goroutines is very cheap, and so you should only be discussing their cost if you’ve proven they are the root cause of a performance issue.
- The sync Package
  - The sync package contains the concurrency primitives that are most useful for low-level memory access synchronization.
  - WaitGroup
    - WaitGroup is a great way to wait for a set of concurrent operations to complete when you either don’t care about the result of the concurrent operation, or you have other means of collecting their results
    - You can think of a WaitGroup like a concurrent-safe counter: calls to Add increment the counter by the integer passed in, and calls to Done decrement the counter by one. Calls to Wait block until the counter is zero.
    - Notice that the calls to Add are done outside the goroutines they’re helping to track. If we didn’t do this, we would have introduced a race condition
    - Had the calls to Add been placed inside the goroutines’ closures, the call to Wait could have returned without blocking at all because the calls to Add would not have taken place.
  - Mutex and RWMutex
    - Mutex stands for “mutual exclusion” and is a way to guard critical sections of your program
    - a critical section is an area of your program that requires exclusive access to a shared resource. A Mutex provides a concurrent-safe way to express exclusive access to these shared resources
    - whereas channels share memory by communicating, a Mutex shares memory by creating a convention developers must follow to synchronize access to the memory. You are responsible for coordinating access to this memory by guarding access to it with a mutex
    - whereas channels share memory by communicating, a Mutex shares memory by creating a convention developers must follow to synchronize access to the memory. You are responsible for coordinating access to this memory by guarding access to it with a mutex
    - You’ll notice that we always call Unlock within a defer statement. This is a very common idiom when utilizing a Mutex to ensure the call always happens, even when panicing. Failing to do so will probably cause your program to deadlock.
    - Critical sections are so named because they reflect a bottleneck in your program. It is somewhat expensive to enter and exit a critical section, and so generally people attempt to minimize the time spent in critical sections.
    - One strategy for doing so is to reduce the cross-section of the critical section. There may be memory that needs to be shared between multiple concurrent processes, but perhaps not all of these processes will read and write to this memory. If this is the case, you can take advantage of a different type of mutex: sync.RWMutex.
    - RWMutex gives you a little bit more control over the memory. You can request a lock for reading, in which case you will be granted access unless the lock is being held for writing. This means that an arbitrary number of readers can hold a reader lock so long as nothing else is holding a writer lock.
    - It’s usually advisable to use RWMutex instead of Mutex when it logically makes sense.
  - Cond
    - a rendezvous point for goroutines waiting for or announcing the occurrence of an event.
    - an “event” is any arbitrary signal between two or more goroutines that carries no information other than the fact that it has occurred.
    - It would be better if there were some kind of way for a goroutine to efficiently sleep until it was signaled to wake and check its condition. This is exactly what the Cond type does for us.
    - Note that the call to Wait doesn’t just block, it suspends the current goroutine, allowing other goroutines to run on the OS thread.
    - A few other things happen when you call Wait: upon entering Wait, Unlock is called on the Cond variable’s Locker, and upon exiting Wait, Lock is called on the Cond variable’s Locker
    - We also have a new method in this example, Signal. This is one of two methods that the Cond type provides for notifying goroutines blocked on a Wait call that the condition has been triggered.
    - the Cond type is much more performant than utilizing channels.
  - Once
    - sync.Once is a type that utilizes some sync primitives internally to ensure that only one call to Do ever calls the function passed in—even on different goroutines
  - Pool
    - It’s commonly used to constrain the creation of things that are expensive (e.g., database connections) so that only a fixed number of them are ever created, but an indeterminate number of operations can still request access to these things
- Channels
  - they are best used to communicate information between goroutines
  - When using channels, you’ll pass a value into a chan variable, and then somewhere else in your program read it off the channel.
  - Channels can also be declared to only support a unidirectional flow of data
  - Sending is done by placing the <- operator to the right of a channel, and receiving is done by placing the <- operator to the left of the channel
  - however, it is an error to try and write a value onto a read-only channel, and an error to read a value from a write-only channel.
  - channels in Go are said to be blocking. This means that any goroutine that attempts to write to a channel that is full will wait until the channel has been emptied, and any goroutine that attempts to read from a channel that is empty will wait until at least one item is placed on it. 
  - This can cause deadlocks if you don’t structure your program correctly. When the anonymous goroutine exits, Go correctly detects that all goroutines are asleep, and reports a deadlock
  - The receiving form of the <- operator can also optionally return two values: the value read from the channel, and a boolean that indicates whether the channel is open or closed
  - In programs, it’s very useful to be able to indicate that no more values will be sent over a channel
  - Notice that we never placed anything on this channel; we closed it immediately. We were still able to perform a read operation, and in fact, we could continue performing reads on this channel indefinitely despite the channel remaining closed.
  - Closing a channel is also one of the ways you can signal multiple goroutines simultaneously.
  - We can also create buffered channels, which are channels that are given a capacity when they’re instantiated. This means that even if no reads are performed on the channel, a goroutine can still perform n writes, where n is the capacity of the buffered channel
  - When we create a buffered channel with a capacity of four, it means that we can place four things onto the channel regardless of whether it’s being read from.
  - an unbuffered channel is simply a buffered channel created with a capacity of 0
  - writes to a channel block if a channel is full, and reads from a channel block if the channel is empty.
  - An unbuffered channel has a capacity of zero and so it’s already full before any writes.
  - An unbuffered channel has a capacity of zero and so it’s already full before any writes.
  - Like unbuffered channels, buffered channels are still blocking; the preconditions that the channel be empty or full are just different. In this way, buffered channels are an in-memory FIFO queue for concurrent processes to communicate over.
  - It also bears mentioning that if a buffered channel is empty and has a receiver, the buffer will be bypassed and the value will be passed directly from the sender to the receiver.
  - Buffered channels can be useful in certain situations, but you should create them with care. As we’ll see in the next chapter, buffered channels can easily become a premature optimization and also hide deadlocks by making them more unlikely to happen.
  - This is an example of an optimization that can be useful under the right conditions: if a goroutine making writes to a channel has knowledge of how many writes it will make, it can be useful to create a buffered channel whose capacity is the number of writes to be made, and then make those writes as quickly as possible
  - If we examine this table (Table 3-2), we see a few areas that could lead to trouble. We have three operations that can cause a goroutine to block, and three operations that can cause your program to panic!
  - The first thing we should do to put channels in the right context is to assign channel ownership. I’ll define ownership as being a goroutine that instantiates, writes, and closes a channel
  - channel owners have a write-access view into the channel (chan or chan<-), and channel utilizers only have a read-only view into the channel (<-chan).
  - Now let’s look at those blocking operations that can occur when reading. As a consumer of a channel, I only have to worry about two things: Knowing when a channel is closed and responsibly handling blocking for any reason.
  - The important thing is that as a consumer you should handle the fact that reads can and will block
  - Channels were one of the things that drew me to Go in the first place. Combined with the simplicity of goroutines and closures, it was apparent to me how easy it would be to write clean, correct, concurrent code. In many ways, channels are the glue that binds goroutines together.
- The select Statement
  - The select statement is the glue that binds channels together; it’s how we’re able to compose channels together in a program to form larger abstractions
  - select statements are one of the most crucial things in a Go program with concurrency
  - select statements can help safely bring channels together with concepts like cancellations, timeouts, waiting, and default values.
  - Unlike switch blocks, case statements in a select block aren’t tested sequentially, and execution won’t automatically fall through if none of the criteria are met.
  - Instead, all channel reads and writes are considered simultaneously to see if any of them are ready: populated or closed channels in the case of reads, and channels that are not at capacity in the case of writes. If none of the channels are ready, the entire select statement blocks. Then when one the channels is ready, that operation will proceed, and its corresponding statements will execute
  - What happens when multiple channels have something to read? What if there are never any channels that become ready? What if we want to do something but no channels are currently ready?
  - The Go runtime will perform a pseudo-random uniform selection over the set of case statements. This just means that of your set of case statements, each has an equal chance of being selected as all the others.
  - By weighting the chance of each channel being utilized equally, all Go programs that utilize the select statement will perform well in the average case.
  - What about the second question: what happens if there are never any channels that become ready? If there’s nothing useful you can do when all the channels are blocked, but you also can’t block forever, you may want to time out.
  - This leaves us the remaining question: what happens when no channel is ready, and we need to do something in the meantime? Like case statements, the select statement also allows for a default clause in case you’d like to do something if all the channels you’re selecting against are blocking.
  - You can see that it ran the default statement almost instantaneously. This allows you to exit a select block without blocking. Usually you’ll see a default clause used in conjunction with a for-select loop. This allows a goroutine to make progress on work while waiting for another goroutine to report a result.
- The GOMAXPROCS Lever
  - people often think this function relates to the number of logical processors on the host machine—and in a roundabout way it does—but really this function controls the number of OS threads that will host so-called “work queues.”
  - So why would you want to tweak this value? Most of the time you won’t want to. Go’s scheduling algorithm is good enough in most situations that increasing or decreasing the number of worker queues and threads will likely do more harm than good, but there are still some situations where changing this value might be useful.
- Conclusion
  - All that remains to understand when writing concurrent Go code is how to combine these primitives in structured ways that scale and are easy to understand

## 04 - Concurrency Patterns in Go

- Confinement
  - When working with concurrent code, there are a few different options for safe operation. We’ve gone over two of them: Synchronization primitives for sharing memory (e.g., sync.Mutex) and Synchronization via communicating (e.g., channels)
  - However, there are a couple of other options that are implicitly safe within multiple concurrent processes: Immutable data and Data protected by confinement
  - Confinement is the simple yet powerful idea of ensuring information is only ever available from one concurrent process. When this is achieved, a concurrent program is implicitly safe and no synchronization is needed. There are two kinds of confinement possible: ad hoc and lexical.
  - Ad hoc confinement is when you achieve confinement through a convention—whether it be set by the languages community, the group you work within, or the codebase you work within (difficult to achieve)
  lexical confinement: it wields the compiler to enforce the confinement.
  - Because of the lexical scope, we’ve made it impossible1 to do the wrong thing, and so we don’t need to synchronize memory access or share data through communication.
  - So what’s the point? Why pursue confinement if we have synchronization available to us? The answer is improved performance and reduced cognitive load on developers. Synchronization comes with a cost, and if you can avoid it you won’t have any critical sections, and therefore you won’t have to pay the cost of synchronizing them. You also sidestep an entire class of issues possible with synchronization;
  - Concurrent code that utilizes lexical confinement also has the benefit of usually being simpler to understand than concurrent code without lexically confined variables. This is because within the context of your lexical scope you can write synchronous code.
  - Having said that, it can be difficult to establish confinement, and so sometimes we have to fall back to our wonderful Go concurrency primitives.
- The for-select Loop
  - Sending iteration variables out on a channel
  - Looping infinitely waiting to be stopped
- Preventing Goroutine Leaks
  - goroutines are not garbage collected by the runtime, so regardless of how small their memory footprint is, we don’t want to leave them lying about our process. So how do we go about ensuring they’re cleaned up?
  - The goroutine has a few paths to termination: When it has completed its work, When it cannot continue its work due to an unrecoverable error, and When it’s told to stop working.
  - whether or not a child goroutine should continue executing might be predicated on knowledge of the state of many other goroutines. The parent goroutine (often the main goroutine) with this full contextual knowledge should be able to tell its child goroutines to terminate
  - The way to successfully mitigate this is to establish a signal between the parent goroutine and its children that allows the parent to signal cancellation to its children. By convention, this signal is usually a read-only channel named done. The parent goroutine passes this channel to the child goroutine and then closes the channel when it wants to cancel the child goroutine
  - If a goroutine is responsible for creating a goroutine, it is also responsible for ensuring it can stop the goroutine.
- The or-channel
  - combine one or more done channels into a single done channel that closes if any of its component channels close.
- Error Handling
  - as we develop our programs, we should give our error paths the same attention we give our algorithms.
  - The most fundamental question when thinking about error handling is, “Who should be responsible for handling the error?” At some point, the program needs to stop ferrying the error up the stack and actually do something with it. What is responsible for this?
  - I suggest you separate your concerns: in general, your concurrent processes should send their errors to another part of your program that has complete information about the state of your program, and can make a more informed decision about what to do.
  - coupled the potential result with the potential error.
  - This represents the complete set of possible outcomes created from the goroutine checkStatus, and allows our main goroutine to make decisions about what to do when errors occur
  - we’ve successfully separated the concerns of error handling from our producer goroutine. This is desirable because the goroutine that spawned the producer goroutine—in this case our main goroutine—has more context about the running program, and can make more intelligent decisions about what to do with errors.
  - the main takeaway here is that errors should be considered first-class citizens when constructing values to return from goroutines. If your goroutine can produce errors, those errors should be tightly coupled with your result type, and passed along through the same lines of communication—just like regular synchronous functions.
- Pipelines
  - When you write a program, you probably don’t sit down and write one long function—at least I hope you don’t! You construct abstractions in the form of functions, structs, methods, etc. Why do we do this? Partly to abstract away details that don’t matter to the greater flow, and partly so that we can work on one area of code without affecting other areas
  - A pipeline is nothing more than a series of things that take data in, perform an operation on it, and pass the data back out. We call each of these operations a stage of the pipeline.
  - what are the properties of a pipeline stage? A stage consumes and returns the same type. A stage must be reified2 by the language so that it may be passed around. Functions in Go are reified and fit this purpose nicely.
  - batch processing. This just means that they operate on chunks of data all at once instead of one discrete value at a time
  - stream processing. This means that the stage receives and emits one element at a time.
  - Channels are uniquely suited to constructing pipelines in Go because they fulfill all of our basic requirements. They can receive and emit values, they can safely be used concurrently, they can be ranged over, and they are reified by the language
  - So in a nutshell, the generator function converts a discrete set of values into a stream of data on a channel. Aptly, this type of function is called a generator. You’ll see this frequently when working with pipelines because at the beginning of the pipeline, you’ll always have some batch of data that you need to convert to a channel
  - Regardless of what state the pipeline stage is in—waiting on the incoming channel, or waiting on the send—closing the done channel will force the pipeline stage to terminate.
  - a generator for a pipeline is any function that converts a set of discrete values into a stream of values on a channel
  - Empty interfaces are a bit taboo in Go, but for pipeline stages it is my opinion that it’s OK to deal in channels of interface{} so that you can use a standard library of pipeline patterns
- Fan-Out, Fan-In
  - Fan-out is a term to describe the process of starting multiple goroutines to handle input from the pipeline, and fan-in is a term to describe the process of combining multiple results into one channel.
  - You might consider fanning out one of your stages if both of the following apply: It doesn’t rely on values that the stage had calculated before and It takes a long time to run.
  - how do we know which one to fan out? Remember our criteria from earlier: order-independence and duration
  - In a nutshell, fanning in involves creating the multiplexed channel consumers will read from, and then spinning up one goroutine for each incoming channel, and one goroutine to close the multiplexed channel when the incoming channels have all been closed
  - A naive implementation of the fan-in, fan-out algorithm only works if the order in which results arrive is unimportant. We have done nothing to guarantee that the order in which items are read from the randIntStream is preserved as it makes its way through the sieve.
- The or-done-channel
- The tee-channel
  - Sometimes you may want to split values coming in from a channel so that you can send them off into two separate areas of your codebase.
- The bridge-channel
  - In some circumstances, you may find yourself wanting to consume values from a sequence of channels
  - Thanks to bridge, we can use the channel of channels from within a single range statement and focus on our loop’s logic
- Queuing
  - Sometimes it’s useful to begin accepting work for your pipeline even though the pipeline is not yet ready for more. This process is called queuing.
  - All this means is that once your stage has completed some work, it stores it in a temporary location in memory so that other stages can retrieve it later, and your stage doesn’t need to hold a reference to it
  - While introducing queuing into your system is very useful, it’s usually one of the last techniques you want to employ when optimizing your program. Adding queuing prematurely can hide synchronization issues such as deadlocks and livelocks, and further, as your program converges toward correctness, you may find that you need more or less queuing.
  - one of the common mistakes people make when trying to tune the performance of a system: introducing queues to try and address performance concerns. Queuing will almost never speed up the total runtime of your program; it will only allow the program to behave differently.
  - So the answer to our question of the utility of introducing a queue isn’t that the runtime of one of stages has been reduced, but rather that the time it’s in a blocking state is reduced. This allows the stage to continue doing its job. In this example, users would likely experience lag in their requests, but they wouldn’t be denied service altogether.
  - In this way, the true utility of queues is to decouple stages so that the runtime of one stage has no impact on the runtime of another.
  - situations in which queuing can increase the overall performance of your system. The only applicable situations are: If batching requests in a stage saves time and If delays in a stage produce a feedback loop into the system.
  - As anticipated, the buffered write is faster than the unbuffered write. This is because in bufio.Writer, the writes are queued internally into a buffer until a sufficient chunk has been accumulated, and then the chunk is written out. This process is often called chunking, for obvious reasons.
  - Chunking is faster because bytes.Buffer must grow its allocated memory to accommodate the bytes it must store. For various reasons, growing memory is expensive; therefore, the less times we have to grow, the more efficient our system as a whole will perform. Thus, queuing has increased the performance of our system as a whole.
  - Usually anytime performing an operation requires an overhead, chunking may increase system performance. Some examples of this are opening database transactions, calculating message checksums, and allocating contiguous space.
  - Aside from chunking, queuing can also help if your algorithm can be optimized by supporting lookbehinds, or ordering.
  - If the efficiency of the pipeline drops below a certain critical threshold, the systems upstream from the pipeline begin increasing their inputs into the pipeline, which causes the pipeline to lose more efficiency, and the death-spiral begins. Without some sort of fail-safe, the system utilizing the pipeline will never recover.
  - By introducing a queue at the entrance to the pipeline, you can break the feedback loop at the cost of creating lag for requests. From the perspective of the caller into the pipeline, the request appears to be processing, but taking a very long time. As long as the caller doesn’t time out, your pipeline will remain stable. If the caller does time out, you need to be sure you support some kind of check for readiness when dequeuing. If you don’t, you can inadvertently create a feedback loop by processing dead requests thereby decreasing the efficiency of your pipeline.
  - So from our examples we can begin to see a pattern emerge; queuing should be implemented either: At the entrance to your pipeline, or In stages where batching will lead to higher efficiency.
  - In a pipeline, a stable system is one in which the rate that work enters the pipeline, or ingress, is equal to the rate in which it exits the system, or egress. If the rate of ingress exceeds the rate of egress, your system is unstable and has entered a death-spiral. If the rate of ingress is less than the rate of egress, you still have an unstable system, but all that’s happening is that your resources aren’t being utilized completely. Not the worst situation in the world, but maybe you care about this if the underutilization is found on a vast scale (e.g., clusters or data centers).
  - Through Little’s Law, we have proven that queuing will not help decrease the amount of time spent in a system.
  - your pipeline will only be as fast as your slowest stage.
  - Remember that as you increase the queue size, it takes your work longer to make it through the system! You’re effectively trading system utilization for lag.
  - Something that Little’s Law can’t provide insight on is handling failure. Keep in mind that if for some reason your pipeline panics, you’ll lose all the requests in your queue. This might be something to guard against if re-creating the requests is difficult or won’t happen. To mitigate this, you can either stick to a queue size of zero, or you can move to a persistent queue, which is simply a queue that is persisted somewhere that can be later read from should the need arise.
  - Queuing can be useful in your system, but because of its complexity, it’s usually one of the last optimizations I would suggest implementing.
- The context Package
  - It would be useful if we could communicate extra information alongside the simple notification to cancel: why the cancellation was occuring, or whether or not our function has a deadline by which it needs to complete.
  - It turns out that the need to wrap a done channel with this information is very common in systems of any size, and so the Go authors decided to create a standard pattern for doing so
  - the context package serves two primary purposes: To provide an API for canceling branches of your call-graph, and To provide a data-bag for transporting request-scoped data through your call-graph.
  - Since both the Context’s key and value are defined as interface{}, we lose Go’s type-safety when attempting to retrieve values. The key could be a different type, or slightly different than the key we provide. The value could be a different type than we’re expecting. For these reasons, the Go authors recommend you follow a few rules when storing and retrieving value from a Context.
  - First, they recommend you define a custom key-type in your package
  - The context package is pretty neat, but it hasn’t been uniformly lauded. Within the Go community, the context package has been somewhat controversial. The cancellation aspect of the package has been pretty well received, but the ability to store arbitrary data in a Context, and the type-unsafe manner in which the data is stored, have caused some divisiveness
  - However, the larger issue is definitely the nature of what developers should store in instances of Context.
  - you shouldn’t be using a Context to fulfill your secret desire for Go to support optional parameters)
  - Here are my heuristics: The data should transit process or API boundaries, The data should be immutable, The data should trend toward simple types, The data should be data, not types with methods, AND The data should help decorate operations, not drive them.
  - If your algorithm behaves differently based on what is or isn’t included in its Context, you have likely crossed over into the territory of optional parameters.
  - These aren’t hard-and-fast rules; they’re heuristics.
  - The final advice I’d leave you with is that the cancellation functionality provided by Context is very useful, and your feelings about the data-bag shouldn’t deter you from using it.

## 05 - Concurrency at Scale

- Error Propagation
  - Many developers make the mistake of thinking of error propagation as secondary, or “other,” to the flow of their system. Careful consideration is given to how data flows through the system, but errors are something that are tolerated and ferried up the stack without much thought, and ultimately dumped in front of the user.
  - Errors indicate that your system has entered a state in which it cannot fulfill an operation that a user either explicitly or implicitly requested. Because of this, it needs to relay a few pieces of critical information: What happened, When and where it occurred, A friendly user-facing message, and How the user can get more information
  - Errors should always contain a complete stack trace starting with how the call was initiated and ending with where the error was instantiated
  - the error should contain information regarding the context it’s running within.
  - the error should contain the time on the machine the error was instantiated on, in UTC.
  - The message that gets displayed to the user should be customized to suit your system and its users. A friendly message is human-centric, gives some indication of whether the issue is transitory, and should be about one line of text.
  - Errors that are presented to users should provide an ID that can be cross-referenced to a corresponding log that displays the full information of the error: time the error occurred (not the time the error was logged), the stack trace—everything you stuffed into the error when it was created.
  - By default, no error will contain all of this information without your intervention. Therefore, you could take the stance that any error that is propagated to the user without this information is a mistake, and therefore a bug.
  -  It’s possible to place all errors into one of two categories: Bugs and Known edge cases (e.g., broken network connections, failed disk writes, etc.
  - Error correctness becomes an emergent property of our system. We also concede perfection from the start by explicitly handling malformed errors, and by doing so we have given ourselves a framework to take mistakes and correct them over time.
  - As we established, all errors should be logged with as much information as is available. But when displaying errors to users, this is where the distinction between bugs and known edge cases comes in.
  - When our user-facing code receives a well-formed error, we can be confident that at all levels in our code, care was taken to craft the error message, and we can simply log it and print it out for the user to see. The confidence that we get from seeing an error with the correct type cannot be understated.
  - you can canvas your top-level error handling and delineate between bugs and well-crafted errors, and then progressively ensure that all the errors you create are considered well-crafted.
- Timeouts and Cancellation
  - When working with concurrent code, timeouts and cancellations are going to turn up frequently
  - Why support timeouts: System saturation, Stale data, and Attempting to prevent deadlocks
  - when to time out: If the request is unlikely to be repeated when it is timed out, If you don’t have the resources to store the requests (e.g., memory for in-memory queues, disk space for persisted queues), and If the need for the request, or the data it’s sending, will go stale
  - The timeout period’s purpose is only to prevent deadlock, and so it only needs to be short enough that a deadlocked system will unblock in a reasonable amount of time for your use case.
  - Therefore, it is preferable to chance a livelock and fix that as time permits, than for a deadlock to occur and have a system recoverable only by restart.
  - Note that this isn’t a recommendation for how to build a system correctly; rather a suggestion for building a system that is tolerant to timing errors you may not have exercised during development and testing. I do recommend you keep the timeouts in place, but the goal should be to converge on a system without deadlocks where the timeouts are never triggered.
  - why a concurrent process might be canceled: Timeouts, User intervention, Parent cancellation, Replicated requests
  - An easy way to do this is to break up the pieces of your goroutine into smaller pieces. You should aim for all nonpreemptable atomic operations to complete in less time than the period you’ve deemed acceptable.
  - Here’s another problem lurking here as well: if our goroutine happens to modify shared state—e.g., a database, a file, an in-memory data structure—what happens when the goroutine is canceled?
  - Another issue you need to be concerned with is duplicated messages.
  - There are a few ways to avoid sending duplicate messages. The easiest (and the method I recommend) is to make it vanishingly unlikely that a parent goroutine will send a cancellation signal after a child goroutine has already reported a result. 
  - When designing your concurrent processes, be sure to take into account timeouts and cancellation. Like many other topics in software engineering, neglecting timeouts and cancellation from the beginning and then attempting to put them in later is a bit like trying to add eggs to a cake after it has been baked.
- Heartbeats
  - Heartbeats are a way for concurrent processes to signal life to outside parties.
  - There are two different types of heartbeats we’ll discuss in this section: Heartbeats that occur on a time interval and Heartbeats that occur at the beginning of a unit of work.
  - Heartbeats that occur on a time interval are useful for concurrent code that might be waiting for something else to happen for it to process a unit of work. Because you don’t know when that work might come in, your goroutine might be sitting around for a while waiting for something to happen. A heartbeat is a way to signal to its listeners that everything is well, and that the silence is expected.
  - Now in a properly functioning system, heartbeats aren’t that interesting. We might use them to gather statistics regarding idle time, but the utility for interval-based heartbeats really shines when your goroutine isn’t behaving as expected.
  - By using a heartbeat, we have successfully avoided a deadlock, and we remain deterministic by not having to rely on a longer timeout.
  - This test is bad because it’s nondeterministic
- Replicated Requests
  - You should only replicate out requests like this to handlers that have different runtime conditions: different processes, machines, paths to a data store, or access to different data stores altogether.
  - Although this is can be expensive to set up and maintain, if speed is your goal, this is a valuable technique. In addition, this naturally provides fault tolerance and scalability.
- Rate Limiting
  - The most obvious answer is that by rate limiting a system, you prevent entire classes of attack vectors against your system
  - The point is: if you don’t rate limit requests to your system, you cannot easily secure it.
  - A user’s mental model is usually that their access to the system is sandboxed and can neither affect nor be affected by other users’ activities. If you break that mental model, your system can feel like it’s not well engineered, and even cause users to become angry or leave.
  - Rate limits allow you to reason about the performance and stability of your system by preventing it from falling outside the boundaries you’ve already investigated. If you need to expand those boundaries, you can do so in a controlled manner after lots of testing and coffee.
  - Hopefully I’ve given enough justification to convince you that rate limits are good even if you set limits that you think will never be reached. They’re pretty simple to create, and they solve so many problems that it’s hard to rationalize not using them.
  - Most rate limiting is done by utilizing an algorithm called the token bucket.
  - In the token bucket algorithm, we define r to be the rate at which tokens are added back to the bucket. It can be one a nanosecond, or one a minute. This becomes what we commonly think of as the rate limit: because we have to wait until new tokens become available, we limit our operations to that refresh rate.
  - So we now have two settings we can fiddle with: how many tokens are available for immediate use—d, the depth of the bucket—and the rate at which they are replenished—r. Between these two we can control both the burstiness and overall rate limit. Burstiness simply means how many requests can be made when the bucket is full.
  - We will probably want to establish multiple tiers of limits: fine-grained controls to limit requests per second, and coarse-grained controls to limit requests per minute, hour, or day.
- Healing Unhealthy Goroutines
  - In long-lived processes such as daemons, it’s very common to have a set of long-lived goroutines. These goroutines are usually blocked, waiting on data to come to them through some means, so that they can wake up, do their work, and then pass the data on
  - The point is that it can be very easy for a goroutine to become stuck in a bad state from which it cannot recover without external help.
  - In a long-running process, it can be useful to create a mechanism that ensures your goroutines remain healthy and restarts them if they become unhealthy. We’ll refer to this process of restarting goroutines as “healing.”

## 06 - Goroutines and the Go Runtime

- Work Stealing
  - In Go, goroutines are tasks.
  - Everything after a goroutine is called is the continuation.
  - Go’s work-stealing algorithm enqueues and steals continuations.
  - All things considered, stealing continuations are considered to be theoretically superior to stealing tasks, and therefore it is best to queue the continuation and not the goroutine
  - As we mentioned, the GOMAXPROCS setting controls how many contexts are available for use by the runtime. The default setting is for there to be one context per logical CPU on the host machine. Unlike contexts, there may be more or less OS threads than cores to help Go’s runtime manage things like garbage collection and goroutines. I bring this up because there is one very important guarantee in the runtime: there will always be at least enough OS threads available to handle hosting every context. This allows the runtime to make an important optimization. The runtime also contains a thread pool for threads that aren’t currently being utilized
  - Scaling, efficiency, and simplicity. This is what makes goroutines so intriguing.

## Appendix

$ go test -race mypkg    # test the package
$ go run -race mysrc.go  # compile and run the program
$ go build -race mycmd   # build the command
$ go install -race mypkg # install the package

runtime/pprof
goroutine    - stack traces of all current goroutines
heap         - a sampling of all heap allocations
threadcreate - stack traces that led to the creation of new OS threads
block        - stack traces that led to blocking on synchronization primitives
mutex        - stack traces of holders of contended mutexes
