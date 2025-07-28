# Chapter 4 - Evaluate AI System

- A model is only useful if it works for its intended purposes. You need to evaluate models in the context of your application
- Chapter3 discusses different approaches to automatic evaluation. This chapter discusses how to use these approaches to evaluate models for your applications.
- This chapter contains three parts. It starts with a discussion of the criteria you might use to evaluate your applications and how these criteria are defined and calculated. For example, many people worry about AI making up facts—how is factual consistency detected? How are domain-specific capabilities like math, science, reasoning, and summarization measured?
- The second part focuses on model selection. Given an increasing number of foundation models to choose from, it can feel overwhelming to choose the right model for your application. Thousands of benchmarks have been introduced to evaluate these models along different criteria. Can these benchmarks be trusted? How do you select what benchmarks to use? How about public leaderboards that aggregate multiple benchmarks?
- The last part discusses developing an evaluation pipeline that can guide the development of your application over time. This part brings together the techniques we’ve learned throughout the book to evaluate concrete applications.

### Evaluation Criteria 

- Which is worse—an application that has never been deployed or an application that is deployed but no one knows whether it’s working? When I asked this question at conferences, most people said the latter
- An application that is deployed but can’t be evaluated is worse. It costs to maintain, but if you want to take it down, it might cost even more.
- At the beginning of the ChatGPT fever, companies rushed to deploy customer support chatbots. Many of them are still unsure if these chatbots help or hurt their user experience.
- Before investing time, money, and resources into building an application, it’s important to understand how this application will be evaluated. I call this approach evaluation-driven development.
- In AI engineering, evaluation-driven development means defining evaluation criteria before building.
- Even though foundation models are open-ended, many of their use cases are close-ended, such as intent classification, sentiment analysis, next-action prediction, etc. It’s much easier to evaluate classification tasks than open-ended tasks.
- While the evaluation-driven development approach makes sense from a business perspective, focusing only on applications whose outcomes can be measured is similar to looking for the lost key under the lamppost (at night). It’s easier to do, but it doesn’t mean we’ll find the key. We might be missing out on many potentially game-changing applications because there is no easy way to evaluate them.
- I believe that evaluation is the biggest bottleneck to AI adoption. Being able to build reliable evaluation pipelines will unlock many new applications.
- An AI application, therefore, should start with a list of evaluation criteria specific to the application. In general, you can think of criteria in the following buckets: domain-specific capability, generation capability, instruction-following capability, and cost and latency.

Domain-Specific Capability

- A model’s domain-specific capabilities are constrained by its configuration (such as model architecture and size) and training data. If a model never saw Latin during its training process, it won’t be able to understand Latin. Models that don’t have the capabilities your application requires won’t work for you.
- To evaluate whether a model has the necessary capabilities, you can rely on domain-specific benchmarks, either public or private. Thousands of public benchmarks have been introduced to evaluate seemingly endless capabilities, including code generation, code debugging, grade school math, science knowledge, common sense, reasoning, legal knowledge, tool use, game playing, etc. The list goes on.
- Domain-specific capabilities are commonly evaluated using exact evaluation.
- Close-ended outputs are easier to verify and reproduce. For example, if you want to evaluate a model’s ability to do math, an open-ended approach is to ask the model to generate the solution to a given problem. A close-ended approach is to give the model several options and let it pick the correct one. If the expected answer is option C and the model outputs option A, the model is wrong.
- Despite the prevalence of close-ended benchmarks, it’s unclear if they are a good way to evaluate foundation models. MCQs test the ability to differentiate good responses from bad responses (classification), which is different from the ability to generate good responses. MCQs are best suited for evaluating knowledge (“does the model know that Paris is the capital of France?”) and reasoning (“can the model infer from a table of business expenses which department is spending the most?”). They aren’t ideal for evaluating generation capabilities such as summarization, translation, and essay writing. Let’s discuss how generation capabilities can be evaluated in the next section.

Generation Capability

- The subfield that studies open-ended text generation is called NLG (natural language generation). NLG tasks in the early 2010s included translation, summarization, and paraphrasing.
- Metrics used to evaluate the quality of generated texts back then included fluency and coherence. Fluency measures whether the text is grammatically correct and natural-sounding (does this sound like something written by a fluent speaker?). Coherence measures how well-structured the whole text is (does it follow a logical structure?)
- Some early NLG metrics, including faithfulness and relevance, have been repurposed, with significant modifications, to evaluate the outputs of foundation models. As generative models improved, many issues of early NLG systems went away, and the metrics used to track these issues became less important
- However, as language models’ generation capabilities have improved, AI-generated texts have become nearly indistinguishable from human-generated texts. Fluency and coherence become less important.
- Generative models, with their new capabilities and new use cases, have new issues that require new metrics to track. The most pressing issue is undesired hallucinations. Hallucinations are desirable for creative tasks, not for tasks that depend on factuality. A metric that many application developers want to measure is factual consistency. Another issue commonly tracked is safety: can the generated outputs cause harm to users and society? Safety is an umbrella term for all types of toxicity and biases.

Factual consistency
- The factual consistency of a model’s output can be verified under two settings: against explicitly provided facts (context) or against open knowledge
  - The output is considered factually consistent if it’s supported by the given context. For example, if the model outputs “the sky is blue” and the given context says that the sky is purple, this output is considered factually inconsistent.
  - Local factual consistency is important for tasks with limited scopes such as summarization (the summary should be consistent with the original document), customer support chatbots (the chatbot’s responses should be consistent with the company’s policies), and business analysis (the extracted insights should be consistent with the data).
  - The output is evaluated against open knowledge. If the model outputs “the sky is blue” and it’s a commonly accepted fact that the sky is blue, this statement is considered factually correct. Global factual consistency is important for tasks with broad scopes such as general chatbots, fact-checking, market research, etc.
- If no context is given, you’ll have to first search for reliable sources, derive facts, and then validate the statement against these facts.
- Often, the hardest part of factual consistency verification is determining what the facts are
- In addition, it’s easy to fall for the absence of evidence fallacy. One might take the statement “there’s no link between X and Y” as factually correct because of a failure to find the evidence that supported the link.
- existing “models rely heavily on the relevance of a website to the query, while largely ignoring stylistic features that humans find important such as whether a text contains scientific references or is written with a neutral tone.”
- When designing metrics to measure hallucinations, it’s important to analyze the model’s outputs to understand the types of queries that it is more likely to hallucinate on. Your benchmark should focus more on these queries.
- Let’s assume for now that you already have the context to evaluate an output against—this context was either provided by users or retrieved by you (context retrieval is discussed in Chapter6). The most straightforward evaluation approach is AI as a judge
- More sophisticated AI as a judge techniques to evaluate factual consistency are self-verification and knowledge-augmented verification:
  - SelfCheckGPT (Manakul et al., 2023) relies on an assumption that if a model generates multiple outputs that disagree with one another, the original output is likely hallucinated. Given a response R to evaluate, SelfCheckGPT generates N new responses and measures how consistent R is with respect to these N new responses. This approach works but can be prohibitively expensive, as it requires many AI queries to evaluate a response.
  - SAFE, Search-Augmented Factuality Evaluator, introduced by Google DeepMind (Wei et al., 2024) in the paper “Long-Form Factuality in Large Language Models”, works by leveraging search engine results to verify the response
- Verifying whether a statement is consistent with a given context can also be framed as textual entailment, which is a long-standing NLP task
- Textual entailment is the task of determining the relationship between two statements. Given a premise (context), it determines which category a hypothesis (the output or part of the output) falls into:
  - Entailment: the hypothesis can be inferred from the premise. 
  - Contradiction: the hypothesis contradicts the premise. 
  - Neutral: the premise neither entails nor contradicts the hypothesis.
- Entailment implies factual consistency, contradiction implies factual inconsistency, and neutral implies that consistency can’t be determined.
- Factual consistency is a crucial evaluation criteria for RAG, retrieval-augmented generation, systems. Given a query, a RAG system retrieves relevant information from external databases to supplement the model’s context. The generated response should be factually consistent with the retrieved context. RAG is a central topic in Chapter\6.

Safety
- unsafe content might belong to one of the following categories:
  - Inappropriate language, including profanity and explicit content.
  - Harmful recommendations and tutorials, such as “step-by-step guide to rob a bank” or encouraging users to engage in self-destructive behavior.
  - Hate speech, including racist, sexist, homophobic speech, and other discriminatory behaviors.
  - Violence, including threats and graphic detail.
  - Stereotypes, such as always using female names for nurses or male names for CEOs.
  - Biases toward a political or religious ideology, 

Instruction-Following Capability

- Instruction-following measurement asks the question: how good is this model at following the instructions you give it? If the model is bad at following instructions, it doesn’t matter how good your instructions are, the outputs will be bad. 
- Being able to follow instructions is a core requirement for foundation models, and most foundation models are trained to do so
- Instruction-following capability is essential for applications that require structured outputs, such as in JSON format or matching a regular expression (regex)
- Instruction-following capability isn’t straightforward to define or measure, as it can be easily conflated with domain-specific capability or generation capability.
- How well a model performs depends on the quality of its instructions, which makes it hard to evaluate AI models. When a model performs poorly, it can either be because the model is bad or the instruction is bad.

Instruction-following criteria
- The Google benchmark IFEval, Instruction-Following Evaluation, focuses on whether the model can produce outputs following an expected format
- GPT-4 isn’t as accurate as human experts, but it’s more accurate than annotators recruited through Amazon Mechanical Turk. They concluded that their benchmark can be automatically verified using AI judges.
- You should curate your own benchmark to evaluate your model’s capability to follow your instructions using your own criteria. If you need a model to output YAML, include YAML instructions in your benchmark. If you want a model to not say things like “As a language model”, evaluate the model on this instruction.

Roleplaying
- One of the most common types of real-world instructions is roleplaying—asking the model to assume a fictional character or a persona.

Cost and Latency

- When evaluating models, it’s important to balance model quality, latency, and cost. Many companies opt for lower-quality models if they provide better cost and latency. Cost and latency optimization are discussed in detail in Chapter9, so this section will be quick.
- When optimizing for multiple objectives, it’s important to be clear about what objectives you can and can’t compromise on
- There are multiple metrics for latency for foundation models, including but not limited to time to first token, time per token, time between tokens, time per query, etc. It’s important to understand what latency metrics matter to you.
- When evaluating models based on latency, it’s important to differentiate between the must-have and the nice-to-have. If you ask users if they want lower latency, nobody will ever say no. But high latency is often an annoyance, not a deal breaker.
- If you use model APIs, your cost per token usually doesn’t change much as you scale. However, if you host your own models, your cost per token can get much cheaper as you scale.
- Therefore, at different scales, companies need to reevaluate whether it makes more sense to use model APIs or to host their own models.

### Model Selection

- At the end of the day, you don’t really care about which model is the best. You care about which model is the best for your applications
- During the application development process, as you progress through different adaptation techniques, you’ll have to do model selection over and over again. For example, prompt engineering might start with the strongest model overall to evaluate feasibility and then work backward to see if smaller models would work. If you decide to do finetuning, you might start with a small model to test your code and move toward the biggest model that fits your hardware constraints (e.g., one GPU).
- In general, the selection process for each technique typically involves two steps:
  - Figuring out the best achievable performance 
  - Mapping models along the cost–performance axes and choosing the model that gives the best performance for your bucks

Model Selection Workflow

- When looking at models, it’s important to differentiate between hard attributes (what is impossible or impractical for you to change) and soft attributes (what you can and are willing to change).
- When estimating how much you can improve on a certain attribute, it can be tricky to balance being optimistic and being realistic. I’ve had situations where a model’s accuracy hovered around 20% for the first few prompts. However, the accuracy jumped to 70% after I decomposed the task into two steps. At the same time, I’ve had situations where a model remained unusable for my task even after weeks of tweaking, and I had to give up on that model.
- What you define as hard and soft attributes depends on both the model and your use case
- At a high level, the evaluation workflow consists of four steps 
  - Filter out models whose hard attributes don’t work for you. 
  - Use publicly available information, e.g., benchmark performance and leaderboard ranking, to narrow down the most promising models to experiment with, balancing different objectives such as model quality, latency, and cost.
  - Run experiments with your own evaluation pipeline to find the best model, again, balancing all your objectives.
  - Continually monitor your model in production to detect failure and collect feedback to improve your application.
- These four steps are iterative—you might want to change the decision from a previous step with newer information from the current step
- Chapter10 discusses monitoring and collecting user feedback. The rest of this chapter will discuss the first three steps. 

Model Build Versus Buy

- To signal whether the data is also open, the term “open weight” is used for models that don’t come with open data, whereas the term “open model” is used for models that come with open data.
- Some people argue that the term open source should be reserved only for fully open models. In this book, for simplicity, I use open source to refer to all models whose weights are made public, regardless of their training data’s availability and licenses.
- As of this writing, the vast majority of open source models are open weight only. Model developers might hide training data information on purpose, as this information can open model developers to public scrutiny and potential lawsuits.
- For a model to be accessible to users, a machine needs to host and run it. The service that hosts the model and receives user queries, runs the model to generate responses for queries, and returns these responses to the users is called an inference service.
- The term model API is typically used to refer to the API of the inference service, but there are also APIs for other model services, such as finetuning APIs and evaluation APIs. Chapter9 discusses how to optimize inference services.
- Model APIs can be available through model providers (such as OpenAI and Anthropic), cloud service providers (such as Azure and GCP [Google Cloud Platform]), or third-party API providers (such as Databricks Mosaic, Anyscale, etc.). 
- For commercial model providers, models are their competitive advantages. For API providers that don’t have their own models, APIs are their competitive advantages. This means API providers might be more motivated to provide better APIs with better pricing.
- Since building scalable inference services for larger models is nontrivial, many companies don’t want to build them themselves. This has led to the creation of many third-party inference and finetuning services on top of open source models. Major cloud providers like AWS, Azure, and GCP all provide API access to popular open source models. A plethora of startups are doing the same.
- The answer to whether to host a model yourself or use a model API depends on the use case. And the same use case can change over time. Here are seven axes to consider: data privacy, data lineage, performance, functionality, costs, control, and on-device deployment.
  - Externally hosted model APIs are out of the question for companies with strict data privacy policies that can’t send data outside of the organization
  - What’s the problem with people using your data to train their models? While research in this area is still sparse, some studies suggest that AI models can memorize their training samples. For example, it’s been found that Hugging Face’s StarCoder model memorizes 8% of its training set. These memorized samples can be accidentally leaked to users or intentionally exploited by bad actors, as demonstrated in Chapter5.
  - For most models, there’s little transparency about what data a model is trained on.
  - Concerns over data lineage have driven some companies toward fully open models, whose training data has been made publicly available. The argument is that this allows the community to inspect the data and make sure that it’s safe to use. While it sounds great in theory, in practice, it’s challenging for any company to thoroughly inspect a dataset of the size typically used to train foundation models.
  - Given the same concern, many companies opt for commercial models instead. Open source models tend to have limited legal resources compared to commercial models. If you use an open source model that infringes on copyrights, the infringed party is unlikely to go after the model developers, and more likely to go after you. However, if you use a commercial model, the contracts you sign with the model providers can potentially protect you from data lineage risks
  - Various benchmarks have shown that the gap between open source models and proprietary models is closing. Figure4-7 shows this gap decreasing on the MMLU benchmark over time. This trend has made many people believe that one day, there will be an open source model that performs just as well, if not better, than the strongest proprietary model.
  - For this reason, it’s likely that the strongest open source model will lag behind the strongest proprietary models for the foreseeable future. However, for many use cases that don’t need the strongest models, open source models might be sufficient.
  - Many functionalities are needed around a model to make it work for a use case. Here are some examples of these functionalities:
    - Scalability: making sure the inference service can support your application’s traffic while maintaining the desirable latency and cost. 
    - Function calling: giving the model the ability to use external tools, which is essential for RAG and agentic use cases, as discussed in Chapter6. 
    - Structured outputs, such as asking models to generate outputs in JSON format. 
    - Output guardrails: mitigating risks in the generated responses, such as making sure the responses aren’t racist or sexist.
  - The downside of using a model API is that you’re restricted to the functionalities that the API provides. A functionality that many use cases need is logprobs, which are very useful for classification tasks, evaluation, and interpretability. However, commercial model providers might be hesitant to expose logprobs for fear of others using logprobs to replicate their models. In fact, many model APIs don’t expose logprobs or expose only limited logprobs.
  - You can also only finetune a commercial model if the model provider lets you. Imagine that you’ve maxed out a model’s performance with prompting and want to finetune that model. If this model is proprietary and the model provider doesn’t have a finetuning API, you won’t be able to do it.
  - Model APIs charge per usage, which means that they can get prohibitively expensive with heavy usage. At a certain scale, a company that is bleeding its resources using APIs might consider hosting their own models
  - However, hosting a model yourself requires nontrivial time, talent, and engineering effort. You’ll need to optimize the model, scale and maintain the inference service as needed, and provide guardrails around your model. APIs are expensive, but engineering can be even more so.
  - In general, you want a model that is easy to use and manipulate. Typically, proprietary models are easier to get started with and scale, but open models might be easier to manipulate as their components are more accessible.
  - Regardless of whether you go with open or proprietary models, you want this model to follow a standard API, which makes it easier to swap models. Many model developers try to make their models mimic the API of the most popular models. As of this writing, many API providers mimic OpenAI’s API.
  - You might also prefer models with good community support. The more capabilities a model has, the more quirks it has. A model with a large community of users means that any issue you encounter may already have been experienced by others, who might have shared solutions online
  - If you want to run a model on-device, third-party APIs are out of the question. In many use cases, running a model locally is desirable. It could be because your use case targets an area without reliable internet access. It could be for privacy reasons, such as when you want to give an AI assistant access to all your data, but don’t want your data to leave your device

Navigate Public Benchmarks

- 