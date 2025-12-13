# Chapter 3. Looking Inside Large Language Models

## The Inputs and Outputs of a Trained Transformer LLM

- The most common picture of understanding the behavior of a Transformer LLM is to think of it as a software system that takes in text and generates text in response. Once a large enough text-in-text-out model is trained on a large enough high-quality dataset, it becomes able to generate impressive and useful outputs
- The model does not generate the text all in one operation; it actually generates one token at a time
- After each token generation, we tweak the input prompt for the next generation step by appending the output token to the end of the input prompt
- This gives us a more accurate picture of the model as it is simply predicting the next token based on an input prompt. Software around the neural network basically runs it in a loop to sequentially expand the generated text until completion.

## The Components of the Forward Pass

- 