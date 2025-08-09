# Chapter 9 - Inference Optimization

- New models come and go, but one thing will always remain relevant: making them better, cheaper, and faster. Up until now, the book has discussed various techniques for making models better. This chapter focuses on making them faster and cheaper.
- Inference optimization can be done at the model, hardware, and service levels. At the model level, you can reduce a trained model’s size or develop more efficient architectures, such as one without the computation bottlenecks in the attention mechanism often used in transformer models. At the hardware level, you can design more powerful hardware.
- Because of this, inference optimization is an interdisciplinary field that often sees collaboration among model researchers, application developers, system engineers, compiler designers, hardware architects, and even data center operators.
- This chapter also covers performance metrics and trade-offs. Sometimes, a technique that speeds up a model can also reduce its cost. For example, reducing a model’s precision makes it smaller and faster. But often, optimization requires trade-offs. For example, the best hardware might make your model run faster but at a higher cost.
- Given the growing availability of open source models, more teams are building their own inference services. However, even if you don’t implement these inference optimization techniques, understanding these techniques will help you evaluate inference services and frameworks. If your application’s latency and cost are hurting you, read on. This chapter might help you diagnose the causes and potential solutions.

### Understanding Inference Optimization

- There are two distinct phases in an AI model’s lifecycle: training and inference. Training refers to the process of building a model. Inference refers to the process of using a model to compute an output for a given input.1 Unless you train or finetune a model, you’ll mostly need to care about inference.

Inference Overview

- In production, the component that runs model inference is called an inference server. It hosts the available models and has access to the necessary hardware. Based on requests from applications (e.g., user prompts), it allocates resources to execute the appropriate models and returns the responses to users.
- Model APIs like those provided by OpenAI and Google are inference services. If you use one of these services, you won’t be implementing most of the techniques discussed in this chapter. However, if you host a model yourself, you’ll be responsible for building, optimizing, and maintaining its inference service.

Computational bottlenecks

- Optimization is about identifying bottlenecks and addressing them.
- Similarly, an inference server should be designed to address the computational bottlenecks of the inference workloads it serves. There are two main computational bottlenecks, compute-bound and memory bandwidth-bound:
  - Compute-bound
    - This refers to tasks whose time-to-complete is determined by the computation needed for the tasks. For example, password decryption is typically compute-bound due to the intensive mathematical calculations required to break encryption algorithms.
  - Memory bandwidth-bound
    - These tasks are constrained by the data transfer rate within the system, such as the speed of data movement between memory and processors. For example, if you store your data in the CPU memory and train a model on GPUs, you have to move data from the CPU to the GPU, which can take a long time. This can be shortened as bandwidth-bound. In literature, memory bandwidth-bound is often referred to as memory-bound.
    - Memory-bound is also used by some people to refer to tasks whose time-to-complete is constrained by memory capacity instead of memory bandwidth. This occurs when your hardware doesn’t have sufficient memory to handle the task, for example, if your machine doesn’t have enough memory to store the entire internet. This memory is often manifested in the error recognizable by engineers everywhere: OOM, out-of-memory
- Different optimization techniques aim to mitigate different bottlenecks. For example, a compute-bound workload might be sped up by spreading it out to more chips or by leveraging chips with more computational power (e.g., a higher FLOP/s number). A memory bandwidth-bound workload might be sped up by leveraging chips with higher bandwidth.
- Different model architectures and workloads result in different computational bottlenecks. For example, inference for image generators like Stable Diffusion is typically compute-bound, whereas inference for autoregression language models is typically memory bandwidth-bound.
- inference for a transformer-based language model consists of two steps, prefilling and decoding:
  - Prefill
    - The model processes the input tokens in parallel.5 How many tokens can be processed at once is limited by the number of operations your hardware can execute in a given time. Therefore, prefilling is compute-bound.
  - Decode
    - The model generates one output token at a time. At a high level, this step typically involves loading large matrices (e.g., model weights) into GPUs, which is limited by how quickly your hardware can load data into memory. Decoding is, therefore, memory bandwidth-bound.
- Because prefill and decode have different computational profiles, they are often decoupled in production with separate machines.
- The factors that affect the amount of prefilling and decoding computation in an LLM inference server, and therefore its bottlenecks, include context length, output length, and request batching strategies. Long context typically results in a memory bandwidth-bound workload, but clever optimization techniques, such as those discussed later in this chapter, can remove this bottleneck.
- As of this writing, due to the prevalence of the transformer architecture and the limitations of the existing accelerator technologies, many AI and data workloads are memory bandwidth-bound. However, future software and hardware advancements will be able to make AI and data workloads compute-bound.

Online and batch inference APIs

- Online APIs optimize for latency. Requests are processed as soon as they arrive.
- Batch APIs optimize for cost. If your application doesn’t have strict latency requirements, you can send them to batch APIs for more efficient processing. Higher latency allows a broader range of optimization techniques, including batching requests together and using cheaper hardware. For example, as of this writing, both Google Gemini and OpenAI offer batch APIs at a 50% cost reduction and significantly higher turnaround time, i.e., in the order of hours instead of seconds or minutes.
- The only real difference is that an online API focuses on lower latency, whereas a batch API focuses on higher throughput.
- Customer-facing use cases, such as chatbots and code generation, typically require lower latency, and, therefore, tend to use online APIs. Use cases with less stringent latency requirements, which are ideal for batch APIs, include the following:
  - Synthetic data generation 
  - Periodic reporting, such as summarizing Slack messages, sentiment analysis of brand mentions on social media, and analyzing customer support tickets 
  - Onboarding new customers who require processing of all their uploaded documents 
  - Migrating to a new model that requires reprocessing of all the data 
  - Generating personalized recommendations or newsletters for a large customer base 
  - Knowledge base updates by reindexing an organization’s data
- Many online APIs offer streaming mode, which returns each token as it’s generated. This reduces the time the users have to wait until the first token. The downside of this approach is that you can’t score a response before showing it to users, increasing the risk of users seeing bad responses. However, you can still retrospectively update or remove a response as soon as the risk is detected.

Inference Performance Metrics

- Before jumping into optimization, it’s important to understand what metrics to optimize for. From the user perspective, the central axis is latency (response quality is a property of the model itself, not of the inference service). However, application developers must also consider throughput and utilization as they determine the cost of their applications.
- Latency, TTFT, and TPOT
- Latency measures the time from when users send a query until they receive the complete response. For autoregressive generation, especially in the streaming mode, the overall latency can be broken into several metrics:
  - Time to first token
    - TTFT measures how quickly the first token is generated after users send a query. It corresponds to the duration of the prefill step and depends on the input’s length
  - Time per output token
    - TPOT measures how quickly each output token is generated after the first token. If each token takes 100 ms, a response of 1,000 tokens will take 100 s. 
    - In the streaming mode, where users read each token as it’s generated, TPOT should be faster than human reading speed but doesn’t have to be much faster. A very fast reader can read 120 ms/token, so a TPOT of around 120 ms, or 6–8 tokens/second, is sufficient for most use cases.
  - Time between tokens and inter-token latency
    - Variations of this metric include time between tokens (TBT) and inter-token latency (ITL).9 Both measure the time between output tokens.
- The total latency will equal `TTFT + TPOT × (number of output tokens)`.
- Two applications with the same total latency can offer different user experiences with different TTFT and TPOT. Would your users prefer instant first tokens with a longer wait between tokens, or would they rather wait slightly longer for the first tokens but enjoy faster token generation afterward? User studies will be necessary to determine the optimal user experience. Reducing TTFT at the cost of higher TPOT is possible by shifting more compute instances from decoding to prefilling and vice versa.
- It’s important to note that the TTFT and TPOT values observed by users might differ from those observed by models, especially in scenarios involving CoT (chain-of-thought) or agentic queries where models generate intermediate steps not shown to users. Some teams use the metric time to publish to make it explicit that it measures time to the first token users see.
- It’s more helpful to look at latency in percentiles, as they tell you something about a certain percentage of your requests. The most common percentile is the 50th percentile, abbreviated as p50 (median)
- Typically, the percentiles you’ll want to look at are p90, p95, and p99. It’s also helpful to plot TTFT values against inputs’ lengths.

Throughput and goodput

- What’s considered good throughput depends on the model, the hardware, and the workload.
- Just like most other software applications, AI applications have the latency/throughput trade-off. Techniques like batching can improve throughput but reduce latency. According to the LinkedIn AI team in their reflection after a year of deploying generative AI products (LinkedIn, 2024), it’s not uncommon to double or triple the throughput if you’re willing to sacrifice TTFT and TPOT.
- Due to this trade-off, focusing on an inference service based solely on its throughput and cost can lead to a bad user experience. Instead, some teams focus on goodput, a metric adapted from networking for LLM applications. Goodput measures the number of requests per second that satisfies the SLO, software-level objective.

Utilization, MFU, and MBU

- Utilization metrics measure how efficiently a resource is being used. It typically quantifies the proportion of the resource actively being used compared to its total available capacity.
- A common but often misunderstood metric is GPU utilization, and NVIDIA is partially to blame for this misunderstanding.
- nvidia-smi’s GPU optimization metric is, therefore, not very useful. A utilization metric you might care about, out of all the operations a machine is capable of computing, is how many it’s doing in a given time. This metric is called MFU (Model FLOP/s Utilization), which distinguishes it from the NVIDIA GPU utilization metric.
- MFU is the ratio of the observed throughput (tokens/s) relative to the theoretical maximum throughput of a system operating at peak FLOP/s. If at the peak FLOP/s advertised by the chip maker, the chip can generate 100 tokens/s, but when used for your inference service, it can generate only 20 tokens/s, your MFU is 20%
- Utilization metrics are helpful to track your system’s efficiency. Higher utilization rates for similar workloads on the same hardware generally mean that your services are becoming more efficient. However, the goal isn’t to get the chips with the highest utilization. What you really care about is how to get your jobs done faster and cheaper. A higher utilization rate means nothing if the cost and latency both increase.

AI Accelerators

- The development of AI models and hardware has always been intertwined. The lack of sufficiently powerful computers was one of the contributing factors to the first AI winter in the 1970s
- Before GPUs, if you wanted to train a model at AlexNet’s scale, you’d have to use thousands of CPUs, like the one Google released just a few months before AlexNet. Compared to thousands of CPUs, a couple of GPUs were a lot more accessible to PhD students and researchers, setting off the deep learning research boom.

What’s an accelerator?

- An accelerator is a chip designed to accelerate a specific type of computational workload. An AI accelerator is designed for AI workloads. The dominant type of AI accelerator is GPUs, and the biggest economic driver during the AI boom in the early 2020s is undoubtedly NVIDIA.
- The main difference between CPUs and GPUs is that CPUs are designed for general-purpose usage, whereas GPUs are designed for parallel processing:
  - CPUs have a few powerful cores, typically up to 64 cores for high-end consumer machines. While many CPU cores can handle multi-threaded workloads effectively, they excel at tasks requiring high single-thread performance, such as running an operating system, managing I/O (input/output) operations, or handling complex, sequential processes. 
  - GPUs have thousands of smaller, less powerful cores optimized for tasks that can be broken down into many smaller, independent calculations, such as graphics rendering and machine learning. The operation that constitutes most ML workloads is matrix multiplication, which is highly parallelizable.15
- As discussed in Chapter7, training demands much more memory due to backpropagation and is generally more difficult to perform in lower precision. Furthermore, training usually emphasizes throughput, whereas inference aims to minimize latency.
- A chip’s specifications contain many details that can be useful when evaluating this chip for each specific use case. However, the main characteristics that matter across use cases are computational capabilities, memory size and bandwidth, and power consumption
- Computational capabilities
  - Computational capabilities are typically measured by the number of operations a chip can perform in a given time. The most common metric is FLOP/s, often written as FLOPS, which measures the peak number of floating-point operations per second. In reality, however, it’s very unlikely that an application can achieve this peak FLOP/s. The ratio between the actual FLOP/s and the theoretical FLOP/s is one utilization metric.
- Memory size and bandwidth
  - Because a GPU has many cores working in parallel, data often needs to be moved from the memory to these cores, and, therefore, data transfer speed is important. Data transfer is crucial when working with AI models that involve large weight matrices and training data. These large amounts of data need to be moved quickly to keep the cores efficiently occupied. Therefore, GPU memory needs to have higher bandwidth and lower latency than CPU memory, and thus, GPU memory requires more advanced memory technologies. This is one of the factors that makes GPU memory more expensive than CPU memory.
  - An accelerator’s memory is measured by its size and bandwidth. These numbers need to be evaluated within the system an accelerator is part of. An accelerator, such as a GPU, typically interacts with three levels of memory,
    - CPU memory (DRAM)
    - GPU high-bandwidth memory (HBM)
    - GPU on-chip SRAM
  - A lot of GPU optimization is about how to make the most out of this memory hierarchy. However, as of this writing, popular frameworks such as PyTorch and TensorFlow don’t yet allow fine-grained control of memory access. This has led many AI researchers and engineers to become interested in GPU programming languages such as CUDA (originally Compute Unified Device Architecture), OpenAI’s Triton, and ROCm (Radeon Open Compute). The latter is AMD’s open source alternative to NVIDIA’s proprietary CUDA.
- Power consumption
  - Chips rely on transistors to perform computation. Each computation is done by transistors switching on and off, which requires energy.
- When evaluating which chips to buy, there are three main questions:
  - Can the hardware run your workloads? 
  - How long does it take to do so? 
  - How much does it cost?
- FLOP/s, memory size, and memory bandwidth are the three big numbers that help you answer the first two questions. The last question is straightforward. Cloud providers’ pricing is typically usage-based and fairly similar across providers. If you buy your hardware, the cost can be calculated based on the initial price and ongoing power consumption.

### Inference Optimization

- Inference optimization can be done at the model, hardware, or service level.
- Model-level optimization is like crafting better arrows. Hardware-level optimization is like training a stronger and better archer. Service-level optimization is like refining the entire shooting process, including the bow and aiming conditions.
- Ideally, optimizing a model for speed and cost shouldn’t change the model’s quality. However, many techniques might cause model degradation

Model Optimization

- Model-level optimization aims to make the model more efficient, often by modifying the model itself, which can alter its behavior. As of this writing, many foundation models follow the transformer architecture and include an autoregressive language model component. These models have three characteristics that make inference resource-intensive: model size, autoregressive decoding, and the attention mechanism

Model compression

- Model compression involves techniques that reduce a model’s size. Making a model smaller can also make it faster. This book has already discussed two model compression techniques: quantization and distillation. Quantization, reducing the precision of a model to reduce its memory footprint and increase its throughput, is discussed in Chapter7. Model distillation, training a small model to mimic the behavior of the large model, is discussed in Chapter8.
- Model distillation suggests that it’s possible to capture a large model’s behaviors using fewer parameters. Could it be that within the large model, there exists a subset of parameters capable of capturing the entire model’s behavior? This is the core concept behind pruning.

Overcoming the autoregressive decoding bottleneck

- As discussed in Chapter2, autoregressive language models generate one token after another. If it takes 100 ms to generate one token, a response of 100 tokens will take 10 s.19 This process is not just slow, it’s also expensive
- Improving the autoregressive generation process by a small percentage can significantly improve user experience.
- As the space is rapidly evolving, new techniques are being developed to overcome this seemingly impossible bottleneck. Perhaps one day, there will be architectures that don’t have this bottleneck. The techniques covered here are to illustrate what the solution might look like, but the techniques are still evolving.
- Speculative decoding
  - Speculative decoding (also called speculative sampling) uses a faster but less powerful model to generate a sequence of tokens, which are then verified by the target model. The target model is the model you want to use. The faster model is called the draft or proposal model because it proposes the draft output.
- Inference with reference
  - Often, a response needs to reference tokens from the input. For example, if you ask your model a question about an attached document, the model might repeat a chunk of text verbatim from the document. Another example is if you ask the model to fix bugs in a piece of code, the model might reuse the majority of the original code with minor changes. Instead of making the model generate these repeated tokens, what if we copy these tokens from the input to speed up the generation? This is the core idea behind inference with reference.
  - Unlike speculative decoding, inference with reference doesn’t require an extra model. However, it’s useful only in generation scenarios where there’s a significant overlap between contexts and outputs, such as in retrieval systems, coding, or multi-turn conversations.
- Parallel decoding
  - Instead of making autoregressive generation faster with draft tokens, some techniques aim to break the sequential dependency. Given an existing sequence of tokens x1, x2,…,xt, these techniques attempt to generate xt + 1, xt + 2,…,xt + k simultaneously. This means that the model generates xt + 2 before it knows that the token before it is xt + 1. 
  - This can work because the knowledge of the existing sequence often is sufficient to predict the next few tokens. For example, given “the cat sits”, without knowing that the next token is “on”, “under”, or “behind”, you might still predict that the word after it is “the”.

Kernels and compilers

- Kernels are specialized pieces of code optimized for specific hardware accelerators, such as GPUs or TPUs. They are typically written to perform computationally intensive routines that need to be executed repeatedly, often in parallel, to maximize the performance of these accelerators.
- Common AI operations, including matrix multiplication, attention computation, and convolution operation, all have specialized kernels to make their computation more efficient on different hardware
- Writing kernels requires a deep understanding of the underlying hardware architecture. This includes knowledge about how the memory hierarchy is structured (such as caches, global memory, shared memory, and registers) and how data is accessed and moved between these different levels.
- Moreover, kernels are typically written in lower-level programming languages like CUDA (for NVIDIA GPUs), Triton (a language developed by OpenAI for writing custom kernels), and ROCm (for AMD GPUs). These languages allow fine-grained control over thread management and memory access but are also harder to learn than the languages that most AI engineers are familiar with, like Python.
- Due to this entry barrier, writing kernels used to be a dark art practiced by a few. Chip makers like NVIDIA and AMD employ optimization engineers to write kernels to make their hardware efficient for AI workloads, whereas AI frameworks like PyTorch and TensorFlow employ kernel engineers to optimize their frameworks on different accelerators.
- However, with the rising demand for inference optimization and the ubiquity of accelerators, more AI engineers have taken an interest in writing kernels. There are many great online tutorials for kernel writing. Here, I’ll cover four common techniques often used to speed up computation:
  - Vectorization
  - Parallelization
  - Loop tiling
  - Operator fusion

Inference Service Optimization

- Most service-level optimization techniques focus on resource management. Given a fixed amount of resources (compute and memory) and dynamic workloads (inference requests from users that may involve different models), the goal is to efficiently allocate resources to these workloads to optimize for latency and cost. Unlike many model-level techniques, service-level techniques don’t modify models and shouldn’t change the output quality.
- Batching
  - One of the easiest ways to reduce your cost is batching. In production, your inference service might receive multiple requests simultaneously. Instead of processing each request separately, batching the requests that arrive around the same time together can significantly reduce the service’s throughput. If processing each request separately is like everyone driving their own car, batching is like putting them together on a bus. A bus can move more people, but it can also make each person’s journey longer. However, if you do it intelligently, the impact on latency can be minimal.
  - The three main techniques for batching are: static batching, dynamic batching, and continuous batching.
  - The simplest batching technique is static batching. The service groups a fixed number of inputs together in a batch. It’s like a bus that waits until every seat is filled before departing. The drawback of static batching is that all requests have to wait until the batch is full to be executed. Thus the first request in a batch is delayed until the batch’s last request arrives, no matter how late the last request is.
  - Dynamic batching, on the other hand, sets a maximum time window for each batch. If the batch size is four and the window is 100 ms, the server processes the batch either when it has four requests or when 100 ms has passed, whichever happens first. It’s like a bus that leaves on a fixed schedule or when it’s full. This approach keeps latency under control, so earlier requests aren’t held up by later ones. The downside is that batches may not always be full when processed, possibly leading to wasted compute
  - In naive batching implementations, all batch requests have to be completed before their responses are returned. For LLMs, some requests might take much longer than others. If one request in a batch generates only 10 response tokens and another request generates 1,000 response tokens, the short response has to wait until the long response is completed before being returned to the user. This results in unnecessary latency for short requests.
  - Continuous batching allows responses in a batch to be returned to users as soon as they are completed. It works by selectively batching operations that don’t cause the generation of one response to hold up another
  - After a request in a batch is completed and its response returned, the service can add another request into the batch in its place, making the batching continuous. It’s like a bus that, after dropping off one passenger, can immediately pick up another passenger to maximize its occupancy rate
- Decoupling prefill and decode
  - LLM inference consists of two steps: prefill and decode. Because prefill is compute-bound and decode is memory bandwidth-bound, using the same machine to perform both can cause them to inefficiently compete for resources and significantly slow down both TTFT and TPOT.
  - One common optimization technique for inference servers is to disaggregate prefill and decode
  - Even though decoupling requires transferring intermediate states from prefill instances to decode instances, the paper shows communication overhead is not substantial in modern GPU clusters with high-bandwidth connections such as NVLink within a node.
- Prompt caching
  - Many prompts in an application have overlapping text segments. A prompt cache stores these overlapping segments for reuse, so you only need to process them once. A common overlapping text segment in different prompts is the system prompt. Without a prompt cache, your model needs to process the system prompt with every query. With a prompt cache, the system prompt needs to be processed just once for the first query.
  - Prompt caching is useful for queries that involve long documents. For example, if many of your user queries are related to the same long document (such as a book or a codebase), this long document can be cached for reuse across queries. It’s also useful for long conversations when the processing of earlier messages can be cached and reused when predicting future messages.
  - For applications with long system prompts, prompt caching can significantly reduce both latency and cost. If your system prompt is 1,000 tokens, and your application generates one million model API calls daily, a prompt cache will save you from processing approximately one billion repetitive input tokens a day! However, this isn’t entirely free. Like the KV cache, prompt cache size can be quite large and take up memory space. Unless you use a model API with this functionality, implementing prompt caching can require significant engineering effort.
- Parallelism
  - Accelerators are designed for parallel processing, and parallelism strategies are the backbone of high-performance computing. Many new parallelization strategies are being developed. This section covers only a few of them for reference.
  - Two families of parallelization strategies that can be applied across all models are data parallelism and model parallelism. A family of strategies applied specifically for LLMs is context and sequence parallelism. An optimization technique might involve multiple parallelism strategies.
  - Replica parallelism is the most straightforward strategy to implement. It simply creates multiple replicas of the model you want to serve.27 More replicas allow you to handle more requests at the same time, potentially at the cost of using more chips. 
  - Often, your model is so big that it can’t fit into one machine. Model parallelism refers to the practice of splitting the same model across multiple machines. Fitting models onto chips can become an even more complicated problem with model parallelism.
  - There are several ways to split a model. The most common approach for inference is tensor parallelism, also known as intra-operator parallelism.
  - Tensor parallelism provides two benefits. First, it makes it possible to serve large models that don’t fit on single machines. Second, it reduces latency. The latency benefit, however, might be reduced due to extra communication overhead.
  - Another way to split a model is pipeline parallelism, which involves dividing a model’s computation into distinct stages and assigning each stage to a different device. As data flows through the model, each stage processes one part while others process subsequent parts, enabling overlapping computations
  - Two techniques that are less common but might warrant a quick mention to illustrate the diversity of techniques are context parallelism and sequence parallelism. They were both developed to make long input sequence processing more efficient, including context parallelism and sequence parallelism.

### Summary

- A model’s usability depends heavily on its inference cost and latency. Cheaper inference makes AI-powered decisions more affordable, while faster inference enables the integration of AI into more applications. Given the massive potential impact of inference optimization, it has attracted many talented individuals who continually come up with innovative approaches.
- This chapter started with common efficiency metrics for latency, throughput, and utilization. For language model-based inference, latency can be broken into time to first token (TTFT), which is influenced by the prefilling phase, and time per output token (TPOT), which is influenced by the decoding phase. Throughput metrics are directly related to cost. There’s a trade-off between latency and throughput. You can potentially reduce cost if you’re okay with increased latency, and reducing latency often involves increasing cost.

