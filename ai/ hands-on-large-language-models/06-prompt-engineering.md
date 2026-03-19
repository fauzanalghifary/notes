# Chapter 6. Prompt Engineering

- Through prompt engineering, we can design these prompts in a way that enhances the quality of the generated text.

## Choosing a Text Generation Model

- Choosing a text generation model starts with choosing between proprietary models or open source models. Although proprietary models are generally more performant, we focus in this book more on open source models as they offer more flexibility and are free to use.

## Controlling Model Output

- A part of what makes LLMs exciting technology is that it can generate different responses for the exact same prompt. Each time an LLM needs to generate a token, it assigns a likelihood number to each possible token.
- The temperature controls the randomness or creativity of the text generated. It defines how likely it is to choose tokens that are less probable. The underlying idea is that a temperature of 0 generates the same response every time because it always chooses the most likely word.
- top_p, also known as nucleus sampling, is a sampling technique that controls which subset of tokens (the nucleus) the LLM can consider. It will consider tokens until it reaches their cumulative probability. If we set top_p to 0.1, it will consider tokens until it reaches that value. If we set top_p to 1, it will consider all tokens.
  - by lowering the value, it will consider fewer tokens and generally give less “creative” output, while increasing the value allows the LLM to choose from more tokens.
- Similarly, the top_k parameter controls exactly how many tokens the LLM can consider. If you change its value to 100, the LLM will only consider the top 100 most probable tokens.

## Intro to Prompt Engineering

- Prompt engineering is more than designing effective prompts. It can be used as a tool to evaluate the output of a model as well as to design safeguards and safety mitigation methods. This is an iterative process of prompt optimization and requires experimentation. There is not and unlikely will ever be a perfect prompt design.
- there is actually a lot of overlap in the prompting techniques used to improve the quality of the output. A non-exhaustive list of these techniques includes:
  - Specificity
  - Hallucination
  - Order
    - Either begin or end your prompt with the instruction.
- Here, specificity is arguably the most important aspect. By restricting and specifying what the model should generate, there is a smaller chance of having it generate something not related to your use case. For instance, if we were to skip the instruction “in two to three sentences” it might generate complete paragraphs. Like human conversations, without any specific instructions or additional context, it is difficult to derive what the task at hand actually is.

## The Potential Complexity of a Prompt

- These advanced components can quickly make a prompt quite complex. Some common components are:
  - Persona
  - Instruction
  - Context
  - Format
  - Audience
  - Tone
  - Data

## In-Context Learning: Providing Examples

- Instead of describing the task, why do we not just show the task?
- We can provide the LLM with examples of exactly the thing that we want to achieve. This is often referred to as in-context learning, where we provide the model with correct examples.
- we believe that “an example is worth a thousand words.” These examples provide a direct example of what and how the LLM should achieve.

## Chain Prompting: Breaking up the Problem

- Instead of breaking the problem within a prompt, we can do so between prompts. Essentially, we take the output of one prompt and use it as input for the next, thereby creating a continuous chain of interactions that solves our problem.
- This can be used for a variety of use cases, including:
  - Response validation
  - Parallel prompts
  - Writing stories

## Reasoning with Generative Models

- Reasoning is a core component of human intelligence and is often compared to the emergent behavior of LLMs that often resembles reasoning. We highlight “resemble” as these models, at the time of writing, are generally considered to demonstrate this behavior through memorization of training data and pattern matching.
- In other words, we work together with the LLM through prompt engineering so we can mimic reasoning processes in order to improve the output of the LLM.

## Chain-of-Thought: Think Before Answering

- The first and major step toward complex reasoning in generative models was through a method called chain-of-thought. Chain-of-thought aims to have the generative model “think” first rather than answering the question directly without any reasoning.
- Adding this reasoning step allows the model to distribute more compute over the reasoning process. Instead of calculating the entire solution based on a few tokens, each additional token in this reasoning process allows the LLM to stabilize its output.
- Although chain-of-thought is a great method for enhancing the output of a generative model, it does require one or more examples of reasoning in the prompt, which the user might not have access to. Instead of providing examples, we can simply ask the generative model to provide the reasoning (zero-shot chain-of-thought). There are many different forms that work but a common and effective method is to use the phrase “Let’s think step-by-step,”

## Self-Consistency: Sampling Outputs

- To counteract this degree of randomness and improve the performance of generative models, self-consistency was introduced. This method asks the generative model the same prompt multiple times and takes the majority result as the final answer.9 During this process, each answer can be affected by different temperature and top_p values to increase the diversity of sampling.
- However, this does require a single question to be asked multiple times. As a result, although the method can improve performance, it becomes n times slower where n is the number of output samples.

## Tree-of-Thought: Exploring Intermediate Steps

- The ideas of chain-of-thought and self-consistency are meant to enable more complex reasoning. By sampling from multiple “thoughts” and making them more thoughtful, we aim to improve the output of generative models.
- These techniques only scratch the surface of what is currently being done to mimic complex reasoning. An improvement to these approaches can be found in tree-of-thought, which allows for an in-depth exploration of several ideas.
- When faced with a problem that requires multiple reasoning steps, it often helps to break it down into pieces. At each step, and as illustrated in Figure6-18, the generative model is prompted to explore different solutions to the problem at hand. It then votes for the best solution and continues to the next step

## Output Verification

- Reasons for validating the output might include:
  - Structured output
  - Valid output
  - Ethics
  - Accuracy
- Generally, there are three ways of controlling the output of a generative model:
  - Examples 
    - Provide a number of examples of the expected output. 
  - Grammar 
    - Control the token selection process. 
  - Fine-tuning
    - Tune a model on data that contains the expected output.

## Grammar: Constrained Sampling

- Few-shot learning has a big disadvantage: we cannot explicitly prevent certain output from being generated. Although we guide the model and give it instructions, it might still not follow it entirely.
- This process can be taken one step further and instead of validating the output we can already perform validation during the token sampling process. When sampling tokens, we can define a number of grammars or rules that the LLM should adhere to when choosing its next token.