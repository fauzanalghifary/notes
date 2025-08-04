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

- The existing prolific world of PEFT generally falls into two buckets: adapter-based methods and soft prompt-based methods. However, it’s likely that newer buckets will be introduced in the future.
- Adapter-based methods refer to all methods that involve additional modules to the model weights
- As of this writing, LoRA (Hu et al., 2021) is by far the most popular adapter-based method, and it will be the topic of the following section. 
- If adapter-based methods add trainable parameters to the model’s architecture, soft prompt-based methods modify how the model processes the input by introducing special trainable tokens. These additional tokens are fed into the model alongside the input tokens. They are called soft prompts because, like the inputs (hard prompts), soft prompts also guide the model’s behaviors
- However, soft prompts differ from hard prompts in two ways:
  - Hard prompts are human-readable. They typically contain discrete tokens such as “I”, “write”, “a”, and “lot”. In contrast, soft prompts are continuous vectors, resembling embedding vectors, and are not human-readable. 
  - Hard prompts are static and not trainable, whereas soft prompts can be optimized through backpropagation during the tuning process, allowing them to be adjusted for specific tasks.
- Some people describe soft prompting as a crossover between prompt engineering and finetuning
- From this analysis, it’s clear that LoRA dominates. Soft prompts are less common, but there seems to be growing interest from those who want more customization than what is afforded by prompt engineering but who don’t want to invest in finetuning.

LoRA

- Unlike the original adapter method by Houlsby et al. (2019), LoRA (Low-Rank Adaptation) (Hu et al., 2021) incorporates additional parameters in a way that doesn’t incur extra inference latency. Instead of introducing additional layers to the base model, LoRA uses modules that can be merged back to the original layers.
- LoRA (Low-Rank Adaptation) is built on the concept of low-rank factorization, a long-standing dimensionality reduction technique. The key idea is that you can factorize a large matrix into a product of two smaller matrices to reduce the number of parameters, which, in turn, reduces both the computation and memory requirements.
- While factorization can significantly reduce the number of parameters, it’s lossy because it only approximates the original matrix. The higher the rank, the more information from the original matrix the factorization can preserve.
- Like the original adapter method, LoRA is parameter-efficient and sample-efficient. The factorization enables LoRA to use even fewer trainable parameters. The LoRA paper showed that, for GPT-3, LoRA achieves comparable or better performance with full finetuning on several tasks while using only ~4.7M trainable parameters, 0.0027% of full finetuning.

Why does LoRA work?

- If a model requires a lot of parameters to learn certain behaviors during pre-training, shouldn’t it also require a lot of parameters to change its behaviors during finetuning?
- The same question can be raised for data. If a model requires a lot of data to learn a behavior, shouldn’t it also require a lot of data to meaningfully change this behavior? How is it possible that you need millions or billions of examples to pre-train a model, but only a few hundreds or thousands of examples to finetune it?
- Many papers have argued that while LLMs have many parameters, they have very low intrinsic dimensions
- This suggests that pre-training acts as a compression framework for downstream tasks. In other words, the better trained an LLM is, the easier it is to finetune the model using a small number of trainable parameters and a small amount of data.
- You might wonder, if low-rank factorization works so well, why don’t we use LoRA for pre-training as well? Instead of pre-training a large model and applying low-rank factorization only during finetuning, could we factorize a model from the start for pre-training? 
- It’s possible that one day not too far in the future, researchers will develop a way to scale up low-rank pre-training to hundreds of billions of parameters. However, if Aghajanyan et al.’s argument is correct—that pre-training implicitly compresses a model’s intrinsic dimension—full-rank pre-training is still necessary to sufficiently reduce the model’s intrinsic dimension to a point where low-rank factorization can work. It would be interesting to study exactly how much full-rank training is necessary before it’s possible to switch to low-rank training.

LoRA configurations

- To apply LoRA, you need to decide what weight matrices to apply LoRA to and the rank of each factorization. This section will discuss the considerations for each of these decisions.
- The efficiency of LoRA, therefore, depends not only on what matrices LoRA is applied to but also on the model’s architecture, as different architectures have different weight matrices.
- While there have been examples of LoRA with other architectures, such as convolutional neural networks (Dutt et al., 2023; Zhong et al., 2024; Aleem et al., 2024), LoRA has been primarily used for transformer models.
- The beauty of LoRA is that while its performance depends on its rank, studies have shown that a small r, such as between 4 and 64, is usually sufficient for many use cases. A smaller r means fewer LoRA parameters, which translates to a lower memory footprint.
- The LoRA authors observed that, to their surprise, increasing the value of r doesn’t increase finetuning performance. This observation is consistent with Databricks’ report that “increasing r beyond a certain value may not yield any discernible increase in quality of model output” (Sooriyarachchi, 2023).25 Some argue that a higher r might even hurt as it can lead to overfitting. However, in some cases, a higher rank might be necessary. Raschka (2023) found that r = 256 achieved the best performance on his tasks.

Serving LoRA adapters

- LoRA not only lets you finetune models using less memory and data, but it also simplifies serving multiple models due to its modularity. To understand this benefit, let’s examine how to serve a LoRA-finetuned model.
- In general, there are two ways to serve a LoRA-finetuned model:
  - Merge the LoRA weights A and B into the original model to create the new matrix Wʹ prior to serving the finetuned model. Since no extra computation is done during inference, no extra latency is added. 
  - Keep W, A, and B separate during serving. The process of merging A and B back to W happens during inference, which adds extra latency.
- The modularity of LoRA adapters means that LoRA adapters can be shared and reused. There are publicly available finetuned LoRA adapters that you can use the way you’d use pre-trained models. You can find them on Hugging Face26 or initiatives like AdapterHub.
- You might be wondering: “LoRA sounds great, but what’s the catch?” The main drawback of LoRA is that it doesn’t offer performance as strong as full finetuning. It’s also more challenging to do than full finetuning as it involves modifying the model’s implementation, which requires an understanding of the model’s architecture and coding skills. However, this is usually only an issue for less popular base models. PEFT frameworks—such as Hugging Face’s PEFT, Axolotl, unsloth, and LitGPT—likely support LoRA for popular base models right out of the box.

Quantized LoRA

- Rather than trying to reduce LoRA’s number of parameters, you can reduce the memory usage more effectively by quantizing the model’s weights, activations, and/or gradients during finetuning

Model Merging and Multi-Task Finetuning

- If finetuning allows you to create a custom model by altering a single model, model merging allows you to create a custom model by combining multiple models. Model merging offers you greater flexibility than finetuning alone. You can take two available models and merge them together to create a new, hopefully more useful, model. You can also finetune any or all of the constituent models before merging them together.
- While you don’t have to further finetune the merged model, its performance can often be improved by finetuning. Without finetuning, model merging can be done without GPUs, making merging particularly attractive to indie model developers that don’t have access to a lot of compute.
- The goal of model merging is to create a single model that provides more value than using all the constituent models separately. The added value can come from improved performance. For example, if you have two models that are good at different things on the same task, you can merge them into a single model that is better than both of them on that task
- The added value can also come from a reduced memory footprint, which leads to reduced costs. For example, if you have two models that can do different tasks, they can be merged into one model that can do both tasks but with fewer parameters. 
- One important use case of model merging is multi-task finetuning.
- Model merging is also appealing when you have to deploy models to devices such as phones, laptops, cars, smartwatches, and warehouse robots. On-device deployment is often challenging because of limited on-device memory capacity. Instead of squeezing multiple models for different tasks onto a device, you can merge these models together into one model that can perform multiple tasks while requiring much less memory.
- Many model-merging techniques are experimental and might become outdated as the community gains a better understanding of the underlying theory. For this reason, I’ll focus on the high-level merging approaches instead of any individual technique.
- Model merging approaches differ in how the constituent parameters are combined. Three approaches covered here are summing, layer stacking, and concatenation
- Summing
  - This approach involves adding the weight values of constituent models together.
- Layer stacking
  - In this approach, you take different layers from one or more models and stack them on top of each other. For example, you might take the first layer from model 1 and the second layer from model 2. This approach is also called passthrough or frankenmerging.
  - Unlike the merging by summing approach, the merged models resulting from layer stacking typically require further finetuning to achieve good performance.
  - An interesting use case of layer stacking is model upscaling. Model upscaling is the study of how to create larger models using fewer resources. Sometimes, you might want a bigger model than what you already have, presumably because bigger models give better performance. For example, your team might have originally trained a model to fit on your 40 GB GPU. However, you obtained a new machine with 80 GB, which allows you to serve a bigger model. Instead of training a new model from scratch, you can use layer stacking to create a larger model from the existing model.
- Concatenation
  - Instead of adding the parameters of the constituent models together in different manners, you can also concatenate them. The merged component’s number of parameters will be the sum of the number of parameters from all constituent components
  - Concatenation isn’t recommended because it doesn’t reduce the memory footprint compared to serving different models separately. Concatenation might give better performance, but the incremental performance might not be worth the number of extra parameters

Finetuning Tactics

Finetuning frameworks and base models

- While many things around finetuning—deciding whether to finetune, acquiring data, and maintaining finetuned models—are hard, the actual process of finetuning is more straightforward. There are three things you need to choose: a base model, a finetuning method, and a framework for finetuning.
- Base models
  - At the beginning of an AI project, when you’re still exploring the feasibility of your task, it’s useful to start with the most powerful model you can afford. If this model struggles to produce good results, weaker models are likely to perform even worse. If the strongest model meets your needs, you can then explore weaker models, using the initial model as a benchmark for comparison.
  - For finetuning, the starting models vary for different projects. OpenAI’s finetuning best practices document gives examples of two development paths: the progression path and the distillation path.
  - The progression path looks like this:
    - Test your finetuning code using the cheapest and fastest model to make sure the code works as expected.
    - Test your data by finetuning a middling model. If the training loss doesn’t go down with more data, something might be wrong. 
    - Run a few more experiments with the best model to see how far you can push performance. 
    - Once you have good results, do a training run with all models to map out the price/performance frontier and select the model that makes the most sense for your use case.
  - The distillation path might look as follows:
    - Start with a small dataset and the strongest model you can afford. Train the best possible model with this small dataset. Because the base model is already strong, it requires less data to achieve good performance. 
    - Use this finetuned model to generate more training data. 
    - Use this new dataset to train a cheaper model.
- Finetuning methods
  - Recall that adapter techniques like LoRA are cost-effective but typically don’t deliver the same level of performance as full finetuning. If you’re just starting with finetuning, try something like LoRA, and attempt full finetuning later.
  - Depending on the base model and the task, full finetuning typically requires at least thousands of examples and often many more. PEFT methods, however, can show good performance with a much smaller dataset. If you have a small dataset, such as a few hundred examples, full finetuning might not outperform LoRA.
  - Take into account how many finetuned models you need and how you want to serve them when deciding on a finetuning method. Adapter-based methods like LoRA allow you to more efficiently serve multiple models that share the same base model. With LoRA, you only need to serve a single full model, whereas full finetuning requires serving multiple full models.
- Finetuning frameworks
  - The easiest way to finetune is to use a finetuning API where you can upload data, select a base model, and get back a finetuned model. Like model inference APIs, finetuning APIs can be provided by model providers, cloud service providers, and third-party providers
  - A limitation of this approach is that you’re limited to the base models that the API supports. Another limitation is that the API might not expose all the knobs you can use for optimal finetuning performance. Finetuning APIs are suitable for those who want something quick and easy, but they might be frustrating for those who want more customization.
  - You can also finetune using one of many great finetuning frameworks available, such as LLaMA-Factory, unsloth, PEFT, Axolotl, and LitGPT. They support a wide range of finetuning methods, especially adapter-based techniques. If you want to do full finetuning, many base models provide their open source training code on GitHub that you can clone and run with your own data. Llama Police has a more comprehensive and up-to-date list of finetuning frameworks and model repositories.
  - If you do only adapter-based techniques, a mid-tier GPU might suffice for most models. If you need more compute, you can choose a framework that integrates seamlessly with your cloud provider.
  - To finetune a model using more than one machine, you’ll need a framework that helps you do distributed training, such as DeepSpeed, PyTorch Distributed, and ColossalAI.
- Finetuning hyperparameters
  - Depending on the base model and the finetuning method, there are many hyperparameters you can tune to improve finetuning efficiency. For specific hyperparameters for your use case, check out the documentation of the base model or the finetuning framework you use. Here, I’ll cover a few important hyperparameters that frequently appear.
  - Learning rate
    - The learning rate determines how fast the model’s parameters should change with each learning step. If you think of learning as finding a path toward a goal, the learning rate is the step size. If the step size is too small, it might take too long to get to the goal. If the step size is too big, you might overstep the goal, and, hence, the model might never converge.
    - A universal optimal learning rate doesn’t exist. You’ll have to experiment with different learning rates, typically between the range of 1e-7 to 1e-3, to see which one works best. A common practice is to take the learning rate at the end of the pre-training phase and multiply it with a constant between 0.1 and 1.
    - The loss curve can give you hints about the learning rate. If the loss curve fluctuates a lot, it’s likely that the learning rate is too big. If the loss curve is stable but takes a long time to decrease, the learning is likely too small. Increase the learning rate as high as the loss curve remains stable.
  - Batch size
    - The batch size determines how many examples a model learns from in each step to update its weights. A batch size that is too small, such as fewer than eight, can lead to unstable training.36 A larger batch size helps aggregate the signals from different examples, resulting in more stable and reliable updates.
    - In general, the larger the batch size, the faster the model can go through training examples. However, the larger the batch size, the more memory is needed to run your model. Thus, batch size is limited by the hardware you use.
    - As of this writing, compute is still a bottleneck for finetuning. Often, models are so large, and memory is so constrained, that only small batch sizes can be used. This can lead to unstable model weight updates. To address this, instead of updating the model weights after each batch, you can accumulate gradients across several batches and update the model weights once enough reliable gradients are accumulated. This technique is called gradient accumulation
  - Number of epochs
    - An epoch is a pass over the training data. The number of epochs determines how many times each training example is trained on.
    - Small datasets may need more epochs than large datasets. For a dataset with millions of examples, 1–2 epochs might be sufficient. A dataset with thousands of examples might still see performance improvement after 4–10 epochs.
    - The difference between the training loss and the validation loss can give you hints about epochs. If both the training loss and the validation loss still steadily decrease, the model can benefit from more epochs (and more data). If the training loss still decreases but the validation loss increases, the model is overfitting to the training data, and you might try lowering the number of epochs.
  - Prompt loss weight
    - For instruction finetuning, each example consists of a prompt and a response, both of which can contribute to the model’s loss during training. During inference, however, prompts are usually provided by users, and the model only needs to generate responses. Therefore, response tokens should contribute more to the model’s loss during training than prompt tokens.
    - The prompt model weight determines how much prompts should contribute to this loss compared to responses. If this weight is 100%, prompts contribute to the loss as much as responses, meaning that the model learns equally from both. If this weight is 0%, the model learns only from responses. Typically, this weight is set to 10% by default, meaning that the model should learn some from prompts but mostly from responses.

### Summary

- Outside of the evaluation chapters, finetuning has been the most challenging chapter to write. It touched on a wide range of concepts
- The process of finetuning itself isn’t hard. Many finetuning frameworks handle the training process for you. These frameworks can even suggest common finetuning methods with sensible default hyperparameters.
- However, the context surrounding finetuning is complex. It starts with whether you should even finetune a model. This chapter started with the reasons for finetuning and the reasons for not finetuning. It also discussed one question that I have been asked many times: when to finetune and when to do RAG.
- In its early days, finetuning was similar to pre-training—both involved updating the model’s entire weights. However, as models increased in size, full finetuning became impractical for most practitioners. The more parameters to update during finetuning, the more memory finetuning needs. Most practitioners don’t have access to sufficient resources (hardware, time, and data) to do full finetuning with foundation models.
- Many finetuning techniques have been developed with the same motivation: to achieve strong performance on a minimal memory footprint
- A comment I often hear from practitioners is that finetuning is easy, but getting data for finetuning is hard. Obtaining high-quality annotated data, especially instruction data, is challenging. The next chapter will dive into these challenges.


