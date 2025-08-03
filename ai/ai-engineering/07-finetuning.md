# Chapter 7 - Finetuning

- Finetuning is the process of adapting a model to a specific task by further training the whole model or part of the model. Chapters 5 and 6 discuss prompt-based methods, which adapt a model by giving it instructions, context, and tools. Finetuning adapts a model by adjusting its weights.
- Finetuning can enhance various aspects of a model. It can improve the model’s domain-specific capabilities, such as coding or medical question answering, and can also strengthen its safety. However, it is most often used to improve the model’s instruction-following ability, particularly to ensure it adheres to specific output styles and formats.
- While finetuning can help create models that are more customized to your needs, it also requires more up-front investment. A question I hear very often is when to finetune and when to do RAG. After an overview of finetuning, this chapter will discuss the reasons for finetuning and the reasons for not finetuning, as well as a simple framework for thinking about choosing between finetuning and alternate methods.
- Compared to prompt-based methods, finetuning incurs a much higher memory footprint. At the scale of today’s foundation models, naive finetuning often requires more memory than what’s available on a single GPU. This makes finetuning expensive and challenging to do.
- A memory-efficient approach that has become dominant in the finetuning space is PEFT (parameter-efficient finetuning). 
- With prompt-based methods, knowledge about how ML models operate under the hood is recommended but not strictly necessary. However, finetuning brings you to the realm of model training, where ML knowledge is required.
- This chapter is the most technically challenging one for me to write, not because of the complexity of the concepts, but because of the broad scope these concepts cover. I suspect it might also be technically challenging to read

### Finetuning Overview

### When to Finetune

### Memory Bottlenecks

### Finetuning Techniques

### Summary