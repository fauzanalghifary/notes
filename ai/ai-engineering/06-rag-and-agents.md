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



### Agents



### Memory



### Summary