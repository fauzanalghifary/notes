# Chapter 6 - RAG and Agents

- To solve a task, a model needs both the instructions on how to do it, and the necessary information to do so. Just like how a human is more likely to give a wrong answer when lacking information, AI models are more likely to make mistakes and hallucinate when they are missing context.
- The last chapter discussed how to write good instructions to the model. This chapter focuses on how to construct the relevant context for each query.
- Two dominating patterns for context construction are RAG, or retrieval-augmented generation, and agents
- The RAG pattern allows the model to retrieve relevant information from external data sources. The agentic pattern allows the model to use tools such as web search and news APIs to gather information.
- While the RAG pattern is chiefly used for constructing context, the agentic pattern can do much more than that. External tools can help models address their shortcomings and expand their capabilities. Most importantly, they give models the ability to directly interact with the world, enabling them to automate many aspects of our lives.
- In a short amount of time, they’ve managed to capture the collective imagination, leading to incredible demos and products that convince many people that they are the future.

### RAG

- RAG is a technique that enhances a model’s generation by retrieving the relevant information from external memory sources. An external memory source can be an internal database, a user’s previous chat sessions, or the internet.
- With RAG, only the information most relevant to the query, as determined by the retriever, is retrieved and input into the model. Lewis et al. found that having access to relevant information can help the model generate more detailed responses while reducing hallucinations
- You can think of RAG as a technique to construct context specific to each query, instead of using the same context for all queries. This helps with managing user data, as it allows you to include data specific to a user only in queries related to this user.
- In the early days of foundation models, RAG emerged as one of the most common patterns. Its main purpose was to overcome the models’ context limitations. Many people think that a sufficiently long context will be the end of RAG. I don’t think so. First, no matter how long a model’s context length is, there will be applications that require context longer than that. After all, the amount of available data only grows over time. People generate and add new data but rarely delete data. Context length is expanding quickly, but not fast enough for the data needs of arbitrary applications
- Second, a model that can process long context doesn’t necessarily use that context well, as discussed in “Context Length and Context Efficiency”. The longer the context, the more likely the model is to focus on the wrong part of the context. Every extra context token incurs extra cost and has the potential to add extra latency. RAG allows a model to use only the most relevant information for each query, reducing the number of input tokens while potentially increasing the model’s performance.
- Efforts to expand context length are happening in parallel with efforts to make models use context more effectively. I wouldn’t be surprised if a model provider incorporates a retrieval-like or attention-like mechanism to help a model pick out the most salient parts of a context to use.
- Anthropic suggested that for Claude models, if “your knowledge base is smaller than 200,000 tokens (about 500 pages of material), you can just include the entire knowledge base in the prompt that you give the model, with no need for RAG or similar methods” (Anthropic, 2024). It’d be amazing if other model developers provide similar guidance for RAG versus long context for their models.

RAG Architecture

- A RAG system has two components: a retriever that retrieves information from external memory sources and a generator that generates a response based on the retrieved information.
- In the original RAG paper, Lewis et al. trained the retriever and the generative model together. In today’s RAG systems, these two components are often trained separately, and many teams build their RAG systems using off-the-shelf retrievers and models. However, finetuning the whole RAG system end-to-end can improve its performance significantly.
- The success of a RAG system depends on the quality of its retriever. A retriever has two main functions: indexing and querying. Indexing involves processing data so that it can be quickly retrieved later. Sending a query to retrieve data relevant to it is called querying. How to index data depends on how you want to retrieve it later on
- For now, let’s assume that all documents have been split into workable chunks. For each query, our goal is to retrieve the data chunks most relevant to this query. Minor post-processing is often needed to join the retrieved data chunks with the user prompt to generate the final prompt. This final prompt is then fed into the generative model.

Retrieval Algorithms

- At its core, retrieval works by ranking documents based on their relevance to a given query. Retrieval algorithms differ based on how relevance scores are computed. I’ll start with two common retrieval mechanisms: term-based retrieval and embedding-based retrieval.
- In the literature, you might encounter the division of retrieval algorithms into the following categories: sparse versus dense. This book, however, opted for term-based versus embedding-based categorization.
  - A sparse vector is a vector where the majority of the values are 0. Term-based retrieval is considered sparse, as each term can be represented using a sparse one-hot vector, a vector that is 0 everywhere except one value of 1.
  - Dense retrievers represent data using dense vectors. A dense vector is a vector where the majority of the values aren’t 0. Embedding-based retrieval is typically considered dense, as embeddings are generally dense vectors.

Term-based retrieval

- Given a query, the most straightforward way to find relevant documents is with keywords. Some people call this approach lexical retrieval. For example, given the query “AI engineering”, the model will retrieve all the documents that contain “AI engineering”. However, this approach has two problems:
  - Many documents might contain the given term, and your model might not have sufficient context space to include all of them as context.
  - A prompt can be long and contain many terms. Some are more important than others. You need a way to identify important terms.
- TF-IDF is an algorithm that combines these two metrics: term frequency (TF) and inverse document frequency (IDF)
- Two common term-based retrieval solutions are Elasticsearch and BM25.

Embedding-based retrieval

- Term-based retrieval computes relevance at a lexical level rather than a semantic level. As mentioned in Chapter3, the appearance of a text doesn’t necessarily capture its meaning. This can result in returning documents irrelevant to your intent.
- embedding-based retrievers aim to rank documents based on how closely their meanings align with the query. This approach is also known as semantic retrieval.
- With embedding-based retrieval, indexing has an extra function: converting the original data chunks into embeddings. The database where the generated embeddings are stored is called a vector database
- Querying then consists of two steps,
  - Embedding model: convert the query into an embedding using the same embedding model used during indexing.
  - Retriever: fetch k data chunks whose embeddings are closest to the query embedding, as determined by the retriever. The number of data chunks to fetch, k, depends on the use case, the generative model, and the query.
- As a reminder, an embedding is typically a vector that aims to preserve the important properties of the original data. An embedding-based retriever doesn’t work if the embedding model is bad.
- Embedding-based retrieval also introduces a new component: vector databases. A vector database stores vectors. However, storing is the easy part of a vector database. The hard part is vector search. Given a query embedding, a vector database is responsible for finding vectors in the database close to the query and returning them. Vectors have to be indexed and stored in a way that makes vector search fast and efficient.
- Vector search is common in any application that uses embeddings: search, recommendation, data organization, information retrieval, clustering, fraud detection, and more.
- Vector search is typically framed as a nearest-neighbor search problem. For example, given a query, find the k nearest vectors
- For large datasets, vector search is typically done using an approximate nearest neighbor (ANN) algorithm. Due to the importance of vector search, many algorithms and libraries have been developed for it
- Most application developers won’t implement vector search themselves, so I’ll give only a quick overview of different approaches. This overview might be helpful as you evaluate solutions.
- In general, vector databases organize vectors into buckets, trees, or graphs. Vector search algorithms differ based on the heuristics they use to increase the likelihood that similar vectors are close to each other. Vectors can also be quantized (reduced precision) or made sparse. The idea is that quantized and sparse vectors are less computationally intensive to work with
- Even though vector databases emerged as their own category with the rise of RAG, any database that can store vectors can be called a vector database. Many traditional databases have extended or will extend to support vector storage and vector search.

Comparing retrieval algorithms

- Due to the long history of retrieval, its many mature solutions make both term-based and embedding-based retrieval relatively easy to start. Each approach has its pros and cons.
- Term-based retrieval is generally much faster than embedding-based retrieval during both indexing and query. Term extraction is faster than embedding generation, and mapping from a term to the documents that contain it can be less computationally expensive than a nearest-neighbor search.
- Term-based retrieval also works well out of the box. Solutions like Elasticsearch and BM25 have successfully powered many search and retrieval applications. However, its simplicity also means that it has fewer components you can tweak to improve its performance.
- Embedding-based retrieval, on the other hand, can be significantly improved over time to outperform term-based retrieval. You can finetune the embedding model and the retriever, either separately, together, or in conjunction with the generative model. 
- However, converting data into embeddings can obscure keywords, such as specific error codes, e.g., EADDRNOTAVAIL (99), or product names, making them harder to search later on. This limitation can be addressed by combining embedding-based retrieval with term-based retrieval, as discussed later in this chapter.
- The quality of a retriever can be evaluated based on the quality of the data it retrieves. Two metrics often used by RAG evaluation frameworks are context precision and context recall, or precision and recall for short (context precision is also called context relevance):
  - Context precision: Out of all the documents retrieved, what percentage is relevant to the query?
  - Context recall: Out of all the documents that are relevant to the query, what percentage is retrieved?
- To compute these metrics, you curate an evaluation set with a list of test queries and a set of documents. For each test query, you annotate each test document to be relevant or not relevant. The annotation can be done either by humans or AI judges. You then compute the precision and recall score of the retriever on this evaluation set.
- Context precision is simpler to compute. You only need to compare the retrieved documents to the query, which can be done by an AI judge.
- The quality of a retriever should also be evaluated in the context of the whole RAG system. Ultimately, a retriever is good if it helps the system generate high-quality answers. Evaluating outputs of generative models is discussed in Chapters 3 and 4.
- Whether the performance promise of a semantic retrieval system is worth pursuing depends on how much you prioritize cost and latency, particularly during the querying phase. Since much of RAG latency comes from output generation, especially for long outputs, the added latency by query embedding generation and vector search might be minimal compared to the total RAG latency. Even so, the added latency still can impact user experience.
- Generating embeddings costs money. This is especially an issue if your data changes frequently and requires frequent embedding regeneration.
- With retrieval systems, you can make certain trade-offs between indexing and querying. The more detailed the index is, the more accurate the retrieval process will be, but the indexing process will be slower and more memory-consuming.
- To summarize, the quality of a RAG system should be evaluated both component by component and end to end. To do this, you should do the following things:
  - Evaluate the retrieval quality. 
  - Evaluate the final RAG outputs.
  - Evaluate the embeddings (for embedding-based retrieval).

Combining retrieval algorithms

- Given the distinct advantages of different retrieval algorithms, a production retrieval system typically combines several approaches. Combining term-based retrieval and embedding-based retrieval is called hybrid search.
- Different algorithms can be used in sequence. First, a cheap, less precise retriever, such as a term-based system, fetches candidates. Then, a more precise but more expensive mechanism, such as k-nearest neighbors, finds the best of these candidates. This second step is also called reranking.
- For example, given the term “transformer”, you can fetch all documents that contain the word transformer, regardless of whether they are about the electric device, the neural architecture, or the movie. Then you use vector search to find among these documents those that are actually related to your transformer query.
- Different algorithms can also be used in parallel as an ensemble. Remember that a retriever works by ranking documents by their relevance scores to the query. You can use multiple retrievers to fetch candidates at the same time, then combine these different rankings together to generate a final ranking.

Retrieval Optimization

- Four tactics discussed here are chunking strategy, reranking, query rewriting, and contextual retrieval.

Chunking strategy

- How your data should be indexed depends on how you intend to retrieve it later.
- This is an important consideration because the chunking strategy you use can significantly impact the performance of your retrieval system.
- The simplest strategy is to chunk documents into chunks of equal length based on a certain unit. Common units are characters, words, sentences, and paragraphs.
- You can also split documents recursively using increasingly smaller units until each chunk fits within your maximum chunk size. For example, you can start by splitting a document into sections. If a section is too long, split it into paragraphs. If a paragraph is still too long, split it into sentences. This reduces the chance of related texts being arbitrarily broken off.
- When a document is split into chunks without overlap, the chunks might be cut off in the middle of important context, leading to the loss of critical information.
- Overlapping ensures that important boundary information is included in at least one chunk
- The chunk size shouldn’t exceed the maximum context length of the generative model. For the embedding-based approach, the chunk size also shouldn’t exceed the embedding model’s context limit.
- You can also chunk documents using tokens, determined by the generative model’s tokenizer, as a unit.
- Chunking by tokens makes it easier to work with downstream models. However, the downside of this approach is that if you switch to another generative model with a different tokenizer, you’d need to reindex your data.
- Regardless of which strategy you choose, chunk sizes matter. A smaller chunk size allows for more diverse information. Smaller chunks mean that you can fit more chunks into the model’s context. If you halve the chunk size, you can fit twice as many chunks. More chunks can provide a model with a wider range of information, which can enable the model to produce a better answer.
- Small chunk sizes, however, can cause the loss of important information.
- Smaller chunk sizes can also increase computational overhead. This is especially an issue for embedding-based retrieval. Halving the chunk size means that you have twice as many chunks to index and twice as many embedding vectors to generate and store. Your vector search space will be twice as big, which can reduce the query speed.
- There is no universal best chunk size or overlap size. You have to experiment to find what works best for you.

Reranking

- The initial document rankings generated by the retriever can be further reranked to be more accurate. Reranking is especially useful when you need to reduce the number of retrieved documents, either to fit them into your model’s context or to reduce the number of input tokens.
-  A cheap but less precise retriever fetches candidates, then a more precise but more expensive mechanism reranks these candidates.
- Documents can also be reranked based on time, giving higher weight to more recent data. This is useful for time-sensitive applications such as news aggregation, chat with your emails (e.g., a chatbot that can answer questions about your emails), or stock market analysis.
- Context reranking differs from traditional search reranking in that the exact position of items is less critical. In search, the rank (e.g., first or fifth) is crucial. In context reranking, the order of documents still matters because it affects how well a model can process them.

Query rewriting

- Query rewriting is also known as query reformulation, query normalization, and sometimes query expansion
- While I put query rewriting in “RAG”, query rewriting isn’t unique to RAG. In traditional search engines, query rewriting is often done using heuristics. In AI applications, query rewriting can also be done using other AI models, using a prompt similar to “Given the following conversation, rewrite the last user input to reflect what the user is actually asking”

Contextual retrieval

- The idea behind contextual retrieval is to augment each chunk with relevant context to make it easier to retrieve the relevant chunks. A simple technique is to augment a chunk with metadata like tags and keywords. For ecommerce, a product can be augmented by its description and reviews. Images and videos can be queried by their titles or captions.
- You can also augment each chunk with the questions it can answer. For customer support, you can augment each article with related questions. For example, the article on how to reset your password can be augmented with queries like “How to reset password?”, “I forgot my password”, “I can’t log in”, or even “Help, I can’t find my account”
- If a document is split into multiple chunks, some chunks might lack the necessary context to help the retriever understand what the chunk is about. To avoid this, you can augment each chunk with the context from the original document, such as the original document’s title and summary. Anthropic used AI models to generate a short context, usually 50-100 tokens, that explains the chunk and its relationship to the original document
- Here are some key factors to keep in mind when evaluating a retrieval solution:
  - What retrieval mechanisms does it support? Does it support hybrid search? 
  - If it’s a vector database, what embedding models and vector search algorithms does it support? 
  - How scalable is it, both in terms of data storage and query traffic? Does it work for your traffic patterns? 
  - How long does it take to index your data? How much data can you process (such as add/delete) in bulk at once? 
  - What’s its query latency for different retrieval algorithms? 
  - If it’s a managed solution, what’s its pricing structure? Is it based on the document/vector volume or on the query volume?

Multimodal RAG

- If your generator is multimodal, its contexts might be augmented not only with text documents but also with images, videos, audio, etc., from external sources.
- If the images have metadata—such as titles, tags, and captions—they can be retrieved using the metadata. For example, an image is retrieved if its caption is considered relevant to the query.

RAG with tabular data

- Most applications work not only with unstructured data like texts and images but also with tabular data. Many queries might need information from data tables to answer. The workflow for augmenting a context using tabular data is significantly different from the classic RAG workflow.
- For the text-to-SQL step, if there are many available tables whose schemas can’t all fit into the model context, you might need an intermediate step to predict what tables to use for each query. Text-to-SQL can be done by the same generator that generates the final response or a specialized text-to-SQL model.

### Agents

- Intelligent agents are considered by many to be the ultimate goal of AI.
- The unprecedented capabilities of foundation models have opened the door to agentic applications that were previously unimaginable. These new capabilities make it finally possible to develop autonomous, intelligent agents to act as our assistants, coworkers, and coaches. They can help us create a website, gather data, plan a trip, do market research, manage a customer account, automate data entry, prepare us for interviews, interview our candidates, negotiate a deal, etc. The possibilities seem endless, and the potential economic value of these agents is enormous.
- AI-powered agents are an emerging field, with no established theoretical frameworks for defining, developing, and evaluating them. This section is a best-effort attempt to build a framework from the existing literature, but it will evolve as the field does. Compared to the rest of the book, this section is more experimental.
- two aspects that determine the capabilities of an agent: tools and planning
- Even though agents are novel, they are built upon concepts that have already appeared in this book, including self-critique, chain-of-thought, and structured outputs.

Agent Overview

- An agent is anything that can perceive its environment and act upon that environment.10 This means that an agent is characterized by the environment it operates in and the set of actions it can perform.
- The environment an agent can operate in is defined by its use case.
- The set of actions an AI agent can perform is augmented by the tools it has access to. 
- There’s a strong dependency between an agent’s environment and its set of tools. The environment determines what tools an agent can potentially use
- An AI agent is meant to accomplish tasks typically provided by the users in the inputs. In an AI agent, AI is the brain that processes the information it receives, including the task and feedback from the environment, plans a sequence of actions to achieve this task, and determines whether the task has been accomplished.
- Compared to non-agent use cases, agents typically require more powerful models for two reasons:
  - Compound mistakes: an agent often needs to perform multiple steps to accomplish a task, and the overall accuracy decreases as the number of steps increases. If the model’s accuracy is 95% per step, over 10 steps, the accuracy will drop to 60%, and over 100 steps, the accuracy will be only 0.6%.
  - Higher stakes: with access to tools, agents are capable of performing more impactful tasks, but any failure could have more severe consequences.
- A task that requires many steps can take time and money to run.11 However, if agents can be autonomous, they can save a lot of human time, making their costs worthwhile.
- Given an environment, the success of an agent in an environment depends on the tool inventory it has access to and the strength of its AI planner.

Tools

- A system doesn’t need access to external tools to be an agent. However, without external tools, the agent’s capabilities would be limited. By itself, a model can typically perform one action—for example, an LLM can generate text, and an image generator can generate images. External tools make an agent vastly more capable.
- Tools help an agent to both perceive the environment and act upon it. Actions that allow an agent to perceive the environment are read-only actions, whereas actions that allow an agent to act upon the environment are write actions.
- The set of tools an agent has access to is its tool inventory. Since an agent’s tool inventory determines what an agent can do, it’s important to think through what and how many tools to give an agent. More tools give an agent more capabilities. However, the more tools there are, the more challenging it is to understand and utilize them well. Experimentation is necessary to find the right set of tools, as discussed in “Tool selection”.
- Depending on the agent’s environment, there are many possible tools. Here are three categories of tools that you might want to consider: 
  - knowledge augmentation (i.e., context construction),
  - capability extension, and 
  - tools that let your agent act upon its environment.
- Knowledge augmentation
  - An important category of tools includes those that help augment your agent’s knowledge of your agent. Some of them have already been discussed: text retriever, image retriever, and SQL executor. Other potential tools include internal people search, an inventory API that returns the status of different products, Slack retrieval, an email reader, etc.
  - However, tools can also give models access to public information, especially from the internet.
  - Web browsing was among the earliest and most anticipated capabilities to be incorporated into chatbots like ChatGPT. Web browsing prevents a model from going stale. A model goes stale when the data it was trained on becomes outdated.
  - Without web browsing, a model won’t be able to tell you about the weather, news, upcoming events, stock prices, flight status, etc.
  - While web browsing allows your agent to reference up-to-date information to generate better responses and reduce hallucinations, it can also open up your agent to the cesspools of the internet. Select your Internet APIs with care.
- Capability extension
  - Other simple tools that can significantly boost a model’s capability include a calendar, timezone converter, unit converter (e.g., from lbs to kg), and translator that can translate to and from the languages that the model isn’t good at.
  - More complex but powerful tools are code interpreters. Instead of training a model to understand code, you can give it access to a code interpreter so that it can execute a piece of code, return the results, or analyze the code’s failures. This capability lets your agents act as coding assistants, data analysts, and even research assistants that can write code to run experiments and report results
  - Agents can also use a code interpreter to generate charts and graphs, a LaTeX compiler to render math equations, or a browser to render web pages from HTML code.
  - Tool use can significantly boost a model’s performance compared to just prompting or even finetuning
- Write actions
  - Write actions enable a system to do more. They can enable you to automate the whole customer outreach workflow: researching potential customers, finding their contacts, drafting emails, sending first emails, reading responses, following up, extracting orders, updating your databases with new orders, etc.
  - Trust in the system’s capabilities and its security measures is crucial. You need to ensure that the system is protected from bad actors who might try to manipulate it into performing harmful actions.

Planning

- At the heart of a foundation model agent is the model responsible for solving a task. A task is defined by its goal and constraints.
- Complex tasks require planning. The output of the planning process is a plan, which is a roadmap outlining the steps needed to accomplish a task. Effective planning typically requires the model to understand the task, consider different options to achieve this task, and choose the most promising one.
- As an important computational problem, planning is well studied and would require several volumes to cover. I’ll only be able to cover the surface here.

Planning overview

- Among the correct solutions, some are more efficient than others. Consider the query, “How many companies without revenue have raised at least $1 billion?” There are many possible ways to solve this, but as an illustration, consider the two options:
  - Find all companies without revenue, then filter them by the amount raised. 
  - Find all companies that have raised at least $1 billion, then filter them by revenue. 
  - The second option is more efficient.
- To avoid fruitless execution, planning should be decoupled from execution. You ask the agent to first generate a plan, and only after this plan is validated is it executed. The plan can be validated using heuristics. For example, one simple heuristic is to eliminate plans with invalid actions.
- Your system now has three components: one to generate plans, one to validate plans, and another to execute plans. If you consider each component an agent, this is a multi-agent system.
- Planning requires understanding the intention behind a task: what’s the user trying to do with this query? An intent classifier is often used to help agents plan. As shown in “Break Complex Tasks into Simpler Subtasks”, intent classification can be done using another prompt or a classification model trained for this task. The intent classification mechanism can be considered another agent in your multi-agent system.
- So far, we’ve assumed that the agent automates all three stages: generating plans, validating plans, and executing plans. In reality, humans can be involved at any of those stages to aid with the process and mitigate risks. A human expert can provide a plan, validate a plan, or execute parts of a plan. For example, for complex tasks for which an agent has trouble generating the whole plan, a human expert can provide a high-level plan that the agent can expand upon. If a plan involves risky operations, such as updating a database or merging a code change, the system can ask for explicit human approval before executing or let humans execute these operations. To make this possible, you need to clearly define the level of automation an agent can have for each action.
- To summarize, solving a task typically involves the following processes.
  - Plan generation
  - Reflection and error correction
  - Execution
  - Reflection and error correction

Foundation models as planners

- An open question is how well foundation models can plan. Many researchers believe that foundation models, at least those built on top of autoregressive language models, cannot. Meta’s Chief AI Scientist Yann LeCun states unequivocally that autoregressive LLMs can’t plan (2023). In the article “Can LLMs Really Reason and Plan?” Kambhampati (2023) argues that LLMs are great at extracting knowledge but not planning.
- However, while there is a lot of anecdotal evidence that LLMs are poor planners, it’s unclear whether it’s because we don’t know how to use LLMs the right way or because LLMs, fundamentally, can’t plan.
- Planning, at its core, is a search problem. You search among different paths to the goal, predict the outcome (reward) of each path, and pick the path with the most promising outcome. Often, you might determine that no path exists that can take you to the goal.
- Search often requires backtracking. For example, imagine you’re at a step where there are two possible actions: A and B. After taking action A, you enter a state that’s not promising, so you need to backtrack to the previous state to take action B.
- This means it’s not sufficient to prompt a model to generate only a sequence of actions like what the popular chain-of-thought prompting technique does. The paper “Reasoning with Language Model is Planning with World Model” (Hao et al., 2023) argues that an LLM, by containing so much information about the world, is capable of predicting the outcome of each action. This LLM can incorporate this outcome prediction to generate coherent plans.
- The agent is a core concept in RL

Plan generation

- The simplest way to turn a model into a plan generator is with prompt engineering
- Here are a few approaches to make an agent better at planning:
  - Write a better system prompt with more examples. 
  - Give better descriptions of the tools and their parameters so that the model understands them better. 
  - Rewrite the functions themselves to make them simpler, such as refactoring a complex function into two simpler functions. 
  - Use a stronger model. In general, stronger models are better at planning. 
  - Finetune a model for plan generation.
- When working with agents, always ask the system to report what parameter values it uses for each function call. Inspect these values to make sure they are correct.

Planning granularity

- A plan is a roadmap outlining the steps needed to accomplish a task. A roadmap can be of different levels of granularity.
- There’s a planning/execution trade-off. A detailed plan is harder to generate but easier to execute. A higher-level plan is easier to generate but harder to execute. An approach to circumvent this trade-off is to plan hierarchically. First, use a planner to generate a high-level plan, such as a quarter-to-quarter plan. Then, for each quarter, use the same or a different planner to generate a month-to-month plan.
- Using more natural language helps your plan generator become robust to changes in tool APIs. If your model was trained mostly on natural language, it’ll likely be better at understanding and generating plans in natural language and less likely to hallucinate.
- The downside of this approach is that you need a translator to translate each natural language action into executable commands.13 However, translating is a much simpler task than planning and can be done by weaker models with a lower risk of hallucination.

Complex plans

- The plan examples so far have been sequential: the next action in the plan is always executed after the previous action is done. The order in which actions can be executed is called a control flow. The sequential form is just one type of control flow
- Other types of control flows include the parallel, if statement, and for loop
  - Sequential
  - Parallel
  - If statement
  - For loop

Reflection and error correction

- Even the best plans need to be constantly evaluated and adjusted to maximize their chance of success. While reflection isn’t strictly necessary for an agent to operate, it’s necessary for an agent to succeed.
- Reflection can be useful in many places during a task process:
  - After receiving a user query to evaluate if the request is feasible. 
  - After the initial plan generation to evaluate whether the plan makes sense. 
  - After each execution step to evaluate if it’s on the right track. 
  - After the whole plan has been executed to determine if the task has been accomplished.
- You can implement reflection in a multi-agent setting: one agent plans and takes actions, and another agent evaluates the outcome after each step or after a number of steps
- If the agent’s response failed to accomplish the task, you can prompt the agent to reflect on why it failed and how to improve. Based on this suggestion, the agent generates a new plan. This allows agents to learn from their mistakes
- Compared to plan generation, reflection is relatively easy to implement and can bring surprisingly good performance improvement. The downside of this approach is latency and cost. Thoughts, observations, and sometimes actions can take a lot of tokens to generate, which increases cost and user-perceived latency, especially for tasks with many intermediate steps

Tool selection

- Because tools often play a crucial role in a task’s success, tool selection requires careful consideration. The tools to give your agent depend on the environment and the task, but they also depend on the AI model that powers the agent.
- Experiments by Lu et al. (2023) also demonstrate two points:
  - Different tasks require different tools
  - Different models have different tool preferences.

Agent Failure Modes and Evaluation

- Evaluation is about detecting failures. The more complex a task an agent performs, the more possible failure points there are. Other than the failure modes common to all AI applications discussed in Chapters 3 and 4, agents also have unique failures caused by planning, tool execution, and efficiency.

Planning failures

- Planning is hard and can fail in many ways. The most common mode of planning failure is tool use failure. The agent might generate a plan with one or more of these errors:
  - Invalid tool
  - Valid tool, invalid parameters.
  - Valid tool, incorrect parameter values
- Another mode of planning failure is goal failure: the agent fails to achieve the goal. This can be because the plan doesn’t solve a task, or it solves the task without following the constraints
- A common constraint that is often overlooked by agent evaluation is time. In many cases, the time an agent takes matters less, because you can assign a task to an agent and only need to check in when it’s done. However, in many cases, the agent becomes less useful with time
- An interesting mode of planning failure is caused by errors in reflection. The agent is convinced that it’s accomplished a task when it hasn’t.
- Compute the following metrics:
  - Out of all generated plans, how many are valid? 
  - For a given task, how many plans does the agent have to generate, on average, to get a valid plan? 
  - Out of all tool calls, how many are valid? 
  - How often are invalid tools called? 
  - How often are valid tools called with invalid parameters? 
  - How often are valid tools called with incorrect parameter values?
- Analyze the agent’s outputs for patterns. What types of tasks does the agent fail more on? Do you have a hypothesis why? What tools does the model frequently make mistakes with? Some tools might be harder for an agent to use. You can improve an agent’s ability to use a challenging tool by better prompting, more examples, or finetuning. If all fail, you might consider swapping this tool for something easier to use.

Tool failures

- Tool failures happen when the correct tool is used, but the tool output is wrong. One failure mode is when a tool just gives the wrong outputs.
- Tool failures can also happen because the agent doesn’t have access to the right tools for the task. An obvious example is when the task involves retrieving the current stock prices from the internet, and the agent doesn’t have access to the internet.
- Detecting missing tool failures requires an understanding of what tools should be used. If your agent frequently fails on a specific domain, this might be because it lacks tools for this domain. Work with human domain experts and observe what tools they would use.

Efficiency

- An agent might generate a valid plan using the right tools to accomplish a task, but it might be inefficient. Here are a few things you might want to track to evaluate an agent’s efficiency:
  - How many steps does the agent need, on average, to complete a task? 
  - How much does the agent cost, on average, to complete a task? 
  - How long does each action typically take? Are there any actions that are especially time-consuming or expensive?
- In this chapter, we’ve discussed in detail how RAG and agent systems function. Both patterns often deal with information that exceeds a model’s context limit. A memory system that supplements the model’s context in handling information can significantly enhance its capabilities. Let’s now explore how a memory system works.

### Memory

- Memory refers to mechanisms that allow a model to retain and utilize information. A memory system is especially useful for knowledge-rich applications like RAG and multi-step applications like agents. A RAG system relies on memory for its augmented context, which can grow over multiple turns as it retrieves more information. An agentic system needs memory to store instructions, examples, context, tool inventories, plans, tool outputs, reflections, and more.
- An AI model typically has three main memory mechanisms:
  - Internal knowledge
    - The model itself is a memory mechanism, as it retains the knowledge from the data it was trained on. This knowledge is its internal knowledge. A model’s internal knowledge doesn’t change unless the model itself is updated.
  - Short-term memory
    - A model’s context is a memory mechanism. Previous messages in a conversation can be added to the model’s context, allowing the model to leverage them to generate future responses
  - Long-term memory
    - External data sources that a model can access via retrieval, such as in a RAG system, are a memory mechanism. This can be considered the model’s long-term memory, as it can be persisted across tasks. Unlike a model’s internal knowledge, information in the long-term memory can be deleted without updating the model.
- Which memory mechanism to use for your data depends on its frequency of use. Information essential for all tasks should be incorporated into the model’s internal knowledge via training or finetuning. Information that is rarely needed should reside in its long-term memory. Short-term memory is reserved for immediate, context-specific information.
- Augmenting an AI model with a memory system has many benefits.
  - Manage information overflow within a session
    - During the process of executing a task, an agent acquires a lot of new information, which can exceed the agent’s maximum context length. The excess information can be stored in a memory system with long-term memories.
  - Persist information between sessions
  - Boost a model’s consistency
    - Having a memory system capable of storing structured data can help maintain the structural integrity of your data.
- A memory system for AI models typically consists of two functions:
  - Memory management: managing what information should be stored in the short-term and long-term memory. 
  - Memory retrieval: retrieving information relevant to the task from long-term memory.
- Long-term memory can be used to store the overflow from short-term memory. This operation depends on how much space you want to allocate for short-term memory. For a given query, the context input into the model consists of both its short-term memory and information retrieved from its long-term memory. A model’s short-term capacity is, therefore, determined by how much of the context should be allocated for information retrieved from long-term memory. For example, if 30% of the context is reserved, then the model can use at most 70% of the context limit for short-term memory. When this threshold is reached, the overflow can be moved to long-term memory.
- When encountering contradicting pieces of information, some people opt to keep the newer ones. Some people ask AI models to judge which one to keep. How to handle contradiction depends on the use case. Having contradictions can cause an agent to be confused but can also help it draw from different perspectives.

### Summary

- This chapter started with RAG, the pattern that emerged first between the two. Many tasks require extensive background knowledge that often exceeds a model’s context window.
- Originally developed to overcome a model’s context limitations, RAG also enables more efficient use of information, improving response quality while reducing costs. From the early days of foundation models, it was clear that the RAG pattern would be immensely valuable for a wide range of applications, and it has since been rapidly adopted across both consumer and enterprise use cases.
- RAG employs a two-step process. It first retrieves relevant information from external memory and then uses this information to generate more accurate responses. The success of a RAG system depends on the quality of its retriever. Term-based retrievers, such as Elasticsearch and BM25, are much lighter to implement and can provide strong baselines. Embedding-based retrievers are more computationally intensive but have the potential to outperform term-based algorithms.
- The RAG pattern can be seen as a special case of agent where the retriever is a tool the model can use. Both patterns allow a model to circumvent its context limitation and stay more up-to-date, but the agentic pattern can do even more than that. An agent is defined by its environment and the tools it can access. In an AI-powered agent, AI is the planner that analyzes its given task, considers different solutions, and picks the most promising one. A complex task can require many steps to solve, which requires a powerful model to plan. A model’s ability to plan can be augmented with reflection and a memory system to help it keep track of its progress.
- Both RAG and agents work with a lot of information, which often exceeds the maximum context length of the underlying model. This necessitates the introduction of a memory system for managing and using all the information a model has. This chapter ended with a short discussion on what this component looks like.
- 

