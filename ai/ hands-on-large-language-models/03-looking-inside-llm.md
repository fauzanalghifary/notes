# Chapter 3. Looking Inside Large Language Models

## The Inputs and Outputs of a Trained Transformer LLM

- The most common picture of understanding the behavior of a Transformer LLM is to think of it as a software system that takes in text and generates text in response. Once a large enough text-in-text-out model is trained on a large enough high-quality dataset, it becomes able to generate impressive and useful outputs
- The model does not generate the text all in one operation; it actually generates one token at a time
- After each token generation, we tweak the input prompt for the next generation step by appending the output token to the end of the input prompt
- This gives us a more accurate picture of the model as it is simply predicting the next token based on an input prompt. Software around the neural network basically runs it in a loop to sequentially expand the generated text until completion.

## The Components of the Forward Pass

- In addition to the loop, two key internal components are the tokenizer and the language modeling head (LM head)
- The tokenizer is followed by the neural network: a stack of Transformer blocks that do all of the processing. That stack is then followed by the LM head, which translates the output of the stack into probability scores for what the most likely next token is.
- the tokenizer contains a table of tokens—the tokenizer’s vocabulary. The model has a vector representation associated with each of these tokens in the vocabulary (token embeddings).
- For each generated token, the process flows once through each of the Transformer blocks in the stack in order, then to the LM head, which finally outputs the probability distribution for the next token
- The LM head is a simple neural network layer itself. 

## Choosing a Single Token from the Probability Distribution (Sampling/Decoding)

- The method of choosing a single token from the probability distribution is called the decoding strategy
- The easiest decoding strategy would be to always pick the token with the highest probability score. In practice, this doesn’t tend to lead to the best outputs for most use cases. A better approach is to add some randomness and sometimes choose the second or third highest probability token. The idea here is to basically sample from the probability distribution based on the probability score, as the statisticians would say.
- Choosing the highest scoring token every time is called greedy decoding. It’s what happens if you set the temperature parameter to zero in an LLM

## Parallel Token Processing and Context Size

- One of the most compelling features of Transformers is that they lend themselves better to parallel computing than previous neural network architectures in language processing
- Current Transformer models have a limit for how many tokens they can process at once. That limit is called the model’s context length. A model with 4K context length can only process 4K tokens and would only have 4K of these streams.
- For text generation, only the output result of the last stream is used to predict the next token. That output vector is the only input into the LM head as it calculates the probabilities of the next token.
- You may wonder why we go through the trouble of calculating all the token streams if we’re discarding the outputs of all but the last token. The answer is that the calculations of the previous streams are required and used in calculating the final stream. Yes, we’re not using their final output vector, but we use earlier outputs (in each Transformer block) in the Transformer block’s attention mechanism.

## Speeding Up Generation by Caching Keys and Values

- Recall that when generating the second token, we simply append the output token to the input and do another forward pass through the model. If we give the model the ability to cache the results of the previous calculation (especially some of the specific vectors in the attention mechanism), we no longer need to repeat the calculations of the previous streams. This time the only needed calculation is for the last stream. This is an optimization technique called the keys and values (kv) cache and it provides a significant speedup of the generation process. Keys and values are some of the central components of the attention mechanism
- This is one reason why LLM APIs stream the output tokens as the model generates them instead of waiting for the entire generation to be completed.

## Inside the Transformer Block

- where the vast majority of processing happens
- A Transformer block (Figure3-12) is made up of two successive components:
  - The attention layer is mainly concerned with incorporating relevant information from other input tokens and positions 
  - The feedforward layer houses the majority of the model’s processing capacity

## The feedforward neural network at a glance

- Memorization is only one ingredient in the recipe of impressive text generation. The model is able to use this same machinery to interpolate between data points and more complex patterns to be able to generalize—which means doing well on inputs it hadn’t seen in the past and were not in its training dataset.
- This is why the language model is then trained on instruction-tuning and human preference and feedback fine-tuning to match people’s expectations of what the model should output.

## The attention layer at a glance

- Context is vital in order to properly model language. Simple memorization and interpolation based on the previous token can only take us so far. 
- Attention is a mechanism that helps the model incorporate context as it’s processing a specific token

## Attention is all you need

- The attention mechanism operates on the input vector at that position. It incorporates relevant information from the context into the vector it produces as the output for that position.
- Two main steps are involved in the attention mechanism:
  - A way to score how relevant each of the previous input tokens are to the current token being processed (in the pink arrow). 
  - Using those scores, we combine the information from the various positions into a single output vector.
- To give the Transformer more extensive attention capability, the attention mechanism is duplicated and executed multiple times in parallel

## How attention is calculated

- Before we start the calculation, let’s observe the following as the starting position:
  - The attention layer (of a generative LLM) is processing attention for a single position.
  - The inputs to the layer are:
    - The vector representation of the current position or token 
    - The vector representations of the previous tokens
  - The goal is to produce a new representation of the current position that incorporates relevant information from the previous tokens:
  - The training process produces three projection matrices that produce the components that interact in this calculation:
    - A query projection matrix 
    - A key projection matrix 
    - A value projection matrix
- Attention starts by multiplying the inputs by the projection matrices to create three new matrices. These are called the queries, keys, and values matrices. These matrices contain the information of the input tokens projected to three different spaces that help carry out the two steps of attention:
  - Relevance scoring 
  - Combining information

## Self-attention: Relevance scoring

- In a generative Transformer, we’re generating one token at a time. This means we’re processing one position at a time. So the attention mechanism here is only concerned with this one position, and how information from other positions can be pulled in to inform this position.
- The relevance scoring step of attention is conducted by multiplying the query vector of the current position with the keys matrix. This produces a score stating how relevant each previous token is. Passing that by a softmax operation normalizes these scores so they sum up to 1

## Self-attention: Combining information

- Now that we have the relevance scores, we multiply the value vector associated with each token by that token’s score. Summing up those resulting vectors produces the output of this attention step

## Recent Improvements to the Transformer Architecture

- 