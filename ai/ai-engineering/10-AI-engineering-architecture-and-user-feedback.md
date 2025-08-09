# Chapter 10 - AI Engineering Architecture and User Feedback

- So far, this book has covered a wide range of techniques to adapt foundation models to specific applications. This chapter will discuss how to bring these techniques together to build successful products.
- To simplify this process, this chapter takes a gradual approach. It starts with the simplest architecture for a foundation model application, highlights the challenges of that architecture, and gradually adds components to address them.
- We can spend eternity reasoning about how to build a successful application, but the only way to find out if an application actually achieves its goal is to put it in front of users. 
- User feedback has always been invaluable for guiding product development, but for AI applications, user feedback has an even more crucial role as a data source for improving models. The conversational interface makes it easier for users to give feedback but harder for developers to extract signals. This chapter will discuss different types of conversational AI feedback and how to design a system to collect the right feedback without hurting user experience.

### AI Engineering Architecture

- This section follows the process that a team might follow in production, starting with the simplest architecture and progressively adding more components
- Despite the diversity of AI applications, they share many common components. The architecture proposed here has been validated at multiple companies to be general for a wide range of applications, but certain applications might deviate.
- In its simplest form, your application receives a query and sends it to the model. The model generates a response, which is returned to the user
- From this simple architecture, you can add more components as needs arise. The process might look as follows:
  - Enhance context input into a model by giving the model access to external data sources and tools for information gathering. 
  - Put in guardrails to protect your system and your users. 
  - Add model router and gateway to support complex pipelines and add more security. 
  - Optimize for latency and costs with caching. 
  - Add complex logic and write actions to maximize your system’s capabilities.
- Monitoring and observability, which are integral to any application for quality control and performance improvement, will be discussed at the end of this process. Orchestration, chaining all these components together, will be discussed after that.

Step 1. Enhance Context

- The initial expansion of a platform usually involves adding mechanisms to allow the system to construct the relevant context needed by the model to answer each query. 
- Context can also be augmented using tools that allow the model to automatically gather information through APIs such as web search, news, weather, events, etc.
- Context construction is like feature engineering for foundation models. It gives the model the necessary information to produce an output. Due to its central role in a system’s output quality, context construction is almost universally supported by model API providers. For example, providers like OpenAI, Claude, and Gemini allow users to upload files and allow their models to use tools.

Step 2. Put in Guardrails

- Guardrails help mitigate risks and protect you and your users. They should be placed whenever there are exposures to risks. In general, they can be categorized into guardrails around inputs and outputs.
- Input guardrails typically protect against two types of risks: leaking private information to external APIs and executing bad prompts that compromise your system.
- There’s no airtight way to eliminate potential leaks when using third-party APIs. However, you can mitigate them with guardrails. You can use one of the many available tools that automatically detect sensitive data. What sensitive data to detect is specified by you
- Many sensitive data detection tools use AI to identify potentially sensitive information, such as determining if a string resembles a valid home address. If a query is found to contain sensitive information, you have two options: block the entire query or remove the sensitive information from it. For instance, you can mask a user’s phone number with the placeholder [PHONE NUMBER]
- Output guardrails have two main functions:
  - Catch output failures 
  - Specify the policy to handle different failure modes
- To catch outputs that fail to meet your standards, you need to understand what failures look like. The easiest failure to detect is when a model returns an empty response when it shouldn’t.2 Failures look different for different applications. Here are some common failures in the two main categories: quality and security.
  - Quality 
    - Malformatted responses that don’t follow the expected output format. For example, the application expects JSON, and the model generates invalid JSON. 
    - Factually inconsistent responses hallucinated by the model. 
    - Generally bad responses. For example, you ask the model to write an essay, and that essay is just bad.
  - Security 
    - Toxic responses that contain racist content, sexual content, or illegal activities. 
    - Responses that contain private and sensitive information. 
    - Responses that trigger remote tool and code execution. 
    - Brand-risk responses that mischaracterize your company or your competitors.
- Many failures can be mitigated by simple retry logic. AI models are probabilistic, which means that if you try a query again, you might get a different response. For example, if the response is empty, try again X times or until you get a nonempty response. Similarly, if the response is malformatted, try again until the response is correctly formatted.
- This retry policy, however, can incur extra latency and cost. Each retry means another round of API calls. If the retry is carried out after failure, the user-perceived latency will double. To reduce latency, you can make calls in parallel. For example, for each query, instead of waiting for the first query to fail before retrying, you send this query to the model twice at the same time, get back two responses, and pick the better one. This increases the number of redundant API calls while keeping latency manageable.
- It’s also common to fall back on humans for tricky requests. For example, you can transfer the queries that contain specific phrases to human operators. Some teams use a specialized model to decide when to transfer a conversation to humans. One team, for instance, transfers a conversation to human operators when their sentiment analysis model detects anger in users’ messages. Another team transfers a conversation after a certain number of turns to prevent users from getting stuck in a loop.
- Guardrails come with trade-offs. One is the reliability versus latency trade-off. While acknowledging the importance of guardrails, some teams told me that latency is more important. The teams decided not to implement guardrails because they can significantly increase the application’s latency
- Output guardrails might not work well in the stream completion mode. By default, the whole response is generated before being shown to the user, which can take a long time. In the stream completion mode, new tokens are streamed to the user as they are generated, reducing the time the user has to wait to see the response. The downside is that it’s hard to evaluate partial responses, so unsafe responses might be streamed to users before the system guardrails can determine that they should be blocked.
- Given the many different places where an application might fail, guardrails can be implemented at many different levels. Model providers give their models guardrails to make their models better and more secure. However, model providers have to balance safety and flexibility. Restrictions might make a model safer but can also make it less usable for specific use cases.

Step 3. Add Model Router and Gateway

- As applications grow to involve more models, routers and gateways emerge to help you manage the complexity and costs of serving multiple models.

Router
- Instead of using one model for all queries, you can have different solutions for different types of queries. This approach has several benefits. First, it allows specialized models, which can potentially perform better than a general-purpose model for specific queries. For example, you can have one model specialized in technical troubleshooting and another specialized in billing. Second, this can help you save costs. Instead of using one expensive model for all queries, you can route simpler queries to cheaper models.
- A router typically consists of an intent classifier that predicts what the user is trying to do. Based on the predicted intent, the query is routed to the appropriate solution
- An intent classifier can prevent your system from engaging in out-of-scope conversations. If the query is deemed inappropriate, the chatbot can politely decline to respond using one of the stock responses without wasting an API call.
- An intent classifier can help the system detect ambiguous queries and ask for clarification. For example, in response to the query “Freezing”, the system might ask, “Do you want to freeze your account or are you talking about the weather?” or simply ask, “I’m sorry. Can you elaborate?”
- Other routers can aid the model in deciding what to do next. For example, for an agent capable of multiple actions, a router can take the form of a next-action predictor: should the model use a code interpreter or a search API next? For a model with a memory system, a router can predict which part of the memory hierarchy the model should pull information from
- Intent classifiers and next-action predictors can be implemented on top of foundation models. Many teams adapt smaller language models like GPT-2, BERT, and Llama 7B as their intent classifiers. Many teams opt to train even smaller classifiers from scratch. Routers should be fast and cheap so that they can use multiples of them without incurring significant extra latency and cost.
- Grouping routers together with other models makes models easier to manage. However, it’s important to note that routing often happens before retrieval. For example, before retrieval, a router can help determine if a query is in-scope and, if yes, if it needs retrieval. Routing can happen after retrieval, too, such as determining if a query should be routed to a human operator. However, routing - retrieval - generation - scoring is a much more common AI application pattern.

Gateway
- A model gateway is an intermediate layer that allows your organization to interface with different models in a unified and secure manner. The most basic functionality of a model gateway is to provide a unified interface to different models, including self-hosted models and models behind commercial APIs. A model gateway makes it easier to maintain your code. If a model API changes, you only need to update the gateway instead of updating all applications that depend on this API
- A model gateway provides access control and cost management. Instead of giving everyone who wants access to the OpenAI API your organizational tokens, which can be easily leaked, you give people access only to the model gateway, creating a centralized and controlled point of access. The gateway can also implement fine-grained access controls, specifying which user or application should have access to which model. Moreover, the gateway can monitor and limit the usage of API calls, preventing abuse and managing costs effectively.
- When the primary API is unavailable, the gateway can route requests to alternative models, retry after a short wait, or handle failures gracefully in other ways. This ensures that your application can operate smoothly without interruptions.

Step 4. Reduce Latency with Caches

- Caching has long been integral to software applications to reduce latency and cost. Many ideas from software caching can be used for AI applications. Inference caching techniques, including KV caching and prompt caching, are discussed in Chapter9. This section focuses on system caching.
- In general, there are two major system caching mechanisms: exact caching and semantic caching.

Exact caching
- With exact caching, cached items are used only when these exact items are requested. For example, if a user asks a model to summarize a product, the system checks the cache to see if a summary of this exact product exists. If yes, fetch this summary. If not, summarize the product and cache the summary.
- Exact caching is also used for embedding-based retrieval to avoid redundant vector search. If an incoming query is already in the vector search cache, fetch the cached result. If not, perform a vector search for this query and cache the result.
- An exact cache can be implemented using in-memory storage for fast retrieval. However, since in-memory storage is limited, a cache can also be implemented using databases like PostgreSQL, Redis, or tiered storage to balance speed and storage capacity. Having an eviction policy is crucial to manage the cache size and maintain performance. Common eviction policies include Least Recently Used (LRU), Least Frequently Used (LFU), and first in, first out (FIFO).

Semantic caching
- Unlike in exact caching, cached items are used even if they are only semantically similar, not identical, to the incoming query. Imagine one user asks, “What’s the capital of Vietnam?” and the model answers, “Hanoi”. Later, another user asks, “What’s the capital city of Vietnam?”, which is semantically the same question but with slightly different wording. With semantic caching, the system can reuse the answer from the first query instead of computing the new query from scratch.
- Reusing similar queries increases the cache’s hit rate and potentially reduces cost. However, semantic caching can reduce your model’s performance.
- Semantic caching works only if you have a reliable way of determining if two queries are similar. One common approach is to use semantic similarity, as discussed in Chapter3.
- This approach requires a vector database to store the embeddings of cached queries.
- Compared to other caching techniques, semantic caching’s value is more dubious because many of its components are prone to failure. Its success relies on high-quality embeddings, functional vector search, and a reliable similarity metric.
- In addition, semantic cache can be time-consuming and compute-intensive, as it involves a vector search. The speed and cost of this vector search depend on the size of your cached embeddings.
- Semantic cache might still be worthwhile if the cache hit rate is high, meaning that a good portion of queries can be effectively answered by leveraging the cached results. However, before incorporating the complexities of a semantic cache, make sure to evaluate the associated efficiency, cost, and performance risks.

Step 5. Add Agent Patterns

- The applications discussed so far are still fairly simple. Each query follows a sequential flow. However, as discussed in Chapter6, an application flow can be more complex with loops, parallel execution, and conditional branching. Agentic patterns, discussed in Chapter6, can help you build complex applications.
- For example, after the system generates an output, it might determine that it hasn’t accomplished the task and that it needs to perform another retrieval to gather more information. The original response, together with the newly retrieved context, is passed into the same model or a different one.
- If you’ve followed all the steps so far, your architecture has likely grown quite complex. While complex systems can solve more tasks, they also introduce more failure modes, making them harder to debug due to the many potential points of failure. The next section will cover best practices for improving system observability.

Monitoring and Observability

- Even though I put observability in its own section, observability should be integral to the design of a product, rather than an afterthought. The more complex a product, the more crucial observability is.
- Observability is a universal practice across all software engineering disciplines. It’s a big industry with established best practices and many ready-to-use proprietary and open source solutions.4 To avoid reinventing the wheel, I’ll focus on what’s unique to applications built on top of foundation models
- The goal of monitoring is the same as the goal of evaluation: to mitigate risks and discover opportunities. Risks that monitoring should help you mitigate include application failures, security attacks, and drifts. Monitoring can help discover opportunities for application improvement and cost savings. Monitoring can also help keep you accountable by giving visibility into your system’s performance.
- Three metrics can help evaluate the quality of your system’s observability, derived from the DevOps community:
  - MTTD (mean time to detection): When something bad happens, how long does it take to detect it? 
  - MTTR (mean time to response): After detection, how long does it take to be resolved? 
  - CFR (change failure rate): The percentage of changes or deployments that result in failures requiring fixes or rollbacks. If you don’t know your CFR, it’s time to redesign your platform to make it more observable.
- Evaluation and monitoring need to work closely together. Evaluation metrics should translate well to monitoring metrics, meaning that a model that does well during evaluation should also do well during monitoring. Issues detected during monitoring should be fed to the evaluation pipeline.
- Since the mid-2010s, the industry has embraced the term “observability” instead of “monitoring.” Monitoring makes no assumption about the relationship between the internal state of a system and its outputs.
  - Observability, on the other hand, makes an assumption stronger than traditional monitoring: that a system’s internal states can be inferred from knowledge of its external outputs. When something goes wrong with an observable system, we should be able to figure out what went wrong by looking at the system’s logs and metrics without having to ship new code to the system
  - Observability is about instrumenting your system in a way that ensures that sufficient information about a system’s runtime is collected and analyzed so that when something goes wrong, it can help you figure out what goes wrong.

Metrics
- The purpose of a metric is to tell you when something is wrong and to identify opportunities for improvement.
- Before listing what metrics to track, it’s important to understand what failure modes you want to catch and design your metrics around these failures. For example, if you don’t want your application to hallucinate, design metrics that help you detect hallucinations. One relevant metric might be whether an application’s output can be inferred from the context.
- Because foundation models can generate open-ended outputs, there are many ways things can go wrong. Metrics design requires analytical thinking, statistical knowledge, and, often, creativity. Which metrics you should track are highly application-specific.
- The easiest types of failures to track are format failures because they are easy to notice and verify. For example, if you expect JSON outputs, track how often the model outputs invalid JSON and, among these invalid JSON outputs, how many can be easily fixed (missing a closing bracket is easy to fix, but missing expected keys is harder).
- 




### User Feedback


### Summary