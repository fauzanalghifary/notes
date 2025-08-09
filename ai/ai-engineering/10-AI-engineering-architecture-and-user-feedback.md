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
- For open-ended generations, consider monitoring factual consistency and relevant generation quality metrics such as conciseness, creativity, or positivity. Many of these metrics can be computed using AI judges.
- Model quality can also be inferred through user natural language feedback and conversational signals. For example, some easy metrics you can track include the following:
  - How often do users stop a generation halfway? 
  - What’s the average number of turns per conversation? 
  - What’s the average number of tokens per input? Are users using your application for more complex tasks, or are they learning to be more concise with their prompts? 
  - What’s the average number of tokens per output? Are some models more verbose than others? Are certain types of queries more likely to result in lengthy answers? 
  - What’s the model’s output token distribution? How has it changed over time? Is the model getting more or less diverse?
- Length-related metrics are also important for tracking latency and costs, as longer contexts and responses typically increase latency and incur higher costs.
- Each component in an application pipeline has its own metrics. For example, in a RAG application, the retrieval quality is often evaluated using context relevance and context precision. A vector database can be evaluated by how much storage it needs to index the data and how long it takes to query the data.
- Given that you’ll likely have multiple metrics, it’s useful to measure how these metrics correlate to each other and, especially, to your business north star metrics, which can be DAU (daily active user), session duration (the length of time a user spends actively engaged with the application), or subscriptions. Metrics that are strongly correlated to your north star might give you ideas on how to improve your north star. Metrics that are not at all correlated might also give you ideas on what not to optimize for.
- Tracking latency is essential for understanding the user experience
- You’ll also want to track costs. Cost-related metrics are the number of queries and the volume of input and output tokens, such as tokens per second (TPS). If you use an API with rate limits, tracking the number of requests per second is important to ensure you stay within your allocated limits and avoid potential service interruptions.
- When calculating metrics, you can choose between spot checks and exhaustive checks. Spot checks involve sampling a subset of data to quickly identify issues, while exhaustive checks evaluate every request for a comprehensive performance view. The choice depends on your system’s requirements and available resources, with a combination of both providing a balanced monitoring strategy.
- When computing metrics, ensure they can be broken down by relevant axes, such as users, releases, prompt/chain versions, prompt/chain types, and time. This granularity helps in understanding performance variations and identifying specific issues.

Logs and traces
- Metrics are typically aggregated. They condense information from events that occur in your system over time. They help you understand, at a glance, how your system is doing. However, there are many questions that metrics can’t help you answer. For example, after seeing a spike in a specific activity, you might wonder: “Has this happened before?” Logs can help you answer this question.
- If metrics are numerical measurements representing attributes and events, logs are an append-only record of events. In production, a debugging process might look like this:
  - Metrics tell you something went wrong five minutes ago, but they don’t tell you what happened. 
  - You look at the logs of events that took place around five minutes ago to figure out what happened. 
  - Correlate the errors in the logs to the metrics to make sure that you’ve identified the right issue.
- For fast detection, metrics need to be computed quickly. For fast response, logs need to be readily available and accessible. If your logs are 15 minutes delayed, you will have to wait for the logs to arrive to track down an issue that happened 5 minutes ago.
- Because you don’t know exactly what logs you’ll need to look at in the future, the general rule for logging is to log everything. Log all the configurations, including the model API endpoint, model name, sampling settings (temperature, top-p, top-k, stopping condition, etc.), and the prompt template.
- Log the user query, the final prompt sent to the model, the output, and the intermediate outputs. Log if it calls any tool. Log the tool outputs. Log when a component starts, ends, when something crashes, etc. When recording a piece of log, make sure to give it tags and IDs that can help you know where this log comes from in the system.
- Logging everything means that the amount of logs you have can grow very quickly. Many tools for automated log analysis and log anomaly detection are powered by AI.
- If logs are a series of disjointed events, traces are reconstructed by linking related events together to form a complete timeline of a transaction or process, showing how each step connects from start to finish. In short, a trace is the detailed recording of a request’s execution path through various system components and services.
- In an AI application, tracing reveals the entire process from when a user sends a query to when the final response is returned, including the actions the system takes, the documents retrieved, and the final prompt sent to the model. It should also show how much time each step takes and its associated cost, if measurable
- Ideally, you should be able to trace each query’s transformation step-by-step through the system. If a query fails, you should be able to pinpoint the exact step where it went wrong: whether it was incorrectly processed, the retrieved context was irrelevant, or the model generated a wrong response.

Drift detection
- The more parts a system has, the more things that can change. In an AI application these can be:
  - System prompt changes
    - A simple logic should be sufficient to catch when your application’s system prompt changes.
  - User behavior changes
    - Over time, users adapt their behaviors to the technology. For example, people have already figured out how to frame their queries to get better results on Google Search or how to make their articles rank higher on search results.
    - It’s likely that your users will change their behaviors to get better results out of your application. For example, your users might learn to write instructions to make the responses more concise. This might cause a gradual drop in response length over time. If you look only at metrics, it might not be obvious what caused this gradual drop. You need investigations to understand the root cause.
  - Underlying model changes
    - When using a model through an API, it’s possible that the API remains unchanged while the underlying model is updated

AI Pipeline Orchestration

- An AI application can get fairly complex, consisting of multiple models, retrieving data from many databases, and having access to a wide range of tools. An orchestrator helps you specify how these different components work together to create an end-to-end pipeline. It ensures that data flows seamlessly between components. At a high level, an orchestrator operates in two steps, components definition and chaining:
  - Components definition
    - You need to tell the orchestrator what components your system uses, including different models, external data sources for retrieval, and tools that your system can use
    - You can also tell the orchestrator if you use any tools for evaluation and monitoring.
  - Chaining
    - Chaining is basically function composition: it combines different functions (components) together. In chaining (pipelining), you tell the orchestrator the steps your system takes from receiving the user query until completing the task. Here’s an example of the steps:
      - Process the raw query. 
      - Retrieve the relevant data based on the processed query. 
      - Combine the original query and the retrieved data to create a prompt in the format expected by the model. 
      - The model generates a response based on the prompt. 
      - Evaluate the response. 
      - If the response is considered good, return it to the user. If not, route the query to a human operator.
- The orchestrator is responsible for passing data between components. It should provide toolings that help ensure that the output from the current step is in the format expected by the next step. Ideally, it should notify you when this data flow is disrupted due to errors such as component failures or data mismatch failures.
- An AI pipeline orchestrator is different from a general workflow orchestrator, like Airflow or Metaflow.
- When designing the pipeline for an application with strict latency requirements, try to do as much in parallel as possible
- There are many AI orchestration tools, including LangChain, LlamaIndex, Flowise, Langflow, and Haystack. Because retrieval and tool use are common application patterns, many RAG and agent frameworks are also orchestration tools.
- While it’s tempting to jump straight to an orchestration tool when starting a project, you might want to start building your application without one first. Any external tool brings additional complexity. An orchestrator can abstract away critical details of how your system works, making it hard to understand and debug your system.
- As you advance to the later stages of your application development process, you might decide that an orchestrator can make your life easier. Here are three aspects to keep in mind when evaluating orchestrators:
  - Integration and extensibility
    - Evaluate whether the orchestrator supports the components you’re already using or might adopt in the future.
    - Given how many models, databases, and frameworks there are, it’s impossible for an orchestrator to support everything. Therefore, you’ll also need to consider an orchestrator’s extensibility. If it doesn’t support a specific component, how hard is it to change that?
  - Support for complex pipelines
    - As your applications grow in complexity, you might need to manage intricate pipelines involving multiple steps and conditional logic. An orchestrator that supports advanced features like branching, parallel processing, and error handling will help you manage these complexities efficiently.
  - Ease of use, performance, and scalability
    - Look for intuitive APIs, comprehensive documentation, and strong community support, as these can significantly reduce the learning curve for you and your team. Avoid orchestrators that initiate hidden API calls or introduce latency to your applications. Additionally, ensure that the orchestrator can scale effectively as the number of applications, developers, and traffic grows.

### User Feedback

- User feedback has always played a critical role in software applications in two key ways: evaluating the application’s performance and informing its development. 
- However, in AI applications, user feedback takes on an even more significant role. User feedback is proprietary data, and data is a competitive advantage. A well-designed user feedback system is necessary to create the data flywheel discussed in Chapter8.
- User feedback can be used not only to personalize models for individual users but also to train future iterations of the models. As data becomes increasingly scarce, proprietary data is more valuable than ever. A product that launches quickly and attracts users early can gather data to continually improve models, making it difficult for competitors to catch up.

Extracting Conversational Feedback

- Traditionally, feedback can be explicit or implicit. Explicit feedback is information users provide in response to explicit requests for feedback in the application, such as thumbs up/thumbs down, upvote/downvote, star rating, or a yes/no answer to the question
- Implicit feedback is information inferred from user actions. For example, if someone buys a product recommended to them, it means it was a good recommendation. What can be considered implicit feedback depends on what actions a user can do within each application and is, therefore, highly application-dependent. Foundation models enable a new world of applications and, with them, many genres of implicit feedback.
- The conversational interface that many AI applications use makes it easier for users to give feedback. Users can encourage good behaviors and correct errors the same way they would give feedback in daily dialogues. The language that a user uses to give directions to AI can convey feedback about both the application’s performance and the user’s preference.
- User feedback, extracted from conversations, can be used for evaluation, development, and personalization:
  - Evaluation: derive metrics to monitor the application 
  - Development: train the future models or guide their development 
  - Personalization: personalize the application to each user
- Implicit conversational feedback can be inferred from both the content of user messages and their patterns of communication. Because feedback is blended into daily conversations, it’s also challenging to extract. While intuition about conversational cues can help you devise an initial set of signals to look for, rigorous data analysis and user studies are necessary to understand.
- While conversational feedback has enjoyed greater attention thanks to the popularity of conversational bots, it had been an active research area for several years before ChatGPT came out. The reinforcement learning community has been trying to get RL algorithms to learn from natural language feedback since the late 2010s

Natural language feedback

- Feedback extracted from the content of messages is called natural language feedback. Here are a couple of natural language feedback signals that tell you how a conversation is going. It’s useful to track these signals in production to monitor your application’s performance.
- Early termination
  - If a user terminates a response early, e.g., stopping a response generation halfway, exiting the app (for web and mobile apps), telling the model to stop (for voice assistants), or simply leaving the agent hanging (e.g., not responding to the agent with which option you want it to go ahead with), it’s likely that the conversation isn’t going well.
- Error correction
  - If a user starts their follow-up with “No, …” or “I meant, …”, the model’s response is likely off the mark.
  - Users can also point out specific things the model should’ve done differently.
  - This kind of action-correcting feedback is especially common for agentic use cases where users might nudge the agent toward more optional actions
- Complaints
  - Often, users just complain about your application’s outputs without trying to correct them. For example, they might complain that an answer is wrong, irrelevant, toxic, lengthy, lacking detail, or just bad
  - Understanding how the bot fails the user is crucial in making it better. For example, if you know that the user doesn’t like verbose answers, you can change the bot’s prompt to make it more concise. If the user is unhappy because the answer lacks details, you can prompt the bot to be more specific.
- Sentiment
  - Complaints can also be general expressions of negative sentiments (frustration, disappointment, ridicule, etc.) without explaining the reason why, such as “Uggh”. This might sound dystopian, but analysis of a user’s sentiments throughout conversations with a bot might give you insights into how the bot is doing
  - Natural language feedback can also be inferred from the model’s responses. One important signal is the model’s refusal rate. If a model says things like “Sorry, I don’t know that one” or “As a language model, I can’t do …”, the user is probably unhappy.

Other conversational feedback

- Regeneration
  - Many applications let users generate another response, sometimes with a different model. If a user chooses regeneration, it might be because they’re not satisfied with the first response. However, it might also be that the first response is adequate, but the user wants options to compare. This is especially common with creative requests like image or story generation.
  - Personally, I often choose regeneration for complex requests to ensure the model’s responses are consistent. If two responses give contradicting answers, I can’t trust either.
- Conversation organization
  - The actions a user takes to organize their conversations—such as delete, rename, share, and bookmark—can also be signals. Deleting a conversation is a pretty strong signal that the conversation is bad, unless it’s an embarrassing conversation and the user wants to remove its trace. Renaming a conversation suggests that the conversation is good, but the auto-generated title is bad.
- Conversation length
  - Another commonly tracked signal is the number of turns per conversation. Whether this is a positive or negative signal depends on the application. For AI companions, a long conversation might indicate that the user enjoys the conversation. However, for chatbots geared toward productivity like customer support, a long conversation might indicate that the bot is inefficient in helping users resolve their issues.
- Dialogue diversity
  - Conversation length can also be interpreted together with dialogue diversity, which can be measured by the distinct token or topic count. For example, if the conversation is long but the bot keeps repeating a few lines, the user might be stuck in a loop.
- Explicit feedback is easier to interpret, but it demands extra effort from users. Since many users may not be willing to put in this additional work, explicit feedback can be sparse, especially in applications with smaller user bases. Explicit feedback also suffers from response biases. For example, unhappy users might be more likely to complain, causing the feedback to appear more negative than it is.
- Implicit feedback is more abundant—what can be considered implicit feedback is limited only by your imagination—but it’s noisier. Interpreting implicit signals can be challenging
- It’s important to study your users to understand why they do each action.

Feedback Design

When to collect feedback
- Feedback can and should be collected throughout the user journey. Users should have the option to give feedback, especially to report errors, whenever this need arises. The feedback collection option, however, should be nonintrusive. It shouldn’t interfere with the user workflow.
- In the beginning
  - When a user has just signed up, user feedback can help calibrate the application for the user
  - For other applications, however, initial feedback should be optional, as it creates friction for users to try out your product. If a user doesn’t specify their preference, you can fall back to a neutral option and calibrate over time.
- When something bad happens
  - When the model hallucinates a response, blocks a legitimate request, generates a compromising image, or takes too long to respond, users should be able to notify you of these failures. You can give users the option to downvote a response, regenerate with the same model, or change to another model. Users might just give conversational feedback like “You’re wrong”, “Too cliche”, or “I want something shorter”.
  - Ideally, when your product makes mistakes, users should still be able to accomplish their tasks. For example, if the model wrongly categorizes a product, users can edit the category. Let users collaborate with the AI. If that doesn’t work, let them collaborate with humans. Many customer support bots offer to transfer users to human agents if the conversation drags on or if users seem frustrated.
  - An example of human–AI collaboration is the inpainting functionality for image generation.9 If a generated image isn’t exactly what the user needs, they can select a region of the image and describe with a prompt how to make it better.
- When the model has low confidence
  - When a model is uncertain about an action, you can ask the user for feedback to increase its confidence. For example, given a request to summarize a paper, if the model is uncertain whether the user would prefer a short, high-level summary or a detailed section-by-section summary, the model can output both summaries side by side, assuming that generating two summaries doesn’t increase the latency for the user. The user can choose which one they prefer. Comparative signals like this can be used for preference finetuning
  - You might wonder: how about feedback when something good happens? Actions that users can take to express their satisfaction include thumbs up, favoriting, or sharing. However, Apple’s human interface guideline warns against asking for both positive and negative feedback. Your application should produce good results by default. Asking for feedback on good results might give users the impression that good results are exceptions. Ultimately, if users are happy, they continue using your application.
  - However, many people I’ve talked to believe users should have the option to give feedback when they encounter something amazing. A product manager for a popular AI-powered product mentioned that their team needs positive feedback because it reveals the features users love enough to give enthusiastic feedback about. This allows the team to concentrate on refining a small set of high-impact features rather than spreading resources across many with minimal added value.

How to collect feedback
- Feedback should seamlessly integrate into the user’s workflow. It should be easy for users to provide feedback without extra work. Feedback collection shouldn’t disrupt user experience and should be easy to ignore. There should be incentives for users to give good feedback.
- One example often cited as good feedback design is from the image generator app Midjourney. For each prompt, Midjourney generates a set of (four) images and gives the user the following options
  - Generate an unscaled version of any of these images. 
  - Generate variations for any of these images. 
  - Regenerate.
- All these options give Midjourney different signals. Options 1 and 2 tell Midjourney which of the four photos is considered by the user to be the most promising. Option 1 gives the strongest positive signal about the chosen photo. Option 2 gives a weaker positive signal. Option 3 signals that none of the photos is good enough. However, users might choose to regenerate even if the existing photos are good just to see what else is possible.
- The feedback alone might be helpful for product analytics. For example, seeing just the thumbs up/thumbs down information is useful for calculating how often people are happy or unhappy with your product. For deeper analysis, though, you would need context around the feedback, such as the previous 5 to 10 dialogue turns. This context can help you figure out what went wrong. However, getting this context might not be possible without explicit user consent, especially if the context might contain personally identifiable information.
- Don’t ask users to do the impossible. For example, if you collect comparative signals from users, don’t ask them to choose between two options they don’t understand.
- Add icons and tooltips to an option if they help people understand it. Avoid a design that can confuse users. Ambiguous instructions can lead to noisy feedback. I once hosted a GPU optimization workshop, using Luma to collect feedback. When I was reading the negative feedback, I was confused. Even though the responses were positive, the star ratings were 1/5. When I dug deeper, I realized that Luma used emojis to represent numbers in their feedback collection form, but the angry emoji, corresponding to a one-star rating, was put where the five-star rating should b

Feedback Limitations
- Leniency bias
  - Leniency bias is the tendency for people to rate items more positively than warranted, often to avoid conflict because they feel compelled to be nice or because it’s the easiest option
  - If you want more granular feedback, removing the strong negative connotation associated with low ratings can help people break out of this bias. For example, instead of showing users numbers one to five, show users options such as the following:
    - “Great ride. Great driver.” 
    - “Pretty good.” 
    - “Nothing to complain about but nothing stellar either.” 
    - “Could’ve been better.” 
    - “Don’t match me with this driver again.”
- Randomness
  - Users often provide random feedback, not out of malice, but because they lack motivation to give more thoughtful input
- Position bias
  - The position in which an option is presented to users influences how this option is perceived. Users are generally more likely to click on the first suggestion than the second. If a user clicks on the first suggestion, this doesn’t necessarily mean that it’s a good suggestion.
  - When designing your feedback system, this bias can be mitigated by randomly varying the positions of your suggestions or by building a model to compute a suggestion’s true success rate based on its position.
- Preference bias
  - Many other biases can affect a person’s feedback, some of which have been discussed in this book. For example, people might prefer the longer response in a side-by-side comparison, even if the longer response is less accurate—length is easier to notice than inaccuracies
- It’s important to inspect your user feedback to uncover its biases. Understanding these biases will help you interpret the feedback correctly, avoiding misleading product decisions.

Degenerate feedback loop
- In a system where user feedback is used to modify a model’s behavior, degenerate feedback loops can arise. A degenerate feedback loop can happen when the predictions themselves influence the feedback, which, in turn, influences the next iteration of the model, amplifying initial biases.
- Imagine you’re building a system to recommend videos. The videos that rank higher show up first, so they get more clicks, reinforcing the system’s belief that they’re the best picks. Initially, the difference between the two videos, A and B, might be minor, but because A was ranked slightly higher, it got more clicks, and the system kept boosting it. Over time, A’s ranking soared, leaving B behind. This feedback loop is why popular videos stay popular, making it tough for new ones to break through. This issue is known as “exposure bias,” “popularity bias,” or “filter bubbles,” and it’s a well-studied problem.
- Acting on user feedback can also turn a conversational agent into, for lack of a better word, a liar. Multiple studies have shown that training a model on user feedback can teach it to give users what it thinks users want, even if that isn’t what’s most accurate or beneficial 
- User feedback is crucial for improving user experience, but if used indiscriminately, it can perpetuate biases and destroy your product. Before incorporating feedback into your product, make sure that you understand the limitations of this feedback and its potential impact.

### Summary

- If each previous chapter focused on a specific aspect of AI engineering, this chapter looked into the process of building applications on top of foundation models as a whole.