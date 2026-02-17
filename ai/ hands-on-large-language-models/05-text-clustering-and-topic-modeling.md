# Chapter 5. Text Clustering and Topic Modeling

- Text clustering aims to group similar texts based on their semantic content, meaning, and relationships
- The recent evolution of language models, which enable contextual and semantic representations of text, has enhanced the effectiveness of text clustering. Language is more than a bag of words, and recent language models have proved to be quite capable of capturing that notion. Text clustering, unbound by supervision, allows for creative solutions and diverse applications, such as finding outliers, speedup labeling, and finding incorrectly labeled data.

## A Common Pipeline for Text Clustering

- Text clustering allows for discovering patterns in data that you may or may not be familiar with. It allows for getting an intuitive understanding of the task, for example, a classification task, but also of its complexity. 
- a common pipeline that has gained popularity involves three steps and algorithms:
  - Convert the input documents to embeddings with an embedding model.
  - Reduce the dimensionality of embeddings with a dimensionality reduction model. 
  - Find groups of semantically similar documents with a cluster model.

## Embedding Documents

- embeddings are numerical representations of text that attempt to capture its meaning.