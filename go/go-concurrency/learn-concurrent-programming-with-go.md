# Book: Learn Concurrent Programming with Go by James Cutajar

(https://learning.oreilly.com/library/view/learn-concurrent-programming/9781633438385)


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
- 