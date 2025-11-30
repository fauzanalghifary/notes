# Chapter 1. An Introduction to Large Language Models

## What Is Language AI?

- The term artificial intelligence (AI) is often used to describe computer systems dedicated to performing tasks close to human intelligence, such as speech recognition, language translation, and visual perception. It is the intelligence of software as opposed to the intelligence of humans.

## Representing Language as a Bag-of-Words

- tokenization, the process of splitting up the sentences into individual words or subwords (tokens)
- after tokenization, we combine all unique words from each sentence to create a vocabulary that we can use to represent the sentences.
- Using our vocabulary, we simply count how often a word in each sentence appears, quite literally creating a bag of words. As a result, a bag-of-words model aims to create representations of text in the form of numbers, also called vectors or vector representations, observed in Figure1-5. Throughout the book, we refer to these kinds of models as representation models.
- Although bag-of-words is a classic method, it is by no means completely obsolete. In Chapter5, we will explore how it can still be used to complement more recent language models.

## Better Representations with Dense Vector Embeddings

- Bag-of-words, although an elegant approach, has a flaw. It considers language to be nothing more than an almost literal bag of words and ignores the semantic nature, or meaning, of text.
- Embeddings are vector representations of data that attempt to capture its meaning.
- To do so, word2vec learns semantic representations of words by training on vast amounts of textual data, like the entirety of Wikipedia.
- To generate these semantic representations, word2vec leverages neural networks. These networks consist of interconnected layers of nodes that process information
- neural networks can have many layers where each connection has a certain weight depending on the input. These weights are often referred to as the parameters of the model.
- During this training process, word2vec learns the relationship between words and distills that information into the embedding. If the two words tend to have the same neighbors, their embeddings will be closer to one another and vice versa
- In practice, these properties are often quite obscure and seldom relate to a single entity or humanly identifiable concept. However, together, these properties make sense to a computer and serve as a good way to translate human language into computer language.
- Embeddings are tremendously helpful as they allow us to measure the semantic similarity between two words. Using various distance metrics, we can judge how close one word is to another

## Types of Embeddings

- There are many types of embeddings, like word embeddings and sentence embeddings that are used to indicate different levels of abstractions (word versus sentence)
- Throughout the book, embeddings will take on a central role as they are utilized in many use cases, such as classification (see Chapter4), clustering (see Chapter5), and semantic search and retrieval-augmented generation (see Chapter8)

## Encoding and Decoding Context with Attention

- The training process of word2vec creates static, downloadable representations of words. For instance, the word “bank” will always have the same embedding regardless of the context in which it is used. However, “bank” can refer to both a financial bank as well as the bank of a river. Its meaning, and therefore its embeddings, should change depending on the context.
- A step in encoding this text was achieved through recurrent neural networks (RNNs).
- To do so, these RNNs are used for two tasks, encoding or representing an input sentence and decoding or generating an output sentence.
- Each step in this architecture is autoregressive. When generating the next word, this architecture needs to consume all previously generated words, as shown in Figure1-12.
- This context embedding, however, makes it difficult to deal with longer sentences since it is merely a single embedding representing the entire input. In 2014, a solution called attention was introduced that highly improved upon the original architecture
- Attention allows a model to focus on parts of the input sequence that are relevant to one another (“attend” to each other) and amplify their signal, as shown in Figure1-14. Attention selectively determines which words are most important in a given sentence.
- For instance, the output word “lama’s” is Dutch for “llamas,” which is why the attention between both is high. Similarly, the words “lama’s” and “I” have lower attention since they aren’t as related. In Chapter3, we will go more in depth on the attention mechanism.
- By adding these attention mechanisms to the decoder step, the RNN can generate signals for each input word in the sequence related to the potential output. Instead of passing only a context embedding to the decoder, the hidden states of all input words are passed. This process is demonstrated in Figure1-15.
- After generating the words “Ik,” “hou,” and “van,” the attention mechanism of the decoder enables it to focus on the word “llamas” before it generates the Dutch translation (“lama’s”).
- Compared to word2vec, this architecture allows for representing the sequential nature of text and the context in which it appears by “attending” to the entire sentence. This sequential nature, however, precludes parallelization during training of the model.

## Attention Is All You Need

- Transformer, which was solely based on the attention mechanism and removed the recurrence network that we saw previously
- Compared to the recurrence network, the Transformer could be trained in parallel, which tremendously sped up training.
- In the Transformer, encoding and decoder components are stacked on top of each other, as illustrated in Figure1-16. This architecture remains autoregressive, needing to consume each generated word before creating a new word.
- The Transformer is a combination of stacked encoder and decoder blocks where the input flows through each encoder and decoder.
- Transformer consists of two parts, self-attention and a feedforward neural network
- Compared to previous methods of attention, self-attention can attend to different positions within a single sequence, thereby more easily and accurately representing the input sequence as illustrated in Figure1-18. Instead of processing one token at a time, it can be used to look at the entire sequence in one go.
- Compared to the encoder, the decoder has an additional layer that pays attention to the output of the encoder (to find the relevant parts of the input). As demonstrated in Figure1-19, this process is similar to the RNN attention decoder that we discussed previously.
- In Chapters 2 and 3, we will go through the many reasons why Transformer models work so well, including multi-head attention, positional embeddings, and layer normalization.

## Representation Models: Encoder-Only Models

- In 2018, a new architecture called Bidirectional Encoder Representations from Transformers (BERT) was introduced that could be leveraged for a wide variety of tasks and would serve as the foundation of Language AI for years to come
- BERT is an encoder-only architecture that focuses on representing language, as illustrated in Figure1-21. This means that it only uses the encoder and removes the decoder entirely.
- BERT-like models are commonly used for transfer learning, which involves first pretraining it for language modeling and then fine-tuning it for a specific task. For instance, by training BERT on the entirety of Wikipedia, it learns to understand the semantic and contextual nature of text
- A huge benefit of pretrained models is that most of the training is already done for us. Fine-tuning on specific tasks is generally less compute-intensive and requires less data.
- Encoder-only models, like BERT, will be used in many parts of the book. For years, they have been and are still used for common tasks, including classification tasks (see Chapter4), clustering tasks (see Chapter5), and semantic search (see Chapter8).
- Throughout the book, we will refer to encoder-only models as representation models to differentiate them from decoder-only, which we refer to as generative models
- Representation models mainly focus on representing language, for instance, by creating embeddings, and typically do not generate text. In contrast, generative models focus primarily on generating text and typically are not trained to generate embeddings.
- Representation models are teal with a small vector icon (to indicate its focus on vectors and embeddings) whilst generative models are pink with a small chat icon (to indicate its generative capabilities).

## Generative Models: Decoder-Only Models

- These generative decoder-only models, especially the “larger” models, are commonly referred to as large language models (LLMs). As we will discuss later in this chapter, the term LLM is not only reserved for generative models (decoder-only) but also representation models (encoder-only).
- A vital part of these completion models is something called the context length or context window. The context length represents the maximum number of tokens the model can process,

## The Training Paradigm of Large Language Models

- Creating LLMs, in contrast, typically consists of at least two steps:
  - Language modeling
    - The first step, called pretraining, takes the majority of computation and training time. An LLM is trained on a vast corpus of internet text allowing the model to learn grammar, context, and language patterns. This broad training phase is not yet directed toward specific tasks or applications beyond predicting the next word. The resulting model is often referred to as a foundation model or base model. These models generally do not follow instructions.
  - Fine-tuning
    - The second step, fine-tuning or sometimes post-training, involves using the previously trained model and further training it on a narrower task. This allows the LLM to adapt to specific tasks or to exhibit desired behavior.
- Any model that goes through the first step, pretraining, we consider a pretrained model, which also includes fine-tuned models. 

## Large Language Model Applications: What Makes Them So Useful?

- Detecting whether a review left by a customer is positive or negative
- Developing a system for finding common topics in ticket issues
- Building a system for retrieval and inspection of relevant documents
- Constructing an LLM chatbot that can leverage external resources, such as tools and documents
- Constructing an LLM capable of writing recipes based on a picture showing the products in your fridge
- LLM applications are incredibly satisfying to create since they are partially bounded by the things you can imagine. As these models grow more accurate, using them in practice for creative use cases such as role-playing and writing children’s books simply becomes more and more fun.