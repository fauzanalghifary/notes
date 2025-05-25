# Book: AI Engineering by Chip Huyen

- https://learning.oreilly.com/library/view/ai-engineering/9781098166298

# Preface

- This book covers the end-to-end process of adapting foundation models to solve real-world problems, encompassing tried-and-true techniques from other engineering fields and techniques emerging with foundation models.

### What This Book Is About

- Some of the many questions that this book can help you answer are:
  - Should I build this AI application?
  - How do I evaluate my application? Can I use AI to evaluate AI outputs?
  - What causes hallucinations? How do I detect and mitigate hallucinations?
  - What are the best practices for prompt engineering?
  - Why does RAG work? What are the strategies for doing RAG? 
  - What’s an agent? How do I build and evaluate an agent?
  - When to finetune a model? When not to finetune a model?
  - How much data do I need? How do I validate the quality of my data? 
  - How do I make my model faster, cheaper, and secure? 
  - How do I create a feedback loop to improve my application continually?
- This book focuses on the fundamentals of AI engineering instead of any specific tool or API. Tools become outdated quickly, but fundamentals should last longer.
- AIE focuses on building applications on top of foundation models, which involves more prompt engineering, context construction, and parameter-efficient finetuning.

### What This Book Is Not

- This book isn’t a tutorial.
- it offers a framework for selecting tools
- This book isn’t an ML theory book
- the book is a practical book that focuses on helping you build successful AI applications to solve real-world problems.
- While it’s possible to build foundation model-based applications without ML expertise, a basic understanding of ML and statistics can help you build better applications and save you from unnecessary suffering.
- You can read this book without any prior ML background. However, you will be more effective while building AI applications if you know the following concepts:
  - Probabilistic concepts such as sampling, determinism, and distribution.
  - ML concepts such as supervision, self-supervision, log-likelihood, gradient descent, backpropagation, loss function, and hyperparameter tuning.
  - Various neural network architectures, including feedforward, recurrent, and transformer.
  - Metrics such as accuracy, F1, precision, recall, cosine similarity, and cross entropy.

### Who This Book Is For

- This book is for you if you can relate to one of the following scenarios:
  - You’re building or optimizing an AI application, whether you’re starting from scratch or looking to move beyond the demo phase into a production-ready stage. You may also be facing issues like hallucinations, security, latency, or costs, and need targeted solutions.
  - You want to streamline your team’s AI development process, making it more systematic, faster, and reliable
  - You want to understand how your organization can leverage foundation models to improve the business’s bottom line and how to build a team to do so.

### Navigating This Book

- Before deciding to build an AI application, it’s necessary to understand what this process involves and answer questions such as: Is this application necessary? Is AI needed?
- It also covers a range of successful use cases to give a sense of what foundation models can do.
- While an ML background is not necessary to build AI applications, understanding how a foundation model works under the hood is useful to make the most out of it.
- Once you’ve committed to building an application with foundation models, evaluation will be an integral part of every step along the way
- Evaluation is one of the hardest, if not the hardest, challenges of AI engineering.
- Given a query, the quality of a model’s response depends on the following aspects (outside of the model’s generation setting):
  - The instructions for how the model should behave
  - The context the model can use to respond to the query
  - The model itself
- Chapter 6 explores why context is important for a model to generate accurate responses. It zooms into two major application patterns for context construction: RAG and agentic. 
- RAG pattern is better understood and has proven to work well in production. On the other hand, while the agentic pattern promises to be much more powerful, it’s also more complex and is still being explored.
- Chapter 7 is about how to adapt a model to an application by changing the model itself with finetuning
- Due to the scale of foundation models, native model finetuning is memory-intensive, and many techniques are developed to allow finetuning better models with less memory. The chapter covers different finetuning approaches, supplemented by a more experimental approach: model merging. This chapter contains a more technical section that shows how to calculate the memory footprint of a model.
- Due to the availability of many finetuning frameworks, the finetuning process itself is often straightforward. However, getting data for finetuning is hard. The next chapter is all about data, including data acquisition, data annotations, data synthesis, and data processing.
- If Chapters 5 to 8 are about improving a model’s quality, Chapter 9 is about making its inference cheaper and faster.
- The last chapter in the book brings together the different concepts from this book to build an application end-to-end. The second part of the chapter is more product-focused, with discussions on how to design a user feedback system that helps you collect useful feedback while maintaining a good user experience.