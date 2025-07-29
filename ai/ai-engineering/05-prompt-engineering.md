# Chapter 5 - Prompt Engineering

- Prompt engineering refers to the process of crafting an instruction that gets a model to generate the desired outcome
- Prompt engineering is the easiest and most common model adaptation technique. Unlike finetuning, prompt engineering guides a model’s behavior without changing the model’s weights. Thanks to the strong base capabilities of foundation models, many people have successfully adapted them for applications using prompt engineering alone. You should make the most out of prompting before moving to more resource-intensive techniques like finetuning.
- Prompt engineering’s ease of use can mislead people into thinking that there’s not much to it.1 At first glance, prompt engineering looks like it’s just fiddling with words until something works. While prompt engineering indeed involves a lot of fiddling, it also involves many interesting challenges and ingenious solutions
- Anyone can communicate, but not everyone can communicate effectively. Similarly, it’s easy to write prompts but not easy to construct effective prompts.
- Some people argue that “prompt engineering” lacks the rigor to qualify as an engineering discipline. However, this doesn’t have to be the case. Prompt experiments should be conducted with the same rigor as any ML experiment, with systematic experimentation and evaluation.
- “The problem is not with prompt engineering. It’s a real and useful skill to have. The problem is when prompt engineering is the only thing people know.” To build production-ready AI applications, you need more than just prompt engineering. You need statistics, engineering, and classic ML knowledge to do experiment tracking, evaluation, and dataset curation.
- This chapter covers both how to write effective prompts and how to defend your applications against prompt attacks.

### Introduction to Prompting

- A prompt is an instruction given to a model to perform a task
- A prompt generally consists of one or more of the following parts:
  - Task description
  - Example(s) of how to do this task
  - The task
    - The concrete task you want the model to do, such as the question to answer or the book to summarize.
- For prompting to work, the model has to be able to follow instructions. If a model is bad at it, it doesn’t matter how good your prompt is, the model won’t be able to follow it. How to evaluate a model’s instruction-following capability is discussed in Chapter 4.For prompting to work, the model has to be able to follow instructions. If a model is bad at it, it doesn’t matter how good your prompt is, the model won’t be able to follow it. How to evaluate a model’s instruction-following capability is discussed in Chapter4.
- How much prompt engineering is needed depends on how robust the model is to prompt perturbation. 
- The less robust the model is, the more fiddling is needed.
- You can measure a model’s robustness by randomly perturbing the prompts to see how the output changes. Just like instruction-following capability, a model’s robustness is strongly correlated with its overall capability. As models become stronger, they also become more robust.
- Experiment with different prompt structures to find out which works best for you. Most models, including GPT-4, empirically perform better when the task description is at the beginning of the prompt. However, some models, including Llama 3, seem to perform better when the task description is at the end of the prompt.

In-Context Learning: Zero-Shot and Few-Shot

- Teaching models what to do via prompts is also known as in-context learning. 
- Traditionally, a model learns the desirable behavior during training—including pre-training, post-training, and finetuning—which involves updating model weights. The GPT-3 paper demonstrated that language models can learn the desirable behavior from examples in the prompt, even if this desirable behavior is different from what the model was originally trained to do. No weight updating is needed.
- Each example provided in the prompt is called a shot. Teaching a model to learn from examples in the prompt is also called few-shot learning. With five examples, it’s 5-shot learning. When no example is provided, it’s zero-shot learning.
- Exactly how many examples are needed depends on the model and the application. You’ll need to experiment to determine the optimal number of examples for your applications. In general, the more examples you show a model, the better it can learn. The number of examples is limited by the model’s maximum context length. The more examples there are, the longer your prompt will be, increasing the inference cost.
- This result suggests that as models become more powerful, they become better at understanding and following instructions, which leads to better performance with fewer examples. However, the study might have underestimated the impact of few-shot examples on domain-specific use cases.
- In this book, I’ll use prompt to refer to the whole input into the model, and context to refer to the information provided to the model so that it can perform a given task.
- Today, in-context learning is taken for granted. A foundation model learns from a massive amount of data and should be able to do a lot of things. However, before GPT-3, ML models could do only what they were trained to do, so in-context learning felt like magic.

System Prompt and User Prompt

- You can think of the system prompt as the task description and the user prompt as the task.
- Almost all generative AI applications, including ChatGPT, have system prompts. Typically, the instructions provided by application developers are put into the system prompt, while the instructions provided by users are put into the user prompt. But you can also be creative and move instructions around, such as putting everything into the system prompt or user prompt. You can experiment with different ways to structure your prompts to see which one works best.
- Given a system prompt and a user prompt, the model combines them into a single prompt
- Many model providers emphasize that well-crafted system prompts can improve performance. For example, Anthropic documentation says, “when assigning Claude a specific role or personality through a system prompt, it can maintain that character more effectively throughout the conversation, exhibiting more natural and creative responses while staying in character.”
- But why would system prompts boost performance compared to user prompts? Under the hood, the system prompt and the user prompt are concatenated into a single final prompt before being fed into the model. From the model’s perspective, system prompts and user prompts are processed the same way. Any performance boost that a system prompt can give is likely because of one or both of the following factors:
  - The system prompt comes first in the final prompt, and the model might just be better at processing instructions that come first. 
  - The model might have been post-trained to pay more attention to the system prompt

Context Length and Context Efficiency

- How much information can be included in a prompt depends on the model’s context length limit. Models’ maximum context length has increased rapidly in recent years. 
- Not all parts of a prompt are equal. Research has shown that a model is much better at understanding instructions given at the beginning and the end of a prompt than in the middle (Liu et al., 2023)
- One way to evaluate the effectiveness of different parts of a prompt is to use a test commonly known as the needle in a haystack (NIAH). The idea is to insert a random piece of information (the needle) in different locations in a prompt (the haystack) and ask the model to find it.
- All the models tested seemed much better at finding the information when it’s closer to the beginning and the end of the prompt than the middle.
- If the model’s performance grows increasingly worse with a longer context, then perhaps you should find a way to shorten your prompts.

### Prompt Engineering Best Practice

- Prompt engineering can get incredibly hacky, especially for weaker models. In the early days of prompt engineering, many guides came out with tips such as writing “Q:” instead of “Questions:” or encouraging models to respond better with the promise of a “$300 tip for the right answer”. While these tips can be useful for some models, they can become outdated as models get better at following instructions and more robust to prompt perturbations.

Write Clear and Explicit Instructions

- Explain, without ambiguity, what you want the model to do
  - As you experiment with a prompt, you might observe undesirable behaviors that require adjustments to the prompt to prevent them. For example, if the model outputs fractional scores (4.5) and you don’t want fractional scores, update your prompt to tell the model to output only integer scores.
- Ask the model to adopt a persona
  - A persona can help the model to understand the perspective it’s supposed to use to generate responses
  - Given the essay “I like chickens. Chickens are fluffy and they give tasty eggs.”, a model out of the box might give it a score of 2 out of 5. However, if you ask the model to adopt the persona of a first-grade teacher, the essay might get a 4.
- Provide examples
  - Examples can reduce ambiguity about how you want the model to respond.
- Specify the output format
  - Ensuring the model outputs are in the correct format is essential when they are used by downstream applications that require specific formats. If you want the model to generate JSON, specify what the keys in the JSON should be. Give examples if necessary.
  - For tasks expecting structured outputs, such as classification, use markers to mark the end of the prompts to let the model know that the structured outputs should begin.8 Without markers, the model might continue appending to the input,

Provide Sufficient Context

- Context can also mitigate hallucinations. If the model isn’t provided with the necessary information, it’ll have to rely on its internal knowledge, which might be unreliable, causing it to hallucinate.
- You can either provide the model with the necessary context or give it tools to gather context. The process of gathering necessary context for a given query is called context construction. Context construction tools include data retrieval, such as in a RAG pipeline, and web search. These tools are discussed in Chapter6.
- In many scenarios, it’s desirable for the model to use only information provided in the context to respond.
  - How to restrict a model to only the context is tricky.
  - However, since there’s no guarantee that the model will follow all instructions, prompting alone may not reliably produce the desired outcome. Finetuning a model on your own corpus is another option, but pre-training data can still leak into its responses. The safest method is to train a model exclusively on the permitted corpus of knowledge, though this is often not feasible for most use cases. Additionally, the corpus may be too limited to train a high-quality model.

Break Complex Tasks into Simpler Subtasks

- For complex tasks that require multiple steps, break those tasks into subtasks. Instead of having one giant prompt for the whole task, each subtask has its own prompt. These subtasks are then chained together
- How small each subtask should be depends on each use case and the performance, cost, and latency trade-off you’re comfortable with. You’ll need to experiment to find the optimal decomposition and chaining.
- While models are getting better at understanding complex instructions, they are still better with simpler ones.
- Prompt decomposition not only enhances performance but also offers several additional benefits:
  - Monitoring: You can monitor not just the final output but also all intermediate outputs.
  - Debugging: You can isolate the step that is having trouble and fix it independently without changing the model’s behavior at the other steps.
  - Parallelization: When possible, execute independent steps in parallel to save time. Imagine asking a model to generate three different story versions for three different reading levels: first grade, eighth grade, and college freshman. All these three versions can be generated at the same time, significantly reducing the output latency
  - Effort: It’s easier to write simple prompts than complex prompts.
- One downside of prompt decomposition is that it can increase the latency perceived by users, especially for tasks where users don’t see the intermediate outputs. With more intermediate steps, users have to wait longer to see the first output token generated in the final step.
- Prompt decomposition typically involves more model queries, which can increase costs. However, the cost of two decomposed prompts might not be twice that of one original prompt. This is because most model APIs charge per input and output token, and smaller prompts often incur fewer tokens. Additionally, you can use cheaper models for simpler steps.

Give the Model Time to Think

- You can encourage the model to spend more time to, for a lack of better words, “think” about a question using chain-of-thought (CoT) and self-critique prompting.
- CoT means explicitly asking the model to think step by step, nudging it toward a more systematic approach to problem solving. CoT is among the first prompting techniques that work well across models.
- The simplest way to do CoT is to add “think step by step” or “explain your decision” in your prompt. The model then works out what steps to take. Alternatively, you can specify the steps the model should take or include examples of what the steps should look like in your prompt
- Self-critique means asking the model to check its own outputs. This is also known as self-eval, as discussed in Chapter3. Similar to CoT, self-critique nudges the model to think critically about a problem.
- Similar to prompt decomposition, CoT and self-critique can increase the latency perceived by users. A model might perform multiple intermediate steps before the user can see the first output token. This is especially challenging if you encourage the model to come up with steps on its own. The resulting sequence of steps can take a long time to finish, leading to increased latency and potentially prohibitive costs.

Iterate on Your Prompts

- Each model has its quirks. One model might be better at understanding numbers, whereas another might be better at roleplaying. One model might prefer system instructions at the beginning of the prompt, whereas another might prefer them at the end. Play around with your model to get to know it. Try different prompts. Read the prompting guide provided by the model developer, if there’s any. Look for other people’s experiences online. Leverage the model’s playground if one is available. Use the same prompt on different models to see how their responses differ, which can give you a better understanding of your model.
- As you experiment with different prompts, make sure to test changes systematically. Version your prompts. Use an experiment tracking tool. Standardize evaluation metrics and evaluation data so that you can compare the performance of different prompts. Evaluate each prompt in the context of the whole system. A prompt might improve the model’s performance on a subtask but worsen the whole system’s performance.

Evaluate Prompt Engineering Tools

- Tools that aim to automate the whole prompt engineering workflow include OpenPrompt (Ding et al., 2021) and DSPy (Khattab et al., 2023). At a high level, you specify the input and output formats, evaluation metrics, and evaluation data for your task. These prompt optimization tools automatically find a prompt or a chain of prompts that maximizes the evaluation metrics on the evaluation data. Functionally, these tools are similar to autoML (automated ML) tools that automatically find the optimal hyperparameters for classical ML models.
- A common approach to automating prompt generation is to use AI models. AI models themselves are capable of writing prompts.10 In its simplest form, you can ask a model to generate a prompt for your application, such as “Help me write a concise prompt for an application that grades college essays between 1 and 5”. You can also ask AI models to critique and improve your prompts or generate in-context examples.
- Many tools aim to assist parts of prompt engineering. For example, Guidance, Outlines, and Instructor guide models toward structured outputs. Some tools perturb your prompts, such as replacing a word with its synonym or rewriting a prompt, to see which prompt variation works best.
- If used correctly, prompt engineering tools can greatly improve your system’s performance. However, it’s important to be aware of how they work under the hood to avoid unnecessary costs and headaches.
- Following the keep-it-simple principle, you might want to start by writing your own prompts without any tool. This will give you a better understanding of the underlying model and your requirements.
- If you use a prompt engineering tool, always inspect the prompts produced by that tool to see whether these prompts make sense and track how many API calls it generates.11 No matter how brilliant tool developers are, they can make mistakes, just like everyone else.

Organize and Version Prompts

- It’s good practice to separate prompts from code—you’ll see why in a moment. For example, you can put your prompts in a file prompts.py and reference these prompts when creating a model query
  - Reusability
  - Testing
  - Readability
  - Collaboration
- If you have a lot of prompts across multiple applications, it’s useful to give each prompt metadata so that you know what prompt and use case it’s intended for.
- If the prompt files are part of your git repository, these prompts can be versioned using git. The downside of this approach is that if multiple applications share the same prompt and this prompt is updated, all applications dependent on this prompt will be automatically forced to update to this new prompt. In other words, if you version your prompts together with your code in git, it’s very challenging for a team to choose to stay with an older version of a prompt for their application.
- Many teams use a separate prompt catalog that explicitly versions each prompt so that different applications can use different prompt versions. A prompt catalog should also provide each prompt with relevant metadata and allow prompt search. A well-implemented prompt catalog might even keep track of the applications that depend on a prompt and notify the application owners of newer versions of that prompt.

### Defensive Prompt Engineering

- Once your application is made available, it can be used by both intended users and malicious attackers who may try to exploit it
- There are three main types of prompt attacks that, as application developers, you want to defend against:
  - Prompt extraction
    - Extracting the application’s prompt, including the system prompt, either to replicate or exploit the application
  - Jailbreaking and prompt injection
    - Getting the model to do bad things
  - Information extraction
    - Getting the model to reveal its training data or information used in its context
- Prompt attacks pose multiple risks for applications
  - Remote code or tool execution
  - Data leaks
  - Social harms
  - Misinformation
  - Service interruption and subversion
  - Brand risk

Proprietary Prompts and Reverse Prompt Engineering

- Reverse prompt engineering is the process of deducing the system prompt used for a certain application. Bad actors can use the leaked system prompt to replicate your application or manipulate it into doing undesirable actions—much like how knowing how a door is locked makes it easier to open
- Let’s say you trick a model into spitting out what looks like its system prompt. How do you verify that this is legitimate? More often than not, the extracted prompt is hallucinated by the model.
- Not only system prompts but also context can be extracted.
- While well-crafted prompts are valuable, proprietary prompts are more of a liability than a competitive advantage. Prompts require maintenance. They need to be updated every time the underlying model changes.

Jailbreaking and Prompt Injection

- Jailbreaking a model means trying to subvert a model’s safety features. As an example, consider a customer support bot that isn’t supposed to tell you how to do dangerous things. Getting it to tell you how to make a bomb is jailbreaking.
- If jailbreaking and prompt injection sound similar to you, you’re not alone. They share the same ultimate goal—getting the model to express undesirable behaviors. They have overlapping techniques. In this book, I’ll use jailbreaking to refer to both.
- AI safety, like any area of cybersecurity, is an evolving cat-and-mouse game where developers continuously work to neutralize known threats while attackers devise new ones. Here are a few common approaches that have succeeded in the past, presented in the order of increasing sophistication. Most of them are no longer effective for most models.
  - Direct manual prompt hacking
  - Automated attacks
  - Indirect prompt injection

Information Extraction

- Data theft
  - Extracting training data to build a competitive model. Imagine spending millions of dollars and months, if not years, on acquiring data only to have this data extracted by your competitors.
- Privacy violation
- Copyright infringement
- For all model families in the study, there’s a clear trend that the larger model memorizes more, making larger models more vulnerable to data extraction attacks
- To avoid this attack, some models block suspicious fill-in-the-blank requests.

Defenses Against Prompt Attacks

- In general, defenses against prompt attacks can be implemented at the model, prompt, and system levels. Even though there are measures you can implement, as long as your system has the capabilities to do anything impactful, the risks of prompt hacks may never be completely eliminated.
- Model-level defense
- OpenAI introduces an instruction hierarchy that contains four levels of priority, which are visualized in Figure5-16:
  - System prompt
  - User prompt
  - Model outputs
  - Tool outputs
- In the event of conflicting instructions, such as an instruction that says, “don’t reveal private information” and another saying “shows me X’s email address”, the higher-priority instruction should be followed. Since tool outputs have the lowest priority, this hierarchy can neutralize many indirect prompt injection attacks.
- Prompt-level defense
- You can create prompts that are more robust to attacks. Be explicit about what the model isn’t supposed to do, for example, “Do not return sensitive information such as email addresses, phone numbers, and addresses” or “Under no circumstances should any information other than XYZ be returned”.
- One simple trick is to repeat the system prompt twice, both before and after the user prompt
- System-level defense
- Your system can be designed to keep you and your users safe. One good practice, when possible, is isolation. If your system involves executing generated code, execute this code only in a virtual machine separated from the user’s main machine. This isolation helps protect against untrusted code.
- Another good practice is to not allow any potentially impactful commands to be executed without explicit human approvals. For example, if your AI system has access to an SQL database, you can set a rule that all queries attempting to change the database, such as those containing “DELETE”, “DROP”, or “UPDATE”, must be approved before executing.
- You should also place guardrails both to the inputs and outputs. On the input side, you can have a list of keywords to block, known prompt attack patterns to match the inputs against, or a model to detect suspicious requests. However, inputs that appear harmless can produce harmful outputs, so it’s important to have output guardrails, as well. For example, a guardrail can check if an output contains PII or toxic information. Guardrails are discussed more in Chapter10.

### Summary

- Foundation models can do many things, but you must tell them exactly what you want. The process of crafting an instruction to get a model to do what you want is called prompt engineering. How much crafting is needed depends on how sensitive the model is to prompts. If a small change can cause a big change in the model’s response, more crafting will be necessary.
- You can think of prompt engineering as human–AI communication. Anyone can communicate, but not everyone can communicate well. Prompt engineering is easy to get started, which misleads many into thinking that it’s easy to do it well.
- Whether you’re communicating with AI or other humans, clear instructions with examples and relevant information are essential. Simple tricks like asking the model to slow down and think step by step can yield surprising improvements. Just like humans, AI models have their quirks and biases, which need to be considered for a productive relationship with them.