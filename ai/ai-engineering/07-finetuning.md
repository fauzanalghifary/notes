# Chapter 7 - Finetuning

- Finetuning is the process of adapting a model to a specific task by further training the whole model or part of the model. Chapters 5 and 6 discuss prompt-based methods, which adapt a model by giving it instructions, context, and tools. Finetuning adapts a model by adjusting its weights.
- Finetuning can enhance various aspects of a model. It can improve the model’s domain-specific capabilities, such as coding or medical question answering, and can also strengthen its safety. However, it is most often used to improve the model’s instruction-following ability, particularly to ensure it adheres to specific output styles and formats.
- While finetuning can help create models that are more customized to your needs, it also requires more up-front investment. A question I hear very often is when to finetune and when to do RAG. After an overview of finetuning, this chapter will discuss the reasons for finetuning and the reasons for not finetuning, as well as a simple framework for thinking about choosing between finetuning and alternate methods.
- Compared to prompt-based methods, finetuning incurs a much higher memory footprint. At the scale of today’s foundation models, naive finetuning often requires more memory than what’s available on a single GPU. This makes finetuning expensive and challenging to do.
- A memory-efficient approach that has become dominant in the finetuning space is PEFT (parameter-efficient finetuning). 
- With prompt-based methods, knowledge about how ML models operate under the hood is recommended but not strictly necessary. However, finetuning brings you to the realm of model training, where ML knowledge is required.
- This chapter is the most technically challenging one for me to write, not because of the complexity of the concepts, but because of the broad scope these concepts cover. I suspect it might also be technically challenging to read

### Finetuning Overview

- The goal of finetuning is to get this model to perform well enough for your specific task.
- Finetuning is one way to do transfer learning, a concept first introduced by Bozinovski and Fulgosi in 1976. Transfer learning focuses on how to transfer the knowledge gained from one task to accelerate learning for a new, related task. This is conceptually similar to how humans transfer skills: for example, knowing how to play the piano can make it easier to learn another musical instrument.
- Since the early days of deep learning, transfer learning has offered a solution for tasks with limited or expensive training data. By training a base model on tasks with abundant data, you can then transfer that knowledge to a target task.
- For LLMs, knowledge gained from pre-training on text completion (a task with abundant data) is transferred to more specialized tasks, like legal question answering or text-to-SQL, which often have less available data. This capability for transfer learning makes foundation models particularly valuable.
- A sample-efficient model learns effectively from fewer samples. For example, while training a model from scratch for legal question answering may need millions of examples, finetuning a good base model might only require a few hundred.
- Ideally, much of what the model needs to learn is already present in the base model, and finetuning just refines the model’s behavior. OpenAI’s InstructGPT paper (2022) suggested viewing finetuning as unlocking the capabilities a model already has but that are difficult for users to access via prompting alone.
- Finetuning is part of a model’s training process. It’s an extension of model pre-training. Because any training that happens after pre-training is finetuning, finetuning can take many different forms. Chapter 2 already discussed two types of finetuning: supervised finetuning and preference finetuning
- Before finetuning this pre-trained model with expensive task-specific data, you can finetune it with self-supervision using cheap task-related data
- The massive amount of data a model can learn from during self-supervised learning outfits the model with a rich understanding of the world, but it might be hard for users to extract that knowledge for their tasks, or the way the model behaves might be misaligned with human preference. Supervised finetuning uses high-quality annotated data to refine the model to align with human usage and preference.
- A model can also be finetuned with reinforcement learning to generate responses that maximize human preference. Preference finetuning requires comparative data that typically follows the format (instruction, winning response, losing response).
- It’s possible to finetune a model to extend its context length. Long-context finetuning typically requires modifying the model’s architecture, such as adjusting the positional embeddings. A long sequence means more possible positions for tokens, and positional embeddings should be able to handle them. Compared to other finetuning techniques, long-context finetuning is harder to do. The resulting model might also degrade on shorter sequences.
- As an application developer, you might finetune a pre-trained model, but most likely, you’ll finetune a model that has been post-trained. The more refined a model is and the more relevant its knowledge is to your task, the less work you’ll have to do to adapt it.


### When to Finetune

- Compared to prompt-based methods, finetuning requires significantly more resources, not just in data and hardware, but also in ML talent. Therefore, finetuning is generally attempted after extensive experiments with prompt-based methods. However, finetuning and prompting aren’t mutually exclusive. Real-world problems often require both approaches.

Reasons to Finetune

- The primary reason for finetuning is to improve a model’s quality, in terms of both general capabilities and task-specific capabilities. Finetuning is commonly used to improve a model’s ability to generate outputs following specific structures, such as JSON or YAML formats.
- A general-purpose model that performs well on a wide range of benchmarks might not perform well on your specific task. If the model you want to use wasn’t sufficiently trained on your task, finetuning it with your data can be especially useful.
- One especially interesting use case of finetuning is bias mitigation. The idea is that if the base model perpetuates certain biases from its training data, exposing it to carefully curated data during finetuning can counteract these biases (Wang and Russakovsky, 2023)
- You can finetune a big model to make it even better, but finetuning smaller models is much more common. Smaller models require less memory, and, therefore, are easier to finetune. They are also cheaper and faster to use in production.
- A common approach is to finetune a small model to imitate the behavior of a larger model using data generated by this large model. Because this approach distills the larger model’s knowledge into the smaller model, it’s called distillation
- A small model, finetuned on a specific task, might outperform a much larger out-of-the-box model on that task. For example, Grammarly found that their finetuned Flan-T5 models (Chung et al., 2022) outperformed a GPT-3 variant specialized in text editing across a wide range of writing assistant tasks despite being 60 times smaller
- In the early days of foundation models, when the strongest models were commercial with limited finetuning access, there weren’t many competitive models available for finetuning. However, as the open source community proliferates with high-quality models of all sizes, tailored for a wide variety of domains, finetuning has become a lot more viable and attractive.

Reasons Not to Finetune

- Finetuning can improve a model’s performance, but so do carefully crafted prompts and context. Finetuning can help with structured outputs, but many other techniques, as discussed in Chapter 2, can also do that.
- First, while finetuning a model for a specific task can improve its performance for that task, it can degrade its performance for other tasks.1 This can be frustrating when you intend this model for an application that expects diverse prompts.
- What do you do in this situation? You can finetune the model on all the queries you care about
- If you’re just starting to experiment with a project, finetuning is rarely the first thing you should attempt. Finetuning requires high up-front investments and continual maintenance. First, you need data. Annotated data can be slow and expensive to acquire manually, especially for tasks that demand critical thinking and domain expertise. Open source data and AI-generated data can mitigate the cost, but their effectiveness is highly variable.
- Second, finetuning requires the knowledge of how to train models. You need to evaluate base models to choose one to finetune. Depending on your needs and resources, options might be limited. While finetuning frameworks and APIs can automate many steps in the actual finetuning process, you still need to understand the different training knobs you can tweak, monitor the learning process, and debug when something is wrong. 
- For example, you need to understand how an optimizer works, what learning rate to use, how much training data is needed, how to address overfitting/underfitting, and how to evaluate your models throughout the process.
- Third, once you have a finetuned model, you’ll need to figure out how to serve it. Will you host it yourself or use an API service? As discussed in Chapter9, inference optimization for large models, especially LLMs, isn’t trivial. Finetuning requires less of a technical leap if you’re already hosting your models in-house and familiar with how to operate models.
- More importantly, you need to establish a policy and budget for monitoring, maintaining, and updating your model. As you iterate on your finetuned model, new base models are being developed at a rapid pace. These base models may improve faster than you can enhance your finetuned model.
- AI engineering experiments should start with prompting, following the best practices discussed in Chapter 6. Explore more advanced solutions only if prompting alone proves inadequate. Ensure you have thoroughly tested various prompts, as a model’s performance can vary greatly with different prompts.
- Many practitioners I’ve spoken with share a similar story that goes like this. Someone complains that prompting is ineffective and insists on finetuning. Upon investigation, it turns out that prompt experiments were minimal and unsystematic. Instructions were unclear, examples didn’t represent actual data, and metrics were poorly defined. After refining the prompt experiment process, the prompt quality improved enough to be sufficient for their application
- Beware of the argument that general-purpose models don’t work well for domain-specific tasks, and, therefore, you must finetune or train models for your specific tasks. As general-purpose models become more capable, they also become better at domain-specific tasks and can outperform the domain-specific models.
- Because benchmarks are insufficient to capture real-world performance, it’s possible that BloombergGPT works well for Bloomberg for their specific use cases. The Bloomberg team certainly gained invaluable experience through training this model, which might enable them to better develop and operate future models.
- Both finetuning and prompting experiments require systematic processes. Doing prompt experiments enables developers to build an evaluation pipeline, data annotation guideline, and experiment tracking practices that will be stepping stones for finetuning.
- One benefit of finetuning, before prompt caching was introduced, was that it can help optimize token usage.

Finetuning and RAG

- Once you’ve maximized the performance gains from prompting, you might wonder whether to do RAG or finetuning next. The answer depends on whether your model’s failures are information-based or behavior-based.
- If the model fails because it lacks information, a RAG system that gives the model access to the relevant sources of information can help. Information-based failures happen when the outputs are factually wrong or outdated. Here are two example scenarios in which information-based failures happen:
  - The model doesn’t have the information.
  - The model has outdated information.
- The paper “Fine-Tuning or Retrieval?” by Ovadia et al. (2024) demonstrated that for tasks that require up-to-date information, such as questions about current events, RAG outperformed finetuned models.
- while finetuning can enhance a model’s performance on a specific task, it may also lead to a decline in performance in other areas.
- On the other hand, if the model has behavioral issues, finetuning might help. One behavioral issue is when the model’s outputs are factually correct but irrelevant to the task. 
  - For example, you ask the model to generate technical specifications for a software project to provide to your engineering teams. While accurate, the generated specs lack the details your teams need. Finetuning the model with well-defined technical specifications can make the outputs more relevant.
- Another issue is when it fails to follow the expected output format. 
  - For example, if you asked the model to write HTML code, but the generated code didn’t compile, it might be because the model wasn’t sufficiently exposed to HTML in its training data. You can correct this by exposing the model to more HTML code during finetuning.
- Semantic parsing is a category of tasks whose success hinges on the model’s ability to generate outputs in the expected format and, therefore, often requires finetuning. Semantic parsing is discussed briefly in Chapters 2 and 6. As a reminder, semantic parsing means converting natural language into a structured format like JSON
- Strong off-the-shelf models are generally good for common, less complex syntaxes like JSON, YAML, and regex. However, they might not be as good for syntaxes with fewer available examples on the internet, such as a domain-specific language for a less popular tool or a complex syntax.
- In short, finetuning is for form, and RAG is for facts. A RAG system gives your model external knowledge to construct more accurate and informative answers. A RAG system can help mitigate your model’s hallucinations. Finetuning, on the other hand, helps your model understand and follow syntaxes and styles
- If your model has both information and behavior issues, start with RAG. RAG is typically easier since you won’t have to worry about curating training data or hosting the finetuned models. When doing RAG, start with simple term-based solutions such as BM25 instead of jumping straight into something that requires vector databases.
- RAG can also introduce a more significant performance boost than finetuning. Ovadia et al. (2024) showed that for almost all question categories in the MMLU benchmark, RAG outperforms finetuning for three different models: Mistral 7B, Llama 2-7B, and Orca 2-7B.
- There’s no universal workflow for all applications. Figure7-3 shows some paths an application development process might follow over time. The arrow indicates what next step you might try. This figure is inspired by an example workflow shown by OpenAI (2023).
- Note that before any of the adaptation steps, you should define your evaluation criteria and design your evaluation pipeline, as discussed in Chapter4. This evaluation pipeline is what you’ll use to benchmark your progress as you develop your application. Evaluation doesn’t happen only in the beginning. It should be present during every step of the process:
- Steps:
  - Try to get a model to perform your task with prompting alone. Use the prompt engineering best practices covered in Chapter 5, including systematically versioning your prompts.
  - Add more examples to the prompt. Depending on the use case, the number of examples needed might be between 1 and 50.
  - If your model frequently fails due to missing information, connect it to data sources that can supply relevant information. When starting with RAG, begin by using basic retrieval methods like term-based search. Even with simple retrieval, adding relevant and accurate knowledge should lead to some improvement in your model’s performance.
  - Depending on your model’s failure modes, you might explore one of these next steps:
    - If the model continues having information-based failures, you might want to try even more advanced RAG methods, such as embedding-based retrieval.
    - If the model continues having behavioral issues, such as it keeps generating irrelevant, malformatted, or unsafe responses, you can opt for finetuning.
  - Combine both RAG and finetuning for even more performance boost.
- If, after considering all the pros and cons of finetuning and other alternate techniques, you decide to finetune your model, the rest of the chapter is for you. First, let’s look into the number one challenge of finetuning: its memory bottleneck.

### Memory Bottlenecks

- Because finetuning is memory-intensive, many finetuning techniques aim to minimize their memory footprint. Understanding what causes this memory bottleneck is necessary to understand why and how these techniques work. This understanding, in turn, can help you select a finetuning method that works best for you.
- Besides explaining finetuning’s memory bottleneck, this section also introduces formulas for back-of-the-napkin calculation of the memory usage of each model. This calculation is useful in estimating what hardware you’d need to serve or finetune a model.
- If you decide to skip this section, here are a few key takeaways. If you find any of these takeaways unfamiliar, the concepts in this section should help explain it:
  - Because of the scale of foundation models, memory is a bottleneck for working with them, both for inference and for finetuning. The memory needed for finetuning is typically much higher than the memory needed for inference because of the way neural networks are trained.
  - The key contributors to a model’s memory footprint during finetuning are its number of parameters, its number of trainable parameters, and its numerical representations.
  - The more trainable parameters, the higher the memory footprint
  - Quantization refers to the practice of converting a model from a format with more bits to a format with fewer bits. Quantization is a straightforward and efficient way to reduce a model’s memory footprint.
  - Inference is typically done using as few bits as possible, such as 16 bits, 8 bits, and even 4 bits.
  - Training is more sensitive to numerical precision, so it’s harder to train a model in lower precision. Training is typically done in mixed precision, with some operations done in higher precision (e.g., 32-bit) and some in lower precision (e.g., 16-bit or 8-bit).

Backpropagation and Trainable Parameters

- A key factor that determines a model’s memory footprint during finetuning is its number of trainable parameters. A trainable parameter is a parameter that can be updated during finetuning. During pre-training, all model parameters are updated. During inference, no model parameters are updated. During finetuning, some or all model parameters may be updated. The parameters that are kept unchanged are frozen parameters.
- The memory needed for each trainable parameter results from the way a model is trained. As of this writing, neural networks are typically trained using a mechanism called backpropagation
- With backpropagation, each training step consists of two phases:
  - Forward pass: the process of computing the output from the input.
  - Backward pass: the process of updating the model’s weights using the aggregated signals from the forward pass.
- During inference, only the forward pass is executed. During training, both passes are executed.
- At a high level, the backward pass works as follows:
  - Compare the computed output from the forward pass against the expected output (ground truth). If they are different, the model made a mistake, and the parameters need to be adjusted. The difference between the computed output and the expected output is called the loss.
  - Compute how much each trainable parameter contributes to the mistake. This value is called the gradient.
  - Adjust trainable parameter values using their corresponding gradient. How much each parameter should be readjusted, given its gradient value, is determined by the optimizer.
- During the backward pass, each trainable parameter comes with additional values, its gradient, and its optimizer states. Therefore, the more trainable parameters there are, the more memory is needed to store these additional values.

Memory Math

- It’s useful to know how much memory a model needs so that you can use the right hardware for it

Memory needed for inference

- During inference, only the forward pass is executed. The forward pass requires memory for the model’s weights. Let N be the model’s parameter count and M be the memory needed for each parameter; the memory needed to load the model’s parameters is: N x M
- N × M × 1.2
- Consider a 13B-parameter model. If each parameter requires 2 bytes, the model’s weights will require 13B × 2 bytes = 26 GB. The total memory for inference will be 26 GB × 1.2 = 31.2 GB.

Memory needed for training

- To train a model, you need memory for the model’s weights and activations, which has already been discussed. Additionally, you need memory for gradients and optimizer states, which scales with the number of trainable parameters.
- Training memory = model weights + activations + gradients + optimizer states

Numerical Representations

- Numerical values in neural networks are traditionally represented as float numbers.
- While FP64 is still used in many computations—as of this writing, FP64 is the default format for NumPy and pandas—it’s rarely used in neural networks because of its memory footprint. FP32 and FP16 are more common
- Numbers can also be represented as integers. Even though not yet as common as floating formats, integer representations are becoming increasingly popular.
- The right format for you depends on the distribution of numerical values of your workload (such as the range of values you need), how sensitive your workload is to small numerical changes, and the underlying hardware

Quantization

- The fewer bits needed to represent a model’s values, the lower the model’s memory footprint will be. A 10B-parameter model in a 32-bit format requires 40 GB for its weights, but the same model in a 16-bit format will require only 20 GB. Reducing precision, also known as quantization, is a cheap and extremely effective way to reduce a model’s memory footprint. It’s straightforward to do and generalizes over tasks and architecture
- What to quantize
  - Ideally, you want to quantize whatever is consuming most of your memory, but it also depends on what you can quantize without hurting performance too much
- When to quantize
  - Quantization can happen during training or post-training. Post-training quantization (PTQ) means quantizing a model after it’s been fully trained. PTQ is by far the most common. It’s also more relevant to AI application developers who don’t usually train models.

- It’s also possible to have different phases of training in different precision levels. For example, a model can be trained in higher precision but finetuned in lower precision. This is especially common with foundation models, where the team training a model from scratch might be an organization with sufficient compute for higher precision training. Once the model is published, developers with less compute access can finetune that model in lower precision.

### Finetuning Techniques

Parameter-Efficient Finetuning

- In the early days of finetuning, models were small enough that people could finetune entire models. This approach is called full finetuning. In full finetuning, the number of trainable parameters is exactly the same as the number of parameters.
- Full finetuning can look similar to training. The main difference is that training starts with randomized model weights, whereas finetuning starts with model weights that have been previously trained.
- We also haven’t touched on the fact that full finetuning, especially supervised finetuning and preference finetuning, typically requires a lot of high-quality annotated data that most people can’t afford. Due to the high memory and data requirements of full finetuning, people started doing partial finetuning.
- n partial finetuning, only some of the model’s parameters are updated. For example, if a model has ten layers, you might freeze the first nine layers and finetune only the last layer,22 reducing the number of trainable parameters to 10% of full finetuning.
- While partial finetuning can reduce the memory footprint, it’s parameter-inefficient. Partial finetuning requires many trainable parameters to achieve performance close to that of full finetuning
- This brings up the question: How to achieve performance close to that of full finetuning while using significantly fewer trainable parameters? Finetuning techniques resulting from this quest are parameter-efficient
- There’s no clear threshold that a finetuning method has to pass to be considered parameter-efficient. However, in general, a technique is considered parameter-efficient if it can achieve performance close to that of full finetuning while using several orders of magnitude fewer trainable parameters.
- During finetuning, they kept the model’s original parameters unchanged and only updated the adapters. The number of trainable parameters is the number of parameters in the adapters. On the GLUE benchmark, they achieved a performance within 0.4% of full finetuning using only 3% of the number of trainable parameters
- However, the downside of this approach is that it increases the inference latency of the finetuned model. The adapters introduce additional layers, which add more computational steps to the forward pass, slowing inference.
- PEFT enables finetuning on more affordable hardware, making it accessible to many more developers. PEFT methods are generally not only parameter-efficient but also sample-efficient. While full finetuning may need tens of thousands to millions of examples to achieve notable quality improvements, some PEFT methods can deliver strong performance with just a few thousand examples.
- Given PEFT’s obvious appeal, PEFT techniques are being rapidly developed. The next section will give an overview of these techniques before diving deeper into the most common PEFT technique: LoRA.

PEFT techniques

- 


### Summary