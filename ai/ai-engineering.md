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

Use Case Evaluation

- It’s easy to build a cool demo with foundation models. It’s hard to create a profitable product.
- Here are a few examples of different levels of risks, ordered from high to low:
  - If you don’t do this, competitors with AI can make you obsolete.
  - If you don’t do this, you’ll miss opportunities to boost profits and productivity.
  - You’re unsure where AI will fit into your business yet, but you don’t want to be left behind
- The more critical AI is to the application, the more accurate and reliable the AI part has to be. People are more accepting of mistakes when AI isn’t core to the application.
- A reactive feature shows its responses in reaction to users’ requests or specific actions, whereas a proactive feature shows its responses when there’s an opportunity for it
  - Because users don’t ask for proactive features, they can view them as intrusive or annoying if the quality is low. Therefore, proactive predictions and generations typically have a higher quality bar.
- Dynamic features are updated continually with user feedback, whereas static features are updated periodically.
- In the case of AI, dynamic features might mean that each user has their own model, continually finetuned on their data, or other mechanisms for personalization such as ChatGPT’s memory feature, which allows ChatGPT to remember each user’s preferences
- human-in-the-loop.
- If something is easy for you to build, it’s also easy for your competitors. What moats do you have to defend your product?
- if the underlying models expand in capabilities, the layer you provide might be subsumed by the models, rendering your application obsolete
- In AI, there are generally three types of competitive advantages: technology, data, and distribution—the ability to bring your product in front of users. With foundation models, the core technologies of most companies will be similar. The distribution advantage likely belongs to big companies. The data advantage is more nuanced
- Even for the scenarios where user data can’t be used to train models directly, usage information can give invaluable insights into user behaviors and product shortcomings, which can be used to guide the data collection and training process

Setting Expectations

- the next step is to figure out what success looks like: how will you measure success? The most important metric is how this will impact your business.
- usefulness threshold: how good it has to be for it to be useful.

Milestone Planning

- Evaluate existing models to understand their capabilities. The stronger the off-the-shelf models, the less work you’ll have to do
- Planning an AI product needs to account for its last mile challenge. Initial success with foundation models can be misleading.
- a good initial demo doesn’t promise a good end product. It might take a weekend to build a demo but months, and even years, to build a product.
- This initial success made them grossly underestimate how much time it’d take them to improve the product
- A lot of time was spent working on the product kinks and dealing with hallucinations.

Maintenance

- However, as each model has its quirks, strengths, and weaknesses, developers working with the new model will need to adjust their workflows, prompts, and data to this new model. Without proper infrastructure for versioning and evaluation in place, the process can cause a lot of headaches.

### AI Engineering Stack

- Instead of trying to keep up with the constantly shifting sand, let’s look into the fundamental building blocks of AI engineering.
- To understand AI engineering, it’s important to recognize that AI engineering evolved out of ML engineering
- Regardless of where organizations position AI engineers and ML engineers, their roles have significant overlap. Existing ML engineers can add AI engineering to their lists of skills to expand their job prospects. However, there are also AI engineers with no previous ML experience.

Three Layers of the AI Stack

- application development, model development, and infrastructure.
- Application development involves providing a model with good prompts and necessary context. This layer requires rigorous evaluation. Good applications also demand good interfaces.
- Model development layer provides tooling for developing models, including frameworks for modeling, training, finetuning, and inference optimization. Because data is central to model development, this layer also contains dataset engineering. Model development also requires rigorous evaluation.
- Infrastructure layer includes tooling for model serving, managing data and compute, and monitoring.
- With classical ML engineering, you experiment with different hyperparameters. With foundation models, you experiment with different models, prompts, retrieval algorithms, sampling variables, and more.
- It’s still important to set up a feedback loop so that we can iteratively improve our applications with production data.

AI Engineering vs ML Engineering

- Without foundation models, you have to train your own models for your applications. With AI engineering, you use a model someone else has trained for you. This means that AI engineering focuses less on modeling and training, and more on model adaptation.
- AI engineering works with models that are bigger, consume more compute resources, and incur higher latency than traditional ML engineering. This means that there’s more pressure for efficient training and inference optimization.
- AI engineering works with models that can produce open-ended outputs. Open-ended outputs give models the flexibility to be used for more tasks, but they are also harder to evaluate. This makes evaluation a much bigger problem in AI engineering.
- In short, AI engineering differs from ML engineering in that it’s less about model development and more about adapting and evaluating models
- model adaptation techniques
  - Prompt-based techniques, which include prompt engineering, adapt a model without updating the model weights. You adapt a model by giving it instructions and context instead of changing the model itself.
  - Prompt engineering is easier to get started and requires less data.
  - prompt engineering might not be enough for complex tasks or applications with strict performance requirements.
  - Finetuning, on the other hand, requires updating model weights. You adapt a model by making changes to the model itself. 
  - In general, finetuning techniques are more complicated and require more data, but they can improve your model’s quality, latency, and cost significantly.

Model Development

- Model development is the layer most commonly associated with traditional ML engineering. It has three main responsibilities: modeling and training, dataset engineering, and inference optimization.\
  - Modeling and training refers to the process of coming up with a model architecture, training it, and finetuning it.
  - Google’s TensorFlow, Hugging Face’s Transformers, and Meta’s PyTorch.
  - Developing ML models requires specialized ML knowledge. It requires knowing different types of ML algorithms (such as clustering, logistic regression, decision trees, and collaborative filtering) and neural network architectures (such as feedforward, recurrent, convolutional, and transformer). It also requires understanding how a model learns, including concepts such as gradient descent, loss function, regularization, etc.
  - With the availability of foundation models, ML knowledge is no longer a must-have for building AI applications. I’ve met many wonderful and successful AI application builders who aren’t at all interested in learning about gradient descent. However, ML knowledge is still extremely valuable, as it expands the set of tools that you can use and helps troubleshooting when a model doesn’t work as expected.

TRAINING, PRE-TRAINING, FINETUNING, AND POST-TRAINING

- Training always involves changing model weights, but not all changes to model weights constitute training
- Pre-training refers to training a model from scratch—the model weights are randomly initialized
  - Out of all training steps, pre-training is often the most resource-intensive by a long shot.
  - Pre-training also takes a long time to do. A small mistake during pre-training can incur a significant financial loss and set back the project significantly.
- Finetuning means continuing to train a previously trained model—the model weights are obtained from the previous training process. Because the model already has certain knowledge from pre-training, finetuning typically requires fewer resources (e.g., data and compute) than pre-training.
  - Conceptually, post-training and finetuning are the same and can be used interchangeably.
  - It’s usually post-training when it’s done by model developers.
  - It’s finetuning when it’s done by application developers.

Dataset Engineering

- Dataset engineering refers to curating, generating, and annotating the data needed for training and adapting AI models.
- In traditional ML engineering, most use cases are close-ended—a model’s output can only be among predefined values.
- Foundation models, however, are open-ended. Annotating open-ended queries is much harder than annotating close-ended queries—it’s easier to determine whether an email is spam than to write an essay. So data annotation is a much bigger challenge for AI engineering.
- Another difference is that traditional ML engineering works more with tabular data, whereas foundation models work with unstructured data. In AI engineering, data manipulation is more about deduplication, tokenization, context retrieval, and quality control, including removing sensitive information and toxic data.
- Many people argue that because models are now commodities, data will be the main differentiator, making dataset engineering more important than ever.
- Regardless of how much data you need, expertise in data is useful when examining a model, as its training data gives important clues about that model’s strengths and weaknesses.

Inference optimization

- Inference optimization means making models faster and cheaper.
- as foundation models scale up to incur even higher inference cost and latency, inference optimization has become even more important.
- One challenge with foundation models is that they are often autoregressive—tokens are generated sequentially.
  - As users are getting notoriously impatient, getting AI applications’ latency down to the 100 ms latency expected for a typical internet application is a huge challenge. Inference optimization has become an active subfield in both industry and academia.

Application development

- With foundation models, where many teams use the same model, differentiation must be gained through the application development process.
- The application development layer consists of these responsibilities: evaluation, prompt engineering, and AI interface.
- Evaluation is about mitigating risks and uncovering opportunities.
  - Evaluation is needed to select models, to benchmark progress, to determine whether an application is ready for deployment, and to detect issues and opportunities for improvement in production.
  - The existence of so many adaptation techniques also makes evaluation harder
- Prompt engineering is about getting AI models to express the desirable behaviors from the input alone, without changing the model weights.
  - By using a different prompt engineering technique, Gemini Ultra’s performance on MMLU went from 83.7% to 90.04%.
  - It’s possible to get a model to do amazing things with just prompts. The right instructions can get a model to perform the task you want, in the format of your choice
  - Prompt engineering is not just about telling a model what to do. It’s also about giving the model the necessary context and tools to do a given task. For complex tasks with long context, you might also need to provide the model with a memory management system so that the model can keep track of its history.
- AI interface means creating an interface for end users to interact with your AI applications

### AI Engineering Versus Full-Stack Engineering

- The rising importance of interfaces leads to a shift in the design of AI toolings to attract more frontend engineers
- Traditionally, ML engineering is Python-centric. Before foundation models, the most popular ML frameworks supported mostly Python APIs. Today, Python is still popular, but there is also increasing support for JavaScript APIs.
  - LangChain.js, Transformers.js, OpenAI’s Node library, and Vercel’s AI SDK.
- While many AI engineers come from traditional ML backgrounds, more are increasingly coming from web development or full-stack backgrounds. An advantage that full-stack engineers have over traditional ML engineers is their ability to quickly turn ideas into demos, get feedback, and iterate.
- With traditional ML engineering, you usually start with gathering data and training a model. Building the product comes last. However, with AI models readily available today, it’s possible to start with building the product first, and only invest in data and models once the product shows promise.
- In traditional ML engineering, model development and product development are often disjointed processes, with ML engineers rarely involved in product decisions at many organizations. However, with foundation models, AI engineers tend to be much more involved in building the product.

### Summary

- One is to explain the emergence of AI engineering as a discipline, thanks to the availability of foundation models.
- Two is to give an overview of the process needed to build applications on top of these models
- While AI engineering is a new term, it evolved out of ML engineering, which is the overarching discipline involved with building applications with all ML models. Many principles from ML engineering are still applicable to AI engineering. However, AI engineering also brings with it new challenges and solutions.


“AI engineering is just software engineering with AI models thrown in the stack.”


# Chapter 2 - Understanding Foundation Models

- In general, however, differences in foundation models can be traced back to decisions about training data, model architecture and size, and how they are post-trained to align with human preferences.
- Since models learn from data, their training data reveals a great deal about their capabilities and limitations
- Sampling is how a model chooses an output from all possible options. It is perhaps one of the most underrated concepts in AI. Not only does sampling explain many seemingly baffling AI behaviors, including hallucinations and inconsistencies, but choosing the right sampling strategy can also significantly boost a model’s performance with relatively little effort.
- Concepts covered in this chapter are fundamental for understanding the rest of the book.

### Training Data

- An AI model is only as good as the data it was trained on.
- If you want a model to improve on a certain task, you might want to include more data for that task in the training data.
- To address this issue, it’s crucial to curate datasets that align with your specific needs.
- While language- and domain-specific foundation models can be trained from scratch, it’s also common to finetune them on top of general-purpose models.
- a model trained with a smaller amount of high-quality data might outperform a model trained with a large amount of low-quality data

Domain-Specific Models

- Even though general-purpose foundation models can answer everyday questions about different domains, they are unlikely to perform well on domain-specific tasks, especially if they never saw these tasks during training.
- To train a model to perform well on these domain-specific tasks, you might need to curate very specific datasets.

- This section gave a high-level overview of how training data impacts a model’s performance. Next, let’s explore the impact of how a model is designed on its performance.

### Modeling

- Before training a model, developers need to decide what the model should look like. What architecture should it follow? How many parameters should it have?

Model Architecture

- As of this writing, the most dominant architecture for language-based foundation models is the transformer architecture
- attention mechanism
- seq2seq
  - In its most basic form, the encoder processes the input tokens sequentially, outputting the final hidden state that represents the input. The decoder then generates output tokens sequentially, conditioned on both the final hidden state of the input and the previously generated token
  - 2 problems:
    - First, the vanilla seq2seq decoder generates output tokens using only the final hidden state of the input. Intuitively, this is like generating answers about a book using the book summary. This limits the quality of the generated outputs.
    - Second, the RNN encoder and decoder mean that both input processing and output generation are done sequentially, making it slow for long sequences. If an input is 200 tokens long, seq2seq has to wait for each input token to finish processing before moving on to the next
- Transformer architecture
  - address seq2seq problems with the attention mechanism.
  - The attention mechanism allows the model to weigh the importance of different input tokens when generating each output token. This is like generating answers by referencing any page in the book.
  - With transformers, the input tokens can be processed in parallel, significantly speeding up input processing
  - While the transformer removes the sequential input bottleneck, transformer-based autoregressive language models still have the sequential output bottleneck.
- Inference for transformer-based language models, therefore, consists of two steps:
  - Prefill
    - The model processes the input tokens in parallel.
  - Decode
    - The model generates one output token at a time.
- As explored later in Chapter 9, the parallelizable nature of prefilling and the sequential aspect of decoding both motivate many optimization techniques to make language model inference cheaper and faster.
- Attention mechanism
  - At the heart of the transformer architecture is the attention mechanism
- Under the hood, the attention mechanism leverages key, value, and query vectors:
  - The query vector (Q): 
    - the current state of the decoder at each decoding step
    -  the person looking for information to create a summary.
  - Each key vector (K): 
    - previous token
    - If each previous token is a page in the book, each key vector is like the page number
    - at a given decoding step, previous tokens include both input tokens and previously generated tokens.
  - Each value vector (V):
    - the actual value of a previous token, as learned by the model.
    - Each value vector is like the page’s content.
- The attention mechanism computes how much attention to give an input token by performing a dot product between the query vector and its key vector. A high score means that the model will use more of that page’s content (its value vector) when generating the book’s summary. 
  - Because each previous token has a corresponding key and value vector, the longer the sequence, the more key and value vectors need to be computed and stored. This is one reason why it’s so hard to extend context length for transformer models.
- in general, each transformer block contains the attention module and the MLP (multi-layer perceptron) module:
  -Attention module: Each attention module consists of four weight matrices: query, key, value, and output projection.
  - MLP module
- Note that while the increased context length impacts the model’s memory footprint, it doesn’t impact the model’s total number of parameters.
- Modeling long sequences remains a core challenge in developing LLMs.

Model Size

- In general, increasing a model’s parameters increases its capacity to learn, resulting in better models.
- The number of parameters helps us estimate the compute resources needed to train and run this model. 
- A larger model can also underperform a smaller model if it’s not trained on enough data.
- When discussing model size, it’s important to consider the size of the data it was trained on. For most models, dataset sizes are measured by the number of training samples
- A better measurement is the number of tokens in the dataset.
- As of this writing, LLMs are trained using datasets in the order of trillions of tokens.
- The number of tokens in a model’s dataset isn’t the same as its number of training tokens. The number of training tokens measures the tokens that the model is trained on. If a dataset contains 1 trillion tokens and a model is trained on that dataset for two epochs—an epoch is a pass through the dataset—the number of training tokens is 2 trillion.
- While this section focuses on the scale of data, quantity isn’t the only thing that matters. Data quality and data diversity matter, too. Quantity, quality, and diversity are the three golden goals for training data
- What’s considered good utilization depends on the model, the workload, and the hardware. Generally, if you can get half the advertised performance, 50% utilization, you’re doing okay. Anything above 70% utilization is considered great.
- In summary, three numbers signal a model’s scale:
  - Number of parameters, which is a proxy for the model’s learning capacity.
  - Number of tokens a model was trained on, which is a proxy for how much a model learned.
  - Number of FLOPs, which is a proxy for the training cost.
- They found that larger language models are sometimes (only sometimes) worse on tasks that require memorization and tasks with strong priors.
- ...
  - Model performance depends on the model size and the dataset size.
  - Bigger models and bigger datasets require more compute.
  - Compute costs money.
- Given a compute budget, the rule that helps calculate the optimal model size and dataset size is called the Chinchilla scaling law
- for compute-optimal training, you need the number of training tokens to be approximately 20 times the model size.
- The model size and the number of training tokens should be scaled equally: for every doubling of the model size, the number of training tokens should also be doubled.
- Smaller models are easier to work with and cheaper to run inference on, which helped their models gain wider adoption
- The performance of a model depends heavily on the values of its hyperparameters
- A parameter can be learned by the model during the training process. A hyperparameter is set by users to configure the model and control how the model learns. Hyperparameters to configure the model include the number of layers, the model dimension, and vocabulary size. Hyperparameters to control how a model learns include batch size, number of epochs, learning rate, per-layer initial variance, and more.
- While it’s hard to answer these questions, there are already two visible bottlenecks for scaling: training data and electricity.
- Once the publicly available data is exhausted, the most feasible paths for more human-generated training data is proprietary data.
- the next critical set of design choices: how to align models with human preferences.

### Post-Training

- in general, post-training consists of two steps:
  - Supervised finetuning (SFT): Finetune the pre-trained model on high-quality instruction data to optimize models for conversations instead of completion.
  - Preference finetuning: Further finetune the model to output responses that align with human preference. Preference finetuning is typically done with reinforcement learning (RL)
- For language-based foundation models, pre-training optimizes token-level quality, where the model is trained to predict the next token accurately. However, users don’t care about token-level quality—they care about the quality of the entire response. Post-training, in general, optimizes the model to generate responses that users prefer. Some people compare pre-training to reading to acquire knowledge, while post-training is like learning how to use that knowledge.
- As post-training consumes a small portion of resources compared to pre-training (InstructGPT used only 2% of compute for post-training and 98% for pre-training), you can think of post-training as unlocking the capabilities that the pre-trained model already has but are hard for users to access via prompting alone.
- Note that a combination of pre-training, SFT, and preference finetuning is the popular solution for building foundation models today, but it’s not the only solution

Supervised Finetuning

- pre-trained model is likely optimized for completion rather than conversing
- demonstration data.
- behavior cloning: you demonstrate how the model should behave, and the model clones this behavior.
- good labelers are important for AIs to learn how to conduct intelligent conversations
- Companies, therefore, often use highly educated labelers to generate demonstration data
- Technically, you can train a model from scratch on the demonstration data instead of finetuning a pre-trained model, effectively eliminating the self-supervised pre-training step. However, the pre-training approach often has returned superior results.

Preference Finetuning

- Demonstration data teaches the model to have a conversation but doesn’t teach the model what kind of conversations it should have. For example, if a user asks the model to write an essay about why one race is inferior or how to hijack a plane, should the model comply?
- The goal of preference finetuning is to get AI models to behave according to human preference.
- given the ambitious nature of the goal, the solution we have today is complicated. The earliest successful preference finetuning algorithm, which is still popular today, is RLHF
- RLHF:
  - Train a reward model that scores the foundation model’s outputs.
  - Optimize the foundation model to generate responses for which the reward model will give maximal scores.
- While RLHF is still used today, newer approaches like DPO
- RLHF, while more complex than DPO, provides more flexibility to tweak the model
- RLHF relies on a reward model. Given a pair of (prompt, response), the reward model outputs a score for how good the response is. Training a model to score a given input is a common ML task. The challenge, similar to that of SFT, is to obtain reliable data.
- Evaluating each sample independently is also called pointwise evaluation.
- An easier task is to ask labelers to compare two responses and decide which one is better.
- The reward model can be trained from scratch or finetuned on top of another model, such as the pre-trained or SFT model. Finetuning on top of the strongest foundation model seems to give the best performance. Some people believe that the reward model should be at least as powerful as the foundation model to be able to score the foundation model’s responses. However, as we’ll see in the Chapter 3 on evaluation, a weak model can judge a stronger model, as judging is believed to be easier than generation.
- Empirically, RLHF and DPO both improve performance compared to SFT alone. However, as of this writing, there are debates on why they work. As the field evolves, I suspect that preference finetuning will change significantly in the future
- Both SFT and preference finetuning are steps taken to address the problem created by the low quality of data used for pre-training. If one day we have better pre-training data or better ways to train foundation models, we might not need SFT and preference at all.
- Some companies find it okay to skip reinforcement learning altogether. For example, Stitch Fix and Grab find that having the reward model alone is good enough for their applications. They get their models to generate multiple outputs and pick the ones given high scores by their reward models. This approach, often referred to as the best of N strategy, leverages how a model samples outputs to improve its performance

### Sampling

- A model constructs its outputs through a process known as sampling.
- Sampling makes AI’s outputs probabilistic. Understanding this probabilistic nature is important for handling AI’s behaviors, such as inconsistency and hallucination. This section ends with a deep dive into what this probabilistic nature means and how to work with it.

Sampling Fundamentals

- Given an input, a neural network produces an output by first computing the probabilities of possible outcomes
- For a language model, to generate the next token, the model first computes the probability distribution over all tokens in the vocabulary
- When working with possible outcomes of different probabilities, a common strategy is to pick the outcome with the highest probability. Always picking the most likely outcome = is called greedy sampling. This often works for classification tasks. For example, if the model thinks that an email is more likely to be spam than not spam, it makes sense to mark it as spam. However, for a language model, greedy sampling creates boring outputs. Imagine a model that, for whatever question you ask, always responds with the most common words.
- Instead of always picking the next most likely token, the model can sample the next token according to the probability distribution over all possible values. Given the context of “My favorite color is …” as shown in Figure 2-14, if “red” has a 30% chance of being the next token and “green” has a 50% chance, “red” will be picked 30% of the time, and “green” 50% of the time.

Sampling Strategies

- The right sampling strategy can make a model generate responses more suitable for your application
- For example, one sampling strategy can make the model generate more creative responses, whereas another strategy can make its generations more predictable
- Temperature
  - One problem with sampling the next token according to the probability distribution is that the model can be less creative.
  - To redistribute the probabilities of the possible values, you can sample with a temperature. Intuitively, a higher temperature reduces the probabilities of common tokens, and as a result, increases the probabilities of rarer tokens. This enables models to create more creative responses.
  - The higher the temperature, the less likely it is that the model is going to pick the most obvious value (the value with the highest logit), making the model’s outputs more creative but potentially less coherent. The lower the temperature, the more likely it is that the model is going to pick the most obvious value, making the model’s output more consistent but potentially more boring.
  - A temperature of 0.7 is often recommended for creative use cases, as it balances creativity and predictability, but you should experiment and find the temperature that works best for you.
  - It’s common practice to set the temperature to 0 for the model’s outputs to be more consistent. Technically, temperature can never be 0—logits can’t be divided by 0. In practice, when we set the temperature to 0, the model just picks the token with the largest logit,25 without doing logit adjustment and softmax calculation.
  - A common debugging technique when working with an AI model is to look at the probabilities this model computes for given inputs. For example, if the probabilities look random, the model hasn’t learned much.
  - Logprobs, short for log probabilities, are probabilities in the log scale. Log scale is preferred when working with a neural network’s probabilities because it helps reduce the underflow problem
  - logprobs are useful for building applications (especially for classification), evaluating applications, and understanding how models work under the hood. However, as of this writing, many model providers don’t expose their models’ logprobs, or if they do, the logprobs API is limited.27 The limited logprobs API is likely due to security reasons as a model’s exposed logprobs make it easier for others to replicate the model.
- Top-k
  - Top-k is a sampling strategy to reduce the computation workload without sacrificing too much of the model’s response diversity.
  - A smaller k value makes the text more predictable but less interesting, as the model is limited to a smaller set of likely words.
- Top-p
  - In top-k sampling, the number of values considered is fixed to k. However, this number should change depending on the situation.
  - Top-p, also known as nucleus sampling, allows for a more dynamic selection of values to be sampled from. In top-p sampling, the model sums the probabilities of the most likely next values in descending order and stops when the sum reaches p. Only the values within this cumulative probability are considered.
  - Common values for top-p (nucleus) sampling in language models typically range from 0.9 to 0.95. A top-p value of 0.9, for example, means that the model will consider the smallest set of values whose cumulative probability exceeds 90%.
  - In theory, there don’t seem to be a lot of benefits to top-p sampling. However, in practice, top-p sampling has proven to work well, causing its popularity to rise.
- Stopping condition

Test Time Compute