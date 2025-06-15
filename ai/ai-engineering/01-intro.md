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
