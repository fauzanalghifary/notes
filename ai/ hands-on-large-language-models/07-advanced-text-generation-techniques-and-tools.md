# Chapter 7. Advanced Text Generation Techniques and Tools

- Fortunately, a great deal of methods and techniques allow us to further improve what we started with in the previous chapter. These more advanced techniques lie at the foundation of numerous LLM-focused systems and are, arguably, one of the first things users implement when designing such systems.
- In this chapter, we will explore several such methods and concepts for improving the quality of the generated text:
  - Model I/O
  - Memory
  - Agents: Combining complex behavior with external tools
  - Chains
- Each of these techniques has significant strengths by themselves but their true value does not exist in isolation. It is when you combine all of these techniques that you get an LLM-based system with incredible performance

## Model I/O: Loading Quantized Models with LangChain

- Quantization is a similar process that reduces the precision of a value (e.g., removing seconds) without removing vital information (e.g., retaining hours and minutes).

## Chains: Extending the Capabilities of LLMs

- LangChain is named after one of its main methods, chains. Although we can run LLMs in isolation, their power is shown when used with additional components or even when used in conjunction with each other. Chains not only allow for extending the capabilities of LLMs but also for multiple chains to be connected together.

## A Single Link in the Chain: Prompt Template

- The idea, as illustrated in Figure7-4, is that we chain the prompt template together with the LLM to get the output we are looking for. Instead of having to copy-paste the prompt template each time we use the LLM, we would only need to define the user and system prompts.
- Adding a prompt template to the chain is just the very first step you need to enhance the capabilities of your LLM. Throughout this chapter, we will see many ways in which we can add additional modular components to existing chains, starting with memory.

## A Chain with Multiple Prompts

- Instead, we could break this complex prompt into smaller subtasks that can be run sequentially. This would require multiple calls to the LLM but with smaller prompts and intermediate outputs
- Instead of using a single chain, we link chains where each link deals with a specific subtask.
- Another advantage of dividing the problem into smaller tasks is that we now have access to these individual components. We can easily extract the title; that might not have been the case if we were to use a single prompt.

## Memory: Helping LLMs to Remember Conversations

- To make these models stateful, we can add specific types of memory to the chain that we created earlier. 
- In this section, we will go through two common methods for helping LLMs to remember conservations:
  - Conversation buffer 
  - Conversation summary

## Conversation Buffer

- One of the most intuitive forms of giving LLMs memory is simply reminding them exactly what has happened in the past
- we can achieve this by copying the full conversation history and pasting that into our prompt.

## Windowed Conversation Buffer

- One method of minimizing the context window is to use the last k conversations instead of maintaining the full chat history. In LangChain, we can use ConversationBufferWindowMemory to decide how many conversations are passed to the input prompt
- Although this method reduces the size of the chat history, it can only retain the last few conversations, which is not ideal for lengthy conversations. Let’s explore how we can summarize the chat history instead.

## Conversation Summary

- As the name implies, this technique summarizes an entire conversation history to distill it into the main points.
- This summarization process is enabled by another LLM that is given the conversation history as input and asked to create a concise summary. A nice advantage of using an external LLM is that we are not confined to using the same LLM during conversation.
- This summarization helps keep the chat history relatively small without using too many tokens during inference. However, since the original question was not explicitly saved in the chat history, the model needed to infer it based on the context. This is a disadvantage if specific information needs to be stored in the chat history. Moreover, multiple calls to the same LLM are needed, one for the prompt and one for the summarization. This can slow down computing time.

## Agents: Creating a System of LLMs

- One of the most promising concepts in LLMs is their ability to determine the actions they can take. This idea is often called agents, systems that leverage a language model to determine which actions they should take and in what order.
- Agents can make use of everything we have seen thus far, such as model I/O, chains, and memory, and extend it further with two vital components:
  - Tools that the agent can use to do things it could not do itself 
  - The agent type, which plans the actions to take or tools to use
- Unlike the chains we have seen thus far, agents are able to show more advanced behavior like creating and self-correcting a roadmap to achieve a goal. They can interact with the real world through the use of tools. As a result, these agents can perform a variety of tasks that go beyond what an LLM is capable of in isolation.
- Suddenly, the capabilities of LLMs increase significantly.
- Although the tools they use are important, the driving force of many agent-based systems is the use of a framework called Reasoning and Acting (ReAct1)

## The Driving Power Behind Agents: Step-by-step Reasoning

- ReAct is a powerful framework that combines two important concepts in behavior: reasoning and acting. LLMs are exceptionally powerful when it comes to reasoning as we explored in detail in Chapter5.
- To give them the ability to act, we could tell an LLM that it can use certain tools
- ReAct merges these two concepts and allows reasoning to affect acting and actions to affect reasoning. In practice, the framework consists of iteratively following these three steps:
  - Thought 
  - Action 
  - Observation
- During this process, the agent describes its thoughts (what it should do), its actions (what it will do), and its observations (the results of the action). It is a cycle of thoughts, actions, and observations that results in the agent’s output.
- With this foundation in place, we are now poised to explore ways in which LLMs can be used to improve existing search systems and even become the core of new, more powerful search systems, as discussed in the next chapter.