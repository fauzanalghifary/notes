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






### Memory



### Summary