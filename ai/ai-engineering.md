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

# Chapter 1 - Introduction to Building AI Applications with Foundation Models

- AI engineering—the process of building applications on top of readily available models
- foundation models, the key catalyst behind the explosion of AI engineering

### The Rise of AI Engineering

From Language Models to Large Language Models

- A language model encodes statistical information about one or more languages.
- The basic unit of a language model is token. A token can be a character, a word, or a part of a word (like -tion), depending on the model
- The process of breaking the original text into tokens is called tokenization.
- There are two main types of language models: masked language models and autoregressive language models.
  - A masked language model is trained to predict missing tokens anywhere in a sequence, using the context from both before and after the missing tokens.
  - masked language models are commonly used for non-generative tasks such as sentiment analysis and text classification.
  - An autoregressive language model is trained to predict the next token in a sequence, using only the preceding tokens
- A model that can generate open-ended outputs is called generative, hence the term generative AI.
- It’s important to note that completions are predictions, based on probabilities, and not guaranteed to be correct.
- As simple as it sounds, completion is incredibly powerful. Many tasks, including translation, summarization, coding, and solving math problems, can be framed as completion tasks.
- Language modeling is just one of many ML algorithms.
- What’s special about language models?
  - The answer is that language models can be trained using self-supervision, while many other models require supervision.
- A drawback of supervision is that data labeling is expensive and time-consuming.
- Self-supervision helps overcome the data labeling bottleneck
- In self-supervision, instead of requiring explicit labels, the model can infer labels from the input data. Language modeling is self-supervised because each input sequence provides both the labels (tokens to be predicted) and the contexts the model can use to predict these labels.
- Self-supervision differs from unsupervision. In self-supervised learning, labels are inferred from the input data. In unsupervised learning, you don’t need labels at all.
- Self-supervised learning means that language models can learn from text sequences without requiring any labeling. Because text sequences are everywhere—in books, blog posts, articles, and Reddit comments—it’s possible to construct a massive amount of training data, allowing language models to scale up to become LLMs.
- A model’s size is typically measured by its number of parameters. 
- A parameter is a variable within an ML model that is updated through the training process
- In general, though this is not always true, the more parameters a model has, the greater its capacity to learn desired behaviors.
- Why do larger models need more data? 
  - Larger models have more capacity to learn, and, therefore, would need more training data to maximize their performance. 
  - You can train a large model on a small dataset too, but it’d be a waste of compute. You could have achieved similar or better results on this dataset with smaller models.

From Large Language Models to Foundation Models

- While language models are capable of incredible tasks, they are limited to text
- Being able to process data beyond text is essential for AI to operate in the real world.
- “incorporating additional modalities (such as image inputs) into LLMs is viewed by some as a key frontier in AI research and development.”
- While many people still call Gemini and GPT-4V LLMs, they’re better characterized as foundation models.
- Foundation models mark a breakthrough from the traditional structure of AI research. For a long time, AI research was divided by data modalities
- A model that can work with more than one data modality is also called a multimodal model
- Self-supervision works for multimodal models too.
- Foundation models also mark the transition from task-specific models to general-purpose models.
- Foundation models, thanks to their scale and the way they are trained, are capable of a wide range of tasks
- However, you can often tweak a general-purpose model to maximize its performance on a specific task.
- There are multiple techniques you can use to get the model to generate what you want. 
  - prompt engineering
  - retrieval-augmented generation (RAG)
  - finetune
- Prompt engineering, RAG, and finetuning are three very common AI engineering techniques that you can use to adapt a model to your needs
- Adapting an existing powerful model to your task is generally a lot easier than building a model for your task from scratch

From Foundation Models to AI Engineering

- AI engineering refers to the process of building applications on top of foundation models
- People have been building AI applications for over a decade—a process often known as ML engineering or MLOps (short for ML operations). Why do we talk about AI engineering now?
- If traditional ML engineering involves developing ML models, AI engineering leverages existing ones.
- Applications previously thought impossible are now possible, and applications not thought of before are emerging.
- This makes AI more useful for more aspects of life, vastly increasing both the user base and the demand for AI applications.
- estimated AI cost for his use cases has gone down two orders of magnitude from April 2022 to April 2023.
- the biggest opportunity for the vast majority of people will be to adapt these models for specific applications.
- AI engineering has rapidly emerged as one of the fastest, and quite possibly the fastest-growing, engineering discipline.
- Many terms are being used to describe the process of building applications on top of foundation models, including ML engineering, MLOps, AIOps, LLMOps, etc. Why did I choose to go with AI engineering for this book?

### Foundation Model Use Cases

- customer experience, employee productivity, and process optimization.
- cost reduction, process efficiency, growth, and accelerating innovation

### Planning AI Applications

- It’s easy to build a cool demo with foundation models. It’s hard to create a profitable product.
- Here are a few examples of different levels of risks, ordered from high to low:
  - If you don’t do this, competitors with AI can make you obsolete.
  - If you don’t do this, you’ll miss opportunities to boost profits and productivity.
  - You’re unsure where AI will fit into your business yet, but you don’t want to be left behind
- The more critical AI is to the application, the more accurate and reliable the AI part has to be. People are more accepting of mistakes when AI isn’t core to the application.
- A reactive feature shows its responses in reaction to users’ requests or specific actions, whereas a proactive feature shows its responses when there’s an opportunity for it
  - Because users don’t ask for proactive features, they can view them as intrusive or annoying if the quality is low. Therefore, proactive predictions and generations typically have a higher quality bar.
- Dynamic features are updated continually with user feedback, whereas static features are updated periodically.