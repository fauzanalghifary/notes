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
