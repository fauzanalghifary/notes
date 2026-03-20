# Chapter 8. Semantic Search and Retrieval-Augmented Generation

- Semantic search, which enables searching by meaning, and not simply keyword matching.
- This problem grew to be known as model “hallucinations,” and one of the leading ways to reduce it is to build systems that can retrieve relevant information and provide it to the LLM to aid it in generating more factual answers. This method, called RAG, is one of the most popular applications of LLMs.

## Overview of Semantic Search and RAG

- Dense retrieval
  - Dense retrieval systems rely on the concept of embeddings, the same concept we’ve encountered in the previous chapters, and turn the search problem into retrieving the nearest neighbors of the search query (after both the query and the documents are converted into embeddings)
- Reranking
  - 