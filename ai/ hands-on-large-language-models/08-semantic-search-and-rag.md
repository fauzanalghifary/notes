# Chapter 8. Semantic Search and Retrieval-Augmented Generation

- Semantic search, which enables searching by meaning, and not simply keyword matching.
- This problem grew to be known as model “hallucinations,” and one of the leading ways to reduce it is to build systems that can retrieve relevant information and provide it to the LLM to aid it in generating more factual answers. This method, called RAG, is one of the most popular applications of LLMs.

## Overview of Semantic Search and RAG

- Dense retrieval
  - Dense retrieval systems rely on the concept of embeddings, the same concept we’ve encountered in the previous chapters, and turn the search problem into retrieving the nearest neighbors of the search query (after both the query and the documents are converted into embeddings)
- Reranking
  - Search systems are often pipelines of multiple steps. A reranking language model is one of these steps and is tasked with scoring the relevance of a subset of results against the query;
- RAG
  - Generative search is a subset of a broader type of category of systems better called RAG systems. 
  - These are text generation systems that incorporate search capabilities to reduce hallucinations, increase factuality, and/or ground the generation model on a specific dataset.

## Semantic Search with Language Models

## Dense Retrieval

- Figure8-6 shows how we chunk a document before proceeding to embed each chunk. Those embedding vectors are then stored in the vector database and are ready for retrieval.
- Another caveat of dense retrieval is when a user wants to find an exact match for a specific phrase. That’s a case that’s perfect for keyword matching. That’s one reason why hybrid search, which includes both semantic search and keyword search, is advised instead of relying solely on dense retrieval.
- Dense retrieval systems also find it challenging to work properly in domains other than the ones that they were trained on. So, for example, if you train a retrieval model on internet and Wikipedia data, and then deploy it on legal texts (without having enough legal data as part of the training set), the model will not work as well in that legal domain.
- One limitation of Transformer language models is that they are limited in context sizes, meaning we cannot feed them very long texts that go above the number of words or tokens that the model supports. So how do we embed long texts?
  - There are several possible ways, and two possible approaches shown in Figure8-7 include indexing one vector per document and indexing multiple vectors per document.
  - The chunking approach is better because it has full coverage of the text and because the vectors tend to capture individual concepts inside the text. This leads to a more expressive search index
  - The best way of chunking a long text will depend on the types of texts and queries your system anticipates.
- The fine-tuning process aims to make the embeddings of these queries close to the embedding of the resulting sentence.

## Reranking

- A lot of organizations have already built search systems. For those organizations, an easier way to incorporate language models is as a final step inside their search pipeline. This step is tasked with changing the order of the search results based on relevance to the search query. This one step can vastly improve search results

## Retrieval Evaluation Metrics

