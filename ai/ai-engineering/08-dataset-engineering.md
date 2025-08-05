# Chapter 8 - Dataset Engineering

- The quality of a model depends on the quality of its training data. The best ML team in the world with infinite compute can’t help you finetune a good model if you don’t have data
- The goal of dataset engineering is to create a dataset that allows you to train the best model, ideally within your allocated budget.
- As fewer companies can afford to develop models from scratch, more are turning to data to differentiate their AI performance. As models demand more data, data handling becomes more challenging and demands more investments in talent and infrastructure.
- Many AI companies now employ data labelers, dataset creators, and data quality engineers, either integrated into or working alongside their core engineering teams.
- If the model landscape is confusing enough with numerous offerings, the data landscape is even more complex, with an ever-growing array of datasets and techniques being introduced. This chapter gives you an overview of the data landscape and considerations to take into account when building your own dataset.
- It begins with data curation, addressing questions like What data do you need? How much? What does it mean for data to be of high quality? It then discusses techniques for data synthesis and processing. Data curation, generation, and processing don’t follow a linear path. You’ll likely have to go back and forth between different steps.
- For the same model, different training phases aim to teach the model different capabilities, and, therefore, require datasets with different attributes
- This chapter focuses on post-training data because that’s more relevant to application developers. However, I’ll also include lessons from pre-training data when these lessons are insightful for post-training.
- The increasing focus on data during AI development has given rise to data-centric AI, as opposed to model-centric AI:
  - Model-centric AI tries to improve AI performance by enhancing the models themselves. This involves designing new architectures, increasing the sizes of the models, or developing new training techniques. 
  - Data-centric AI tries to improve AI performance by enhancing the data. This involves developing new data processing techniques and creating high-quality datasets that allow better models to be trained with fewer resources.
  - In the early days of deep learning, many AI benchmarks were model-centric. Given a dataset like ImageNet, people try to train the best possible model using the same dataset. In recent years, more benchmarks have become data-centric. Given the same model, people try to develop a dataset that gives this model the best performance.
  - The model-centric and data-centric division helps guide research. In reality, however, meaningful technological progress often requires investment in both model and data improvements.

### Data Curation

- While not all issues with AI models can be solved with data, data is often a key part of the solution. The right data can make the model more capable, safer, and able to handle longer contexts. Conversely, poor data can cause the model to increase biases and hallucinations. Mistakes in data can harm the model and waste resources.
- Data curation is a science that requires understanding how the model learns and what resources are available to help it learn. Dataset builders should work closely with application and model developers. In a small team, they might be the same person—the person responsible for training a model is also responsible for acquiring the data for it. However, organizations with high data demands often employ specialized roles
- What data you need depends on your task and what you want to teach the model. For self-supervised finetuning, you need sequences of data. For instruction finetuning, you need data in the (instruction, response) format. For preference finetuning, you need data in the (instruction, winning response, losing response) format. To train a reward model, you can use the same data format as preference finetuning or use data with annotated scores for each of your examples in the ((instruction, response), score) format.
- Training data should exhibit the behaviors you want your model to learn. Acquiring high-quality data annotations is always challenging, but it’s even more challenging if you want to teach models complex behaviors such as chain-of-thought (CoT) reasoning and tool use
- Chain-of-thought
  - Generating multi-step responses can be tedious and time-consuming—explaining how to solve a math problem step-by-step is much more challenging than simply giving the final answer.
- Tool use
  - Given the vast amount of knowledge a model acquires during pre-training, many models might intuitively know how to use certain tools. However, a model’s tool use ability can be improved by showing it tool use examples.
- Data curation isn’t just about creating new data to help a model learn new behaviors but is also about removing existing data to help a model unlearn bad behaviors. 
- At a high level, however, data curation follows the three criteria: data quality, data coverage, and data quantity.
- To give an intuition about these terms, if you think of model training as cooking, the data fed into the model is the ingredients. Data quality is equivalent to the quality of the ingredients—you can’t have good food if your ingredients are spoiled. Data coverage is equivalent to having the right mix of ingredients (e.g., you shouldn’t have too much or too little sugar). Data quantity is about how many ingredients you should have

Data Quality

- A small amount of high-quality data can outperform a large amount of noisy data, e.g., data that is irrelevant or inconsistent. The creators of the Yi model family found that 10K carefully crafted instructions are superior to hundreds of thousands of noisy instructions
- Similarly, “LIMA: Less Is More for Alignment” (Zhou et al., 2023) shows that a 65B-parameter Llama model, finetuned with 1,000 carefully curated prompts and responses, can produce answers that are either equivalent or strictly preferred to GPT-4 in 43% of cases, as judged by human annotators. However, the downside of having too few data examples is that LIMA is not as robust as product-grade models.
- Notably, they found that human-generated data is more prone to errors and inconsistencies, particularly for nuanced safety policies. This led them to develop AI-assisted annotation tools to ensure high data quality.
- Most people understand the importance of data quality, but what does it mean for data to be high-quality? The short answer is that data is considered high-quality if it helps you do your job efficiently and reliably
- In general, data can be considered high-quality if it has the following six characteristics: relevant, aligned with task requirements, consistent, correctly formatted, unique, and compliant
  - Relevant
    - The training examples should be relevant to the task you’re training the model to do.
  - Aligned with task requirements
    - The annotations should align with the task’s requirements. For example, if the task requires factual consistency, the annotations should be factually correct. If the task requires creativity, the annotations should be creative.
  - Consistent
    - Annotations should be consistent across examples and annotators. If you ask two annotators to annotate the same example, their annotations shouldn’t be too different. If the task is to score essays from 1 to 5, would two essays with the same score be of the same quality? Inconsistent annotations can confuse the model, making it harder for the model to learn.
  - Correctly formatted
    - All examples should follow the format expected by the model. Redundant formatting tokens can interfere with the model’s learning, and, therefore, they should be removed
  - Sufficiently unique
    - This refers to unique examples in your data.5 In the context of model training, duplications can introduce biases and cause data contamination. I use “sufficiently unique” because specific use cases can tolerate different levels of duplications.
  - Compliant
    - Data should be compliant with all relevant internal and external policies (including laws and regulations)

Data Coverage

- A model’s training data should cover the range of problems you expect it to solve. Real-world users often have a wide range of problems, and the way they express those problems can vary significantly. Having data that captures the diverse usage patterns of your application is key for the model to perform well.
- For example, if some users construct detailed instructions with abundant references while some other users prefer short instructions, your finetuning data should include both detailed and short instructions. If user queries typically have typos, you should include examples with typos. If your application works with multiple programming languages, your training data should include the programming languages your users care about.
- Meta shared that Llama 3 doesn’t deviate significantly from older Llama versions in terms of model architecture. Llama 3’s performance gains are “primarily driven by improvements in data quality and diversity as well as by increased training scale.”
- This confirms a common belief that high-quality code and math data is more effective than natural language text in boosting the model’s reasoning capabilities.
- This brings up a question: How do we decide on the right data mix? A simple approach is to choose a data mix that accurately reflects the real-world application usage. You can also use experiments to find optimal data mixes.

Data Quantity

- Asking how much data you need is like asking how much money you need
- While millions of examples sounds like a lot, it’s small compared to the data typically needed to train a foundation model from scratch. For reference, Llama 2 and Llama 3 were trained using 2 trillion and 16 trillion tokens, respectively. If each example is 2,000 tokens, it’d be equivalent to 1 billion and 15 billion examples.
- You might wonder: if I have millions of examples, shouldn’t I just train a model from scratch? You can and should evaluate whether training a model from scratch would improve your performance. While finetuning on top of a pre-trained model is typically more efficient than training from scratch, there are situations when finetuning can be worse, especially when you have a lot of training data. This is due to a phenomenon called ossification, where pre-training can ossify (i.e., freeze) the model weights so that they don’t adapt as well to the finetuning data (Hernandez et al., 2021). Smaller models are more susceptible to ossification than larger models.
- Other than data quality and data diversity, three other factors influence how much data you need:
  - Finetuning techniques
    - Full finetuning promises to give the best performance, but it requires orders of magnitude more data than PEFT methods like LoRA. If you have tens of thousands to millions of (instruction, response) pairs, you might want to attempt full finetuning. If you have only a few hundred or a few thousand examples, PEFT might work best.
  - Task complexity
    - A simple task, such as classifying whether a product review is positive or negative, will require much less data than a complex task, such as a question answering about financial filings.
  - Base model’s performance
    - The closer the base model is to the desirable performance, the fewer examples are needed to get there. Assuming that bigger base models are better, you might need fewer examples to finetune big models. This is the opposite of pre-training, where bigger models need more training data.
- OpenAI’s finetuning guide shows that if you have fewer examples (100), more advanced models give you better finetuning performance. This is likely because the more advanced models already perform better out of the box. However, after finetuning on a lot of examples (550,000), all five models in the experiment performed similarly
- In short, if you have a small amount of data, you might want to use PEFT methods on more advanced models. If you have a large amount of data, use full finetuning with smaller models.
- Before investing in curating a large dataset, you might want to start with a small, well-crafted dataset (e.g., 50 examples) to see if finetuning can improve the model. If this small dataset is sufficient to achieve your desirable performance, that’s great. Clear improvements suggest that more data will improve the performance even more. If no improvement is observed with small data, a bigger dataset will rarely do the trick.
- However, be careful before concluding that finetuning with a small dataset doesn’t improve a model. Many things, other than data, can impact finetuning’s results, such as the choice of hyperparameters (e.g., the learning rate is too high or too low), data quality, poorly crafted prompts, etc. 
- It’s possible to reduce the amount of high-quality data needed by first finetuning your model using lower-quality or less-relevant data. Here are three examples of this approach:
  - Self-supervised → supervised
    - You want to finetune a model to answer legal questions. Your (question, answer) set is small, but you have many legal documents. You can first finetune your model on legal documents in a self-supervised manner, then further finetune the model on (question, answer) pairs.
  - Less-relevant data → relevant data
    - You want to finetune a model to classify sentiments for product reviews, but you have little product sentiment data and much more tweet sentiment data. You can first finetune your model to classify tweet sentiments, then further finetune it to classify product sentiments.
  - Synthetic data → real data
    - You want to finetune a model to predict medical conditions from medical reports. Due to the sensitive nature of this task, your data is limited. You can use AI models to synthesize a large amount of data to finetune your model first, then further finetune it on your real data. This approach is harder to get right, as you’ll have to do two distinct finetuning jobs while coordinating the transitioning between them. If you don’t know what you’re doing, you might end up using more compute just to produce a model worse than what you would’ve gotten by just finetuning with high-quality data
- Experimenting with a small dataset can help you estimate how much more data you’ll need. You can finetune a model on subsets of your current dataset—e.g., 25%, 50%, 100%—and plot how performance scales with dataset size. A steep performance gain slope with increasing dataset size means that you can expect significant performance improvement by doubling your data. A plateau slope means that doubling your data will give only a small improvement
- In most cases, additional training examples yield diminishing returns: the same number of examples typically gives a lower performance boost as the dataset grows. For example, the first 1,000 examples might improve a model’s accuracy by ten percentage points, but the next 1,000 examples might only improve it by five.
- While a larger number of finetuning examples generally improves a model’s performance, the diversity of the examples matters, too.
- The diversity of data can be reflected in task types (such as summarization and question answering), topic diversity (such as fashion, finance, and technology), and the expected output formats (such as JSON outputs or yes-or-no answers).
- How much data to use for finetuning is determined not just by what you need but also by what you can afford. If you budget $10,000 for data annotation and each example costs $2 to annotate, you can have at most 5,000 examples. You might also need to balance the budget for data and compute. Spending more money on data leaves you less money for compute, and vice versa.

Data Acquisition and Annotation

- The goal of data acquisition is to produce a sufficiently large dataset with the quality and diversity you need, while ensuring that your data practices respect user privacy and comply with regulations. Data acquisition involves gathering data through methods such as sourcing public data, purchasing proprietary data, annotating data, and synthesizing data.
- The most important source of data, however, is typically data from your own application. If you can figure out a way to create a data flywheel that leverages data generated by your users to continually improve your product, you will gain a significant advantage
- Application data is ideal because it’s perfectly relevant and aligned with your task. In other words, it matches the distribution of the data that you care about, which is incredibly hard to achieve with other data sources.
- For example, the process of creating an (instruction, response) dataset might look as follows:
  - Find available datasets with the desirable characteristics. You might find one promising dataset with 10,000 examples. 
  - Remove low-quality instructions. Let’s say this leaves you with 9,000 examples. 
  - Set aside the instructions with low-quality responses. Let’s say you find 3,000 such examples. This leaves you with 6,000 examples of high-quality instructions and high-quality responses. 
  - Manually write responses for the 3,000 high-quality instructions. Now your dataset has a total of 9,000 high-quality examples. 
  - Realizing that there’s not enough data for topic X, manually create a set of 100 instruction templates about X. Use an AI model to synthesize 2,000 instructions using these 10 templates. 
  - Manually annotate these 2,000 synthetic instructions. Now your dataset has a total of 11,000 examples.
- RESOURCES FOR PUBLICLY AVAILABLE DATASETS
  - Hugging Face and Kaggle each host hundreds of thousands of datasets. 
  - Google has a wonderful and underrated Dataset Search. 
  - Governments are often great providers of open data. Data.gov hosts hundreds of thousands of datasets, and data.gov.in hosts tens of thousands. 
  - University of Michigan’s Institute for Social Research ICPSR has data from tens of thousands of social studies. 
  - UC Irvine’s Machine Learning Repository and OpenML are two older dataset repositories, each hosting several thousand datasets. 
  - The Open Data Network lets you search among tens of thousands of datasets. 
  - Cloud service providers often host a small collection of open datasets; the most notable one is AWS’s Open Data. 
  - ML frameworks often have small pre-built datasets that you can load while using the framework, such as TensorFlow datasets. 
  - Some evaluation harness tools host evaluation benchmark datasets that are sufficiently large for PEFT finetuning. For example, Eleuther AI’s lm-evaluation-harness hosts 400+ benchmark datasets, averaging 2,000+ examples per dataset. 
  - The Stanford Large Network Dataset Collection is a great repository for graph datasets.
- Often, you might need to annotate your own data for finetuning. Annotation is challenging not just because of the annotation process but also due to the complexity of creating clear annotation guidelines. For example, you need to explicitly state what a good response looks like, and what makes it good. Can a response be correct but unhelpful? What’s the difference between responses that deserve a score of 3 and 4? Annotation guidelines are needed for both manual and AI-powered annotations.
- Some teams, including LinkedIn, have reported that annotation guidelines were among the most challenging parts of their AI engineering pipeline. It’s alarming how often people abandon careful annotation halfway due to the time and effort required, hoping instead that their models will figure out the right responses on their own. Many models are strong enough that they can occasionally succeed, but relying on models to figure that out might be too risky for many applications.
- The good news is that these guidelines are the same as those for evaluation data, as discussed in Chapter 4. This is another argument for why you should invest more time in curating evaluation guidelines and data. If you’re lucky, your evaluation examples can be augmented or used as seed examples to synthesize new data. In the next section we’ll discuss how to do so.

### Data Augmentation and Synthesis

- 

### Data Processing


### Summary