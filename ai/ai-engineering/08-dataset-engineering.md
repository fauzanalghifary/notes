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

- Together with compute and talent, data is the hardest challenge of AI. It’s been a long-term goal of the whole industry to be able to generate data programmatically. Two processes commonly used are data augmentation and data synthesis:
  - Data augmentation creates new data from existing data (which is real). For example, given a real image of a cat, you can flip it to create a new image of the same cat.8 
  - Data synthesis generates data to mimic the properties of real data. For example, you can simulate how a mouse moves through a web page to generate data for what bot movements would look like.
- In other words, augmented data is derived from real data, whereas synthetic data isn’t real. However, since the goal of both augmentation and synthesis is to automate data creation, sometimes the two terms are used interchangeably. In this chapter, I’ll often use data synthesis to refer to both.
- With AI being capable of generating data indistinguishable from that generated by humans, it’s possible to synthesize much more sophisticated data, such as doctor’s notes, contracts, financial statements, product descriptions, images, video commercials, etc. This makes it easier to generate data and enables more synthetic data use cases.
- While synthetic data promises to significantly reduce the pressure for human-generated data, synthetic data doesn’t completely replace human data. In many use cases, as discussed in “Limitations to AI-generated data”, mixing human- and AI-generated data often produces the best value.

Synthesis

- You can synthesize data to improve the golden data trio: quantity, coverage, and quality. You can also synthesize data to mitigate privacy concerns and distill models:
  - To increase data quantity
  - To increase data coverage
  - To increase data quality
  - To mitigate privacy concerns
  - To distill models
  - These are just five of the many reasons why people turn to data synthesis. Because of its undeniable appeal, more models are being trained with synthetic data and more techniques are being developed to synthesize data.

Traditional Data Synthesis Techniques

- Data synthesis isn’t unique to AI. It has a long history in software testing, gaming, and robotics. Using algorithms to generate data is also called procedural generation, as opposed to manual generation. Procedural generation is commonly used in gaming to generate content such as levels, maps, items, and characters on the fly.10 Most data generation techniques used in these industries can be applied to AI.
- Traditionally, two approaches for data synthesis and augmentation have been rule-based and simulation. A newer method made possible by advanced AI models is using AI itself to synthesize data. This section gives a quick overview of these two traditional techniques before moving on to AI-powered data synthesis in the next section.

Rule-based data synthesis

- The simplest way to generate data is to use predefined rules and templates. For example, to create a credit card transaction, start with a transaction template and use a random generator like Faker to populate each field in this template:
- You can procedurally generate new data from existing data by applying simple transformations. For images, you can randomly rotate, crop, scale, or erase part of an image. A flipped image of a cat should still be a cat. A slightly cropped image of a soccer game should still be a soccer game
- For texts, you can randomly replace a word with a similar word, assuming that this replacement wouldn’t change the meaning or the sentiment of the sentence. For example, the original sentence “She’s a fantastic nurse” can generate a new example: “She’s a great nurse”.
- One interesting transformation is perturbation: adding noise to existing data to generate new data

Simulation

- Instead of running experiments to collect data in the real world, where it can be expensive and dangerous, you can simulate these experiments virtually. For example, to test how a self-driving car reacts when encountering a horse on the highway, it’d be dangerous to release an actual horse on the highway. Instead, you simulate this situation in a virtual environment.
- Simulations allow you to run multiple experiments with minimal costs while avoiding accidents and physical damage. A robot that works in simulations might not work in the real world, but if it fails in simulations, it’ll likely fail in the real world.
- Simulations are particularly valuable for generating data for rare events. For example, in finance, researchers can simulate scenarios such as a company successfully going public or a significant bankruptcy to understand their market impacts.
- Both rule-based and simulation-based techniques have been useful for many use cases, but it wasn’t until AI become capable of generating realistic and high-quality data that data synthesis really took off. Let’s look into those methods next.

AI-Powered Data Synthesis

- Powerful AI models open many new possibilities for simulations. AI can simulate the outcomes of arbitrary programs.
- AI can simulate humans. For example, imagine you want to train a bot to play chess. A game played by humans might take too long. Matches with AI players would be much faster. To train its Dota 2 bot, OpenAI used a simulator that enabled the bot to play approximately 180 years’ worth of games every day. The bot learned by playing against itself, an approach called self-play, which helped it develop and refine strategies over time (OpenAI, 2019). Similarly, DeepMind used self-play to collect data from millions of Go games to train AlphaGo (Silver et al., 2016).
- AI’s paraphrasing and translation abilities can be used to augment existing datasets
- AI can generate data for both pre-training and post-training, though synthetic data is intentionally included much more often in post-training than in pre-training. One possible explanation for this is that pre-training’s goal is to increase the model’s knowledge, and while AI can synthesize existing knowledge in different formats, it’s harder to synthesize new knowledge.
- However, as the internet becomes flooded with AI-generated content, models that rely on internet data are likely already pre-trained on synthetic data.
- Data synthesis for post-training is also more common because post-training data, including both instruction data and preference data, generally demands the most effort to produce.

Instruction data synthesis

- During instruction finetuning, each example includes an instruction and a response. AI can be used to synthesize the instructions, the responses, or both. For example, you can use AI to generate instructions and humans to write responses. You can also use humans to write instructions and AI to generate responses:
- A creative approach is to use synthetic data to finetune a model for understanding longer contexts. For example, if your current model processes a maximum of 8K tokens but you want it to handle 128K tokens, the long-context finetuning process might look like this:
  - Split long documents into shorter chunks (e.g., under 8K tokens). 
  - For each short chunk, generate several (question, answer) pairs. 
  - For each (question, answer) pair, use the original long document, which may exceed 8K tokens but be shorter than your target length, as the context. This trains the model to use the extended context to answer questions.
- Combining all three methods together—code translation, code back-translation, and code generation—Llama 3’s data synthesis workflow is quite impressive. To summarize, here’s how these three methods work together:
  - Use AI to generate problem descriptions. 
  - Use AI to generate solutions for each problem in different programming languages. 
  - Use AI to generate unit tests to test the generated code. 
  - Prompt AI to fix errors in the synthesized code. 
  - Use AI to translate generated code to different programming languages. Filter out translated code that doesn’t pass tests. 
  - Use AI to generate conversations about the code, including code explanation and adding documentation. Filter out generated explanations and documentation that doesn’t pass back-translation verification. 
  - Using this pipeline, Dubey et al. were able to generate over 2.7 million synthetic coding-related examples for the supervised finetuning of Llama 3.1.

Data verification

- Given the importance of data quality in the model’s performance, it’s crucial that we have a way to verify the quality of data. The quality of AI-generated data can be measured the same way you’d evaluate other AI outputs—by functional correctness and AI judges.
- For synthetic data that can’t be verified by functional correctness, it’s common to use AI verifiers. An AI verifier can be a general-purpose AI judge or a specialized scorer. There are many ways to frame the verification problem. In the simplest form, the AI verifier can assign each generated example a score from 1 to 5 or classify each example as good or bad. You can also describe to a foundation model the quality requirements and instruct the model to determine if a data example meets these requirements.
- If you care about the factual consistency of data, you can use the factual inconsistency detection techniques discussed in Chapter4 to filter out examples that are likely to contain hallucinations.
- Even though there are many techniques to evaluate synthetic data, evaluation remains challenging. As with other AI applications, the ultimate quality test for AI-generated data is its real-world performance—whether it can improve the model’s performance—and synthetic data has passed this test for many models.

Limitations to AI-generated data

- Given the increasing usefulness of synthetic data, it’s exciting to imagine the possibility of never having to worry about human-annotated data again. However, while the role of synthetic data will certainly continue to grow in importance over time, AI-generated data might never entirely replace human-generated data. There are many reasons why, but the four major ones are the difference in quality, the limitations of imitation, potential model collapse, and the way AI generation of data obscures its lineage.
- Quality control
  - AI’s generated data can be of low quality, and, as people never tire of saying, “garbage in, garbage out.” As mentioned earlier, people will be hesitant to use synthetic data if they can’t verify its quality. Being able to develop reliable methods and metrics to evaluate data will be essential in making synthetic data more useful.
- Superficial imitation
  - This research shows that the imitation models are good at mimicking the style of the teacher models but might struggle with factual accuracy and generalization to tasks outside the training data.
  - Gudibande et al. (2023) suggest that for improvement in reasoning capabilities, we need to focus on improving the quality of the base models.
- Potential model collapse
  - It’s also unclear how much AI-generated data a model can train on. Some studies have shown that recursively using AI-generated data in training causes irreversible defects in the resulting models, degrading their performance over time.
  - One possible explanation is that AI models are more likely to generate probable events (e.g., not having cancer) and less likely to generate improbable events (e.g., having cancer). Over multiple iterations, probable events become over-represented, whereas improbable events become under-represented in the generated data. This causes models to output more common events over time while forgetting rare events.
  - In “Is Model Collapse Inevitable?” Gerstgrasser et al. (2024) argue that while model collapse is inevitable if the entire training dataset is synthetic, it can be avoided by mixing synthetic data with real data
  - AI-generated data might also perpetuate biases.
- Obscure data lineage
  -  AI models are influenced by their training data and can sometimes regurgitate it without the user knowing. This creates risks. Let’s say you use model X to generate data to train your model. If model X was trained on data with copyright violations, your model might also violate copyrights.
  - Or imagine you then use benchmark B to evaluate your model, which shows a strong performance. However, if model X was also trained on benchmark B, your result on B is contaminated. Without clear data lineage, it’s hard to assess a model’s commercial viability or trust its performance.
- In the next section, let’s switch gears to discuss one special use case of data synthesis where AI-generated data isn’t just supplementary but is required: model distillation.

Model Distillation

- Model distillation (also called knowledge distillation) is a method in which a small model (student) is trained to mimic a larger model (teacher) (Hinton et al., 2015). The knowledge of the big model is distilled into the small model, hence the term distillation.
- Traditionally, the goal of model distillation is to produce smaller models for deployment. Deploying a big model can be resource-intensive. Distillation can produce a smaller, faster student model that retains performance comparable to the teacher. For example, DistilBERT, a model distilled from BERT, reduces the size of a BERT model by 40% while retaining 97% of its language comprehension capabilities and being 60% faster
- Not all models can be distilled. Many model licenses prohibit using their outputs to train other models, particularly to train competing models.
- The Llama 3 paper notes that while training on data generated by a more competent model can significantly improve a model’s performance, training indiscriminately on self-generated data doesn’t improve the model’s performance and can even degrade it. However, by introducing mechanisms to verify the quality of synthetic data and using only verified synthetic data, they were able to continually improve a model using its generated data.

### Data Processing

- Data needs to be processed according to the requirements of each use case. This section discusses some data processing steps for reference.
- With a large amount of data, each of these processing steps can take hours, if not days.

Inspect Data

- Let’s say that after combing through public and internal data, you’ve gathered a raw dataset. The first thing to do is inspect the data to get a sense of its quality. Get the data’s information and statistics. Where does the data come from? How has it been processed? What else has it been used for?
- There are many data exploration tools you should use, but they won’t be replacements for manual data inspection. In every project I’ve worked on, staring at data for just 15 minutes usually gives me some insight that could save me hours of headaches. Greg Brockman, an OpenAI co-founder, tweeted: “Manual inspection of data has probably the highest value-to-prestige ratio of any activity in machine learning.”
- Look at your data to see if the examples make sense. If it’s annotated data, pick out a few queries and try to annotate them yourself to see if your annotations match the given annotations. This will give you a sense of how trustworthy the annotations are. Fact-check the responses. How unique are the examples? Are there any examples with the same query but with different responses? Are there any examples with the same responses but with different queries?

Deduplicate Data

- Duplicated data can skew the data distribution and introduce biases into your model.
- The duplicated entries might lead the model to the wrong conclusion that all red-colored items should be expensive. Duplications can cause test set contamination. When splitting duplicated data into train and test sets, one example might be in the train set and its duplicate in the test set.
- An Anthropic study demonstrated that repeating 0.1% of the data 100 times can cause an 800M parameter model’s performance to degrade to that of a 400M parameter model despite the other 90% of the training tokens remaining unique (Hernandez et al., 2022). Even when duplications don’t hurt your model’s performance, they can waste your time and compute.
- Depending on the data, there are many forms of duplication, some of which are harder to detect. For example, here are a few types of duplications in a dataset of documents:
  - Whole document duplications: the same document appearing more than once. 
  - Intra-document duplications: e.g., the same paragraph appears twice in one document. 
  - Cross-document duplications: e.g., the same popular quote appears in multiple documents.
- The task of deduplication can leverage the same techniques used for similarity measurements (discussed in Chapter3). Data deduplication is also used for identity resolution, determining whether two identities (e.g., two social media profiles) are the same. Here are some concrete ways you can deduplicate data:
  - Pairwise comparison
  - Hashing
  - Dimensionality reduction

Clean and Filter Data

- Data needs to be cleaned to make your model performant and safe.
- First, you might want to remove extraneous formatting tokens. Since many public datasets are scraped from the internet, extraneous HTML tags are quite common. Unless you want to train your model on HMTL tags, remove them. Databricks found that removing extraneous Markdown and HTML tokens improved their model’s accuracy by 20% while reducing their input token lengths by 60%.
- You need to clean your data of anything that isn’t compliant with your policies, such as PII, sensitive data, copyrighted data, or data that is considered toxic.
- You also might want to remove low-quality data, using techniques discussed in “Data verification” to detect low-quality data.
- If there is more data than you need or can afford to use (e.g., due to your compute budget), you can further filter your data. For example, you can use active learning techniques to select examples that are the most helpful for your model to learn from. You can also use importance sampling to find examples that are most important to your task. Their efficiencies depend on whether you have a good way to evaluate the importance of each training example. Meta researchers, in their paper on data pruning (Sorscher et al., 2022), concluded that the discovery of good data-pruning metrics can significantly reduce the resource costs of modern deep learning.

Format Data

- Once you’ve deduplicated and cleaned your data, you need to get it into the right format expected by the model you’re finetuning. Each model uses a specific tokenizer and expects data in a specific chat template, as discussed in Chapter 5. Getting data into the wrong chat template can cause strange bugs in your model.
- Different finetuning data formats can impact your finetuned model’s performance. Experiments to determine the best format for you can be helpful.

### Summary

- Even though the actual process of creating training data is incredibly intricate, the principles of creating a dataset are surprisingly straightforward. To build a dataset to train a model, you start by thinking through the behaviors you want your model to learn and then design a dataset to show these behaviors. Due to the importance of data, teams are introducing dedicated data roles responsible for acquiring appropriate datasets while ensuring privacy and compliance
- However, dataset design across training phases shares the same three core criteria: quality, coverage, and quantity.
- While how much data a model is trained on grabs headlines, having high-quality data with sufficient coverage is just as important. A small amount of high-quality data can outperform a large amount of noisy data. Similarly, many teams have found that increasing the diversity of their datasets is key to improving their models’ performance.
- Due to the challenge of acquiring high-quality data, many teams have turned to synthetic data. While generating data programmatically has long been a goal, it wasn’t until AI could create realistic, complex data that synthetic data became a practical solution for many more use cases.
- Data is challenging because many steps in dataset creation aren’t easily automatable. It’s hard to annotate data, but it’s even harder to create annotation guidelines. It’s hard to automate data generation, but it’s even harder to automate verifying it. While data synthesis helps generate more data, you can’t automate thinking through what data you want. You can’t easily automate annotation guidelines. You can’t automate paying attention to details.
- However, challenging problems lead to creative solutions. One thing that stood out to me when doing research for this chapter is how much creativity is involved in dataset design. There are so many ways people construct and evaluate data. I hope that the range of data synthesis and verification techniques discussed in this chapter will give you inspiration for how to design your dataset.
