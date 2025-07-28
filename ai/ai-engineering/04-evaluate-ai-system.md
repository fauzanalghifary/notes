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

- 