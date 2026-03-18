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

## Reducing the Dimensionality of Embeddings

- Before we cluster the embeddings, we will first need to take their high dimensionality into account. As the number of dimensions increases, there is an exponential growth in the number of possible values within each dimension. Finding all subspaces within each dimension becomes increasingly complex.
- As a result, high-dimensional data can be troublesome for many clustering techniques as it gets more difficult to identify meaningful clusters. Instead, we can make use of dimensionality reduction
- Dimensionality reduction techniques aim to preserve the global structure of high-dimensional data by finding low-dimensional representations.
- To help the cluster model create meaningful clusters, the second step in our clustering pipeline is therefore dimensionality reduction

## Cluster the Reduced Embeddings

- Instead, a density-based algorithm freely calculates the number of clusters and does not force all data points to be part of a cluster

## Inspecting the Clusters

- Now that we have generated our clusters, we can inspect each cluster manually and explore the assigned documents to get an understanding of its content.
- We can take this one step further and attempt to visualize our results instead of going through all documents manually. To do so, we will need to reduce our document embeddings to two dimensions, as that allows us to plot the documents on an x/y plane
- This is visually appealing but does not yet allow us to see what is happening inside the clusters. Instead, we can extend this visualization by going from text clustering to topic modeling.

## From Text Clustering to Topic Modeling

- Text clustering is a powerful tool for finding structure among large collections of documents
- This idea of finding themes or latent topics in a collection of textual data is often referred to as topic modeling. Traditionally, it involves finding a set of keywords or phrases that best represent and capture the meaning of the topic

## BERTopic: A Modular Topic Modeling Framework

- BERTopic is a topic modeling technique that leverages clusters of semantically similar texts to extract various types of topic representations
- Steps:
  - We embed documents, reduce their dimensionality, and finally cluster the reduced embedding to create groups of semantically similar documents.
  -  it models a distribution over words in the corpus’s vocabulary by leveraging a classic method, namely bag-of-words
- With this pipeline, we can cluster semantically similar documents and from the clusters generate topics represented by several keywords. The higher a word’s weight in a topic, the more representative it is of that topic.

## Summary

- Despite supervised methods like classification being prevalent in recent years, unsupervised approaches such as text clustering hold immense potential due to their ability to group texts based on semantic content without prior labeling.
- We covered a common pipeline for clustering textual documents that starts with converting input text into numerical representations, which we call embeddings. Then, dimensionality reduction is applied to these embeddings to simplify high-dimensional data for better clustering outcomes. Finally, a clustering algorithm on the dimensionality-reduced embeddings is applied to cluster the input text. Manually inspecting the clusters helped us understand which documents they contained and how to interpret these clusters.
- To transition away from this manual inspection, we explored how BERTopic extends this text clustering pipeline with a method for automatically representing the clusters. This methodology is often referred to as topic modeling, which attempts to uncover themes within large amounts of documents.