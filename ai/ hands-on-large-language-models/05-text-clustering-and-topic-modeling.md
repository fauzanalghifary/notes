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